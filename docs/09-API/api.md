---
title: API - Firmware Behavior & UART Message Specification
tags:
- ESP32-S3
- UART
- MQTT
- Recon Amphibot
- EGR314
---

# API — Firmware Behavior & UART Message Specification

**Subsystem:** ESP32-S3 Wireless Communication Gateway  
**Board ID:** `0x01`  
**Team:** 302 - R6 Recon Amphibot  

---

## Packet Frame (All Messages)

All messages follow the standardized 64-byte UART packet:

```yaml
config:
  packet:
    bitsPerRow: 16
    bitWidth: 64
packet-beta
title Message
0:  "0x41"
1:  "0x5A"
2:  "Source ID"
3:  "Dest ID"
4-61:  "Message (Variable Length ≤ 58 Bytes)"
62: "0x59"
63: "0x42"
```

**All multi-byte values are transmitted in big-endian format.**

### Board IDs

| Board | Source / Dest ID |
|-------|-----------------|
| ESP32 Wireless Gateway | `0x01` |
| Sensor + HMI PIC | `0x02` |
| Actuator PIC | `0x03` |
| Broadcast | `0xFF` |

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
- `0x0005` Emergency Stop → immediate safe-stop
- `0x0008` System Status → logged + forwarded

---

## Firmware Overview

The ESP32 firmware performs:

1. Periodic sensor polling (UART)
2. UART message reception and forwarding
3. MQTT publish (telemetry, status, errors)
4. MQTT subscribe → UART command translation
5. System failsafe (WiFi loss → emergency stop)

---

## Firmware Constants

```python
MY_ID = 0x01
SENSOR_ID = 0x02
ACTUATOR_ID = 0x03
BROADCAST_ID = 0xFF

HEADER = bytes([0x41, 0x5A])
FOOTER = bytes([0x59, 0x42])
PACKET_LEN = 64

SEND_INTERVAL_MS = 1000

def build_packet(dest_id: int, msg_type: int, data: bytes) -> bytes:
    """Build a valid UART packet."""
    assert len(data) <= 56, "Payload too long"
    assert b'\x41\x5A' not in data and b'\x59\x42' not in data
    payload = bytes([msg_type >> 8, msg_type & 0xFF]) + data
    payload = payload.ljust(58, b'\x00')
    return HEADER + bytes([MY_ID, dest_id]) + payload + FOOTER

def uart_send(packet: bytes):
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
| `0x0001 → actuator` | Set motor speed | — | — |
| `0x0002 → sensor` | Request sensor data | — | — |
| `0x0005 broadcast` | Emergency stop | `status` | `"ESTOP"` |
| `0x0008 broadcast` | Status update | `status` | status string |
| `0x0003` | Sensor data | `telemetry` | JSON |
| `0x0004` | Motor telemetry | `telemetry` | JSON |
| `0x0006/0x0007` | Errors | `error` | code/string |
| `0x0043` | Button press | `status` | button # |

---

## Message Specifications

### `0x0001` — Motor Speed Command (SEND)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0001` | `0x0001` | `0x0001` |
| 6 | motor_id | `uint8_t` | 1 | 4 | 1 |
| 7–8 | motor_speed | `int16_t` | -500 | 500 | 200 |

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
| 6 | motor_state | `uint8_t` | 0 | 2 | 1 |
| 7–8 | current_speed | `int16_t` | -500 | 500 | 180 |

### `0x0005` — Emergency Stop (SEND + RECEIVE + BROADCAST)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0005` | `0x0005` | `0x0005` |
| 6 | stop_source | `uint8_t` | 1 | 3 | 3 |

### `0x0006` — Error Code (RECEIVE)

| Byte | Name | Type | Min | Max | Example |
|------|------|------|-----|-----|---------|
| 4–5 | message_type | `uint16_t` | `0x0006` | `0x0006` | `0x0006` |
| 6 | subsystem_id | `uint8_t` | 1 | 3 | 2 |
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
if len(data) > PACKET_LEN:
    continue

if src == MY_ID:
    continue

if dest != MY_ID and dest != BROADCAST_ID:
    uart_send(frame)
```

---

## Sender Implementation

- Always send valid 64-byte packets.
- Respect prefix/suffix.
- Prevent invalid payload content.
- Rate-limit transmission.
- Prioritize forwarding messages.

---

## Acknowledgement Behavior

```python
def send_ack(dest_id, msg_type):
    data = bytes([msg_type >> 8, msg_type & 0xFF])
    ack = build_packet(dest_id, 0x00FF, data)
    uart_send(ack)
```

---

## Notes

- Payload must not contain header/footer patterns.
- Messages must not exceed 64 bytes.
- System enters safe-stop on WiFi loss.
- UART forwarding has priority.

---

## Example Frame

ESP32 (`Source ID = 0x01`) requests sensor data from the Sensor board (`Dest ID = 0x02`):

- Byte 0: `0x41`
- Byte 1: `0x5A`
- Byte 2: `0x01` (Source ID = ESP32)
- Byte 3: `0x02` (Dest ID = Sensor board)
- Byte 4: `0x00` (Message Type high byte)
- Byte 5: `0x02` (Message Type low byte → `0x0002` Request Sensor Data)
- Byte 6: `0x01` (Sensor ID)
- Byte 7–61: zero-filled/reserved
- Byte 62: `0x59`
- Byte 63: `0x42`