---
title: Module Requirements
tags:
  - EGR314
  - Team 302
  - ESP32
  - Wireless Communication
---

# ESP32 Wireless Communication Gateway Requirements

This section defines the functional and performance requirements for the **ESP32 Wireless Communication Gateway** subsystem. The gateway enables remote robot control, MQTT communication, telemetry transmission, and safe system operation during wireless communication failures. Each requirement includes threshold and target objectives together with the final implementation outcome.

---

## Module Requirements

**Subsystem:** ESP32 Wireless Communication Gateway

**Engineer:** Mihir Patel

| Requirement Description | Threshold | Target | Stretch | Final Result |
|-------------------------|-----------|--------|----------|--------------|
| Surface-mounted 3.3 V switching regulator | Output ≥ 3.2 V | Stable 3.3 V output | No | **Met** — AP63203WU-7 provided a stable 3.3 V supply under full system load |
| ESP32 microcontroller operation | Successful boot | Reliable Wi-Fi and MQTT operation | No | **Met** — ESP32-S3 booted consistently and maintained reliable MQTT connectivity |
| Wireless communication | Basic one-way communication | Full two-way MQTT messaging | No | **Met** — Complete publish/subscribe MQTT communication implemented |
| UART communication with system bus | Basic TX/RX | Structured packets with CRC | No | **Met** — 64-byte UART packet protocol implemented and successfully forwarded between subsystems |
| Video data handling | Image acquisition | Stable low-latency video streaming | Yes | **Partially Met** — Camera firmware (`camera_module.py`) and laptop viewer (`viewer.py`) were fully implemented with chunked MQTT frame transmission, but live streaming was not demonstrated on the final hardware |
| Connection loss handling | Detect wireless disconnection | Automatic safe-stop behavior | Yes | **Met** — Wi-Fi failsafe implemented; emergency stop packet broadcast over UART whenever the wireless connection was lost |

---

# Requirement Review

## Successfully Implemented Requirements

The five primary subsystem requirements were successfully implemented and verified.

The **AP63203WU-7** switching regulator delivered a stable **3.3 V** output throughout testing while maintaining approximately **847.5 mA** of available current headroom beyond the calculated worst-case system load.

The **ESP32-S3-WROOM-1-N8R8** booted reliably and maintained stable Wi-Fi and MQTT connectivity throughout subsystem testing.

Two-way MQTT communication operated as intended, allowing telemetry to be published to the MQTT broker while receiving remote control commands that were forwarded across the UART daisy-chain.

The structured **64-byte UART communication protocol** was implemented according to the subsystem API specification, including packet framing, board identification, routing, and message-type handling.

The Wi-Fi connection-loss failsafe, originally identified as a stretch objective, was also fully implemented. Whenever the MQTT connection was interrupted, the firmware detected the failure and immediately broadcast an emergency stop packet (`0x0005`) across the UART network to safely halt the actuator subsystem.

---

## Stretch Goal: Video Data Handling

Video streaming was the only requirement that reached software completion without full hardware verification.

The complete software infrastructure was implemented, including:

- `camera_module.py` for OV5640 initialization, image capture, and chunked Base64 MQTT frame publishing.
- `viewer.py` for laptop-side frame reconstruction and real-time OpenCV display.
- `config_additions.py` defining the camera MQTT topics (`TOPIC_CAM_CMD`, `TOPIC_CAM_FRAME`, and `TOPIC_CAM_STATUS`).

The architecture supports:

- On-demand image capture
- Continuous video streaming
- Configurable frame rates
- Remote stream start and stop using MQTT commands

Although the software implementation was completed, live end-to-end camera streaming was not demonstrated on the final assembled hardware.

The primary limitation was the time available for complete hardware bring-up following PCB assembly. The OV5640 interface requires careful hardware verification, and the MicroPython `esp32-camera` library requires firmware compiled with camera support enabled. These final validation steps were not completed within the project timeline.

Additional discussion is provided in the **Hardware V2.0** and **Reflection** sections.

---

# Feature-to-Requirement Rationale

The ESP32 Wireless Communication Gateway is a critical subsystem because it provides the communication interface between the operator and every onboard subsystem.

Reliable wireless communication enables:

- Remote driving
- Real-time telemetry monitoring
- MQTT-based command and control
- System status reporting
- Safe operation during communication failures

These system-level responsibilities directly resulted in requirements for reliable bidirectional Wi-Fi communication, structured UART messaging, and automatic failsafe behavior.

Power-related requirements ensure stable ESP32 operation while preventing communication failures caused by voltage instability.

The video-streaming requirement originated from the Amphibot's primary reconnaissance mission, where a live FPV feed provides the operator with situational awareness during exploration.

Although this stretch objective was not fully demonstrated on hardware, it directly influenced several hardware design decisions. Most notably, it justified selecting the **ESP32-S3-WROOM-1-N8R8** instead of the N4 variant because the additional **8 MB PSRAM** is required by the OV5640 camera driver and cannot be replaced through software optimization alone.

---

# Requirement Verification Summary

| Total Requirements | Fully Met | Partially Met | Not Met |
|-------------------:|----------:|--------------:|---------:|
| 6 | 5 | 1 | 0 |