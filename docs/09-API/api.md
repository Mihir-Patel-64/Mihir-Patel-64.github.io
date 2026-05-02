---
title: API - Firmware Behavior & UART Message Specification
tags:
- ESP32-S3
- UART
- MQTT
- Recon Amphibot
- EGR314
---

# API - Firmware Behavior & UART Message Specification

**Subsystem:** ESP32-S3 Wireless Communication Gateway  
**Board ID:** `'M'` (ASCII 0x4D)  
**Team:** 302 - R6 Recon Amphibot  

---

## Packet Frame (All Messages)

All messages follow the standardized 64-byte UART packet:
[ 0x41 | 0x5A | Source ID | Dest ID | Message (58 Bytes) | 0x59 | 0x42 ]

**All multi-byte values are transmitted in big-endian format.**

### Board IDs

| Board | Source / Dest ID | ASCII |
|-------|-----------------|-------|
| ESP32 Wireless Gateway (Mihir) | `'M'` | 0x4D |
| Sensor + HMI PIC (Laksh) | `'L'` | 0x4C |
| Actuator PIC (Raunak) | `'R'` | 0x52 |
| Broadcast (all boards) | `'*'` | 0x2A |

---

## Message Responsibility Summary

### Messages Sent (by ESP32)
- `0x0001` — Motor Speed Command
- `0x0002` — Sensor Data Request
- `0x0005` — Emergency Stop
- `0x0008` — System Status
- `0x00FF` — Acknowledgement (ACK)

### Messages Received (by ESP32)
- `0x0003` — Sensor Data Response
- `0x0004` — Motor Telemetry
- `0x0006` — Error Code
- `0x0007` — Error Message
- `0x0043` — Button Event

### Broadcast Messages Handled
- `0x0005` Emergency Stop → immediate safe-stop + camera stream stop
- `0x0008` System Status → logged + forwarded to MQTT

---

## Firmware Overview

The ESP32 firmware performs:

1. Periodic sensor polling via UART every 15 seconds
2. Periodic system status broadcast every 15 seconds
3. UART message reception, routing, and forwarding
4. MQTT publish (telemetry, status, errors, heartbeat)
5. MQTT subscribe → UART command translation
6. Camera command handling (capture, stream on/off)
7. System failsafe (WiFi loss → emergency stop broadcast over UART)

---

## Firmware Constants

```python
MY_ID        = ord('M')   # 0x4D
SENSOR_ID    = ord('L')   # 0x4C
ACTUATOR_ID  = ord('R')   # 0x52
BROADCAST_ID = ord('*')   # 0x2A

HEADER     = bytes([0x41, 0x5A])   # 'A' 'Z'
FOOTER     = bytes([0x59, 0x42])   # 'Y' 'B'
PACKET_LEN = 64
SEND_INTERVAL_MS = 500             # minimum ms between UART transmissions

def build_packet(dest_id: int, msg_type: int, data: bytes) -> bytes:
    """Build a valid 64-byte UART packet."""
    assert len(data) <= 56, "Payload too long"
    # Sanitize payload to prevent accidental header/footer sequences
    clean = bytearray(data)
    for i in range(len(clean) - 1):
        if clean[i:i+2] == b'\x41\x5A' or clean[i:i+2] == b'\x59\x42':
            clean[i] = 0x00
    payload = bytearray([msg_type >> 8, msg_type & 0xFF])
    payload += bytearray(clean)
    while len(payload) < 58:
        payload += b'\x00'
    return HEADER + bytes([MY_ID, dest_id]) + bytes(payload) + FOOTER

def uart_send(packet: bytes):
    """Rate-limited UART transmit — enforces 500ms minimum interval."""
    assert len(packet) == PACKET_LEN
    uart.write(packet)

def is_valid_packet(frame: bytes) -> bool:
    return (
        len(frame) == PACKET_LEN
        and frame[0:2] == HEADER
        and frame[62:64] == FOOTER
    )
```

---

## UART ↔ MQTT Mapping

| UART Frame | Meaning | MQTT Topic | Payload |
|-------------|---------|------------|---------|
| `0x0001 → 'R'` | Set motor speed | — | — |
| `0x0002 → 'L'` | Request sensor data | — | — |
| `0x0005 → '*'` | Emergency stop broadcast | `TOPIC_STATUS` | `"ESTOP"` |
| `0x0008 → '*'` | Status update broadcast | `TOPIC_STATUS` | status string |
| `0x0003` received | Sensor data | `TOPIC_TELEMETRY` | JSON |
| `0x0004` received | Motor telemetry | `TOPIC_TELEMETRY` | JSON |
| `0x0006` received | Error code | `TOPIC_ERROR` | JSON |
| `0x0007` received | Error message | `TOPIC_ERROR` | string |
| `0x0043` received | Button press | `TOPIC_STATUS` | `camera_capture btn=N` |

---

## MQTT Subscribe Commands

Commands received on `TOPIC_SUB` are translated into UART packets or camera actions:

| MQTT Command | Example | Action |
|---|---|---|
| `forward <rpm>` | `forward 500` | Sends `0x0001` to actuator, direction=1, rpm=500 |
| `reverse <rpm>` | `reverse 300` | Sends `0x0001` to actuator, direction=2, rpm=300 |
| `stop` / `estop` | `stop` | Broadcasts `0x0005` ESTOP, stops camera stream |
| `setpoint <rpm>` | `setpoint 400` | Sends `0x0001`, positive=forward, negative=reverse |
| `sensor <id>` | `sensor 1` | Sends `0x0002` sensor request to Sensor board |
| `status` | `status` | Broadcasts `0x0008` system status |
| `capture` | `capture` | Triggers single OV5640 JPEG capture → MQTT |
| `stream on` | `stream on` | Starts continuous JPEG stream → MQTT |
| `stream off` | `stream off` | Stops continuous JPEG stream |

---

## Camera MQTT Topics

The camera subsystem uses three dedicated MQTT topics, separate from the main
telemetry topics. These are defined in `config.py` via `config_additions.py`:

| Topic | Direction | Description |
|---|---|---|
| `TOPIC_CAM_CMD` | Subscribe (inbound) | Receives camera commands: `capture`, `stream on`, `stream off` |
| `TOPIC_CAM_FRAME` | Publish (outbound) | Chunked base64 JPEG frames, QoS 0 |
| `TOPIC_CAM_STATUS` | Publish (outbound) | Camera health and frame metadata, QoS 1 |

### Camera Frame Format

Each MQTT message on `TOPIC_CAM_FRAME` is a JSON object:

```json
{
  "seq":   1,
  "chunk": 0,
  "total": 3,
  "data":  "<base64-encoded JPEG bytes>"
}
```

The receiver reassembles chunks in order using `seq` and `chunk` indices.
Chunk size is 4096 bytes of raw JPEG per message (~5460 base64 characters).

### Button Event Camera Mapping

Button events received over UART (`0x0043`) are mapped to camera commands:

| Button Number | Camera Action |
|---|---|
| 1 | Single capture |
| 2 | Stream on |
| 3 | Stream off |
| Other | Publishes `camera_capture btn=N` to `TOPIC_STATUS` |

---

## Message Specifications

### `0x0001` — Motor Speed Command (SEND)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0001` | `0x0001` | `0x0001` |
| 6 | direction | `uint8_t` | 0 | 2 | 1 |
| 7–8 | rpm | `uint16_t` | 0 | 2300 | 500 |

_Direction: 0=STOP, 1=FORWARD, 2=REVERSE. RPM range: 0–2300._

### `0x0002` — Sensor Request (SEND)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0002` | `0x0002` | `0x0002` |
| 6 | sensor_id | `uint8_t` | 1 | 3 | 1 |

### `0x0003` — Sensor Data (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0003` | `0x0003` | `0x0003` |
| 6–7 | imu_tilt | `int16_t` | -18000 | 18000 | 450 |
| 8–9 | temperature | `int16_t` | -4000 | 12500 | 2350 |
| 10–11 | hazard_score | `uint16_t` | 0 | 10000 | 2500 |
| 12–13 | humidity | `uint16_t` | 0 | 10000 | 5500 |

_All values are fixed-point ×100._

### `0x0004` — Motor Telemetry (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0004` | `0x0004` | `0x0004` |
| 6 | direction | `uint8_t` | 0 | 2 | 1 |
| 7–8 | current_rpm | `uint16_t` | 0 | 2300 | 180 |

_Direction: 0=STOPPED, 1=FORWARD, 2=REVERSE._

### `0x0005` — Emergency Stop (SEND + RECEIVE + BROADCAST)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0005` | `0x0005` | `0x0005` |
| 6 | stop_source | `uint8_t` | — | — | `'M'` |

_stop_source is the ASCII board ID of the board initiating the stop._

### `0x0006` — Error Code (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0006` | `0x0006` | `0x0006` |
| 6 | subsystem_id | `uint8_t` | — | — | `'L'` |
| 7 | error_code | `uint8_t` | 0 | 255 | 10 |

### `0x0007` — Error Message (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0007` | `0x0007` | `0x0007` |
| 6–60 | error_msg | `char[55]` | — | — | `"Motor fault"` |
| 61 | null | `uint8_t` | 0 | 0 | `0x00` |

### `0x0008` — System Status (SEND + BROADCAST)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0008` | `0x0008` | `0x0008` |
| 6 | status_code | `uint8_t` | 0 | 4 | 1 |

_Status codes: 0=IDLE, 1=RUNNING, 2=WARNING, 3=ERROR, 4=ESTOP._

### `0x0043` — Button Event (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0043` | `0x0043` | `0x0043` |
| 6 | button_num | `uint8_t` | 1 | 8 | 1 |

### `0x00FF` — Acknowledgement (SEND)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x00FF` | `0x00FF` | `0x00FF` |
| 6–7 | acked_msg_type | `uint16_t` | `0x0001` | `0xFFFF` | `0x0003` |

---

## Receiver Implementation

```python
if src == MY_ID:
    continue   # drop looped-back own packets

if dest != MY_ID and dest != BROADCAST_ID:
    uart.write(frame)   # forward — not addressed to us
    return
```

---

## Sender Implementation

- Always send valid 64-byte packets.
- Respect `AZ` prefix and `YB` suffix.
- Sanitize payload to prevent accidental header/footer byte sequences.
- Rate-limit transmission — minimum 500ms between UART sends.
- Prioritize forwarding packets not addressed to this board.

---

## Acknowledgement Behavior

```python
def send_ack(dest_id: int, msg_type: int):
    data = bytes([msg_type >> 8, msg_type & 0xFF])
    ack = build_packet(dest_id, MSG_ACK, data)
    uart_send(ack)
```

ACKs are sent in response to: `0x0003`, `0x0004`, `0x0005`, `0x0006`,
`0x0007`, and `0x0043`.

---

## WiFi Failsafe Behavior

When the WiFi connection is lost, the firmware automatically:

1. Sets the WiFi LED (GPIO35) to OFF
2. Stops any active camera stream
3. Broadcasts an emergency stop packet (`0x0005`) over UART to all boards
4. Flashes all four LEDs three times to indicate the fault condition

```python
async def wifi_han(state):
    led_wifi.value(1 if state else 0)
    if not state:
        cam.stream_stop()
        uart.write(build_packet(BROADCAST_ID, MSG_EMERGENCY_STOP, bytes([MY_ID])))
```

---

## Notes

- Payload must not contain header (`0x41 0x5A`) or footer (`0x59 0x42`) byte sequences.
- All packets must be exactly 64 bytes.
- System enters safe-stop on WiFi loss automatically.
- UART forwarding takes priority over local processing.
- Heartbeat published to `TOPIC_HB` every 5 seconds to confirm broker connectivity.

---

## Example Frame

ESP32 (`Source ID = 'M'`) requests sensor data from the Sensor board
(`Dest ID = 'L'`):

- Byte 0: `0x41` (header 'A')
- Byte 1: `0x5A` (header 'Z')
- Byte 2: `0x4D` (Source ID = 'M', ESP32)
- Byte 3: `0x4C` (Dest ID = 'L', Sensor board)
- Byte 4: `0x00` (Message Type high byte)
- Byte 5: `0x02` (Message Type low byte → `0x0002` Request Sensor Data)
- Byte 6: `0x01` (Sensor ID)
- Byte 7–61: zero-filled/reserved
- Byte 62: `0x59` (footer 'Y')
- Byte 63: `0x42` (footer 'B')