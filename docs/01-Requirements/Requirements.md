---
title: Module's Requirements
tags:
- EGR314
- Team 302
---

# Project Requirements

The purpose of this section is to define the engineering requirements for the **Wireless Communication subsystem (ESP32)**. This module is responsible for enabling remote control of the exploration device, transmitting telemetry and video data to the user, and ensuring safe system behavior if the wireless connection is lost. Each requirement below describes the minimum functionality required for the module to be considered operational, along with target performance goals and optional stretch objectives.

---

## Module Requirements — Wireless Communication (ESP32)  
**Teammate: Mihir Patel**

| Requirement Description | Measure of Threshold | Target Measure | Stretch (Yes/No) | Final Result |
|------------------------|----------------------|----------------|------------------|--------------|
| Surface-mounted 3.3 V switching regulator | Output ≥ 3.2 V | Stable 3.3 V output | No | Met — AP63203WU-7 produced stable 3.3 V under full load |
| ESP32 microcontroller operation | Boots successfully | Runs Wi-Fi and MQTT | No | Met — ESP32-S3 booted reliably and connected to MQTT broker |
| Wireless communication | One-way data transfer | Two-way MQTT messaging | No | Met — Full publish/subscribe MQTT messaging achieved |
| UART communication with system bus | Basic TX/RX | Structured packets with CRC | No | Met — 64-byte structured UART packets implemented and forwarded |
| Video data handling | Video stream present | Stable low-latency stream | Yes | Partially Met — Camera firmware (`camera_module.py`) and laptop viewer (`viewer.py`) were fully implemented with chunked MQTT frame delivery, but live streaming was not demonstrated on final hardware |
| Connection loss handling | Detect disconnection | Trigger safe-stop behavior | Yes | Met — WiFi failsafe implemented; emergency stop packet broadcast over UART on connection loss |

---

## Requirement Review

### Requirements Successfully Met

The four core requirements were all achieved. The AP63203WU-7 switching regulator provided stable 3.3 V output throughout testing, with 847.5 mA of headroom remaining above the calculated worst-case load. The ESP32-S3-WROOM-1-N8R8 booted reliably and maintained a persistent Wi-Fi and MQTT connection across all test sessions. Two-way MQTT messaging was fully operational, with telemetry published to the broker and motor commands received and forwarded over UART. The 64-byte structured UART packet format was implemented as specified in the API, with correct header/footer framing, board ID routing, and message type handling.

The WiFi connection-loss failsafe, listed as a stretch goal, was also successfully implemented. When the MQTT broker connection dropped, the firmware detected the disconnection and broadcast an emergency stop packet (message type `0x0005`) over UART to halt the actuator subsystem, satisfying the safe-stop behavior requirement.

### Stretch Goal: Video Data Handling

The video streaming requirement was the one area where software completion outpaced hardware verification. The full software stack was built: `camera_module.py` implements OV5640 initialization, JPEG capture, and chunked base64 MQTT frame publishing; `viewer.py` provides a laptop-side OpenCV viewer that reassembles and displays frames in real time; and `config_additions.py` defines the three camera MQTT topics (`TOPIC_CAM_CMD`, `TOPIC_CAM_FRAME`, `TOPIC_CAM_STATUS`). The architecture supports on-demand capture, continuous streaming at a configurable frame rate, and graceful stream start/stop via MQTT commands.

However, live end-to-end camera streaming was not demonstrated on the final populated PCB. The primary constraint was time available for hardware bring-up after board assembly. The OV5640 camera interface requires careful pin-by-pin verification against the schematic, and the MicroPython `esp32-camera` module requires firmware compiled with camera support enabled, both of which were not completed within the project timeline. This is discussed further in the [Hardware V2.0](../hardware-v2) and [Reflection](../reflection) pages.

---

## Feature-to-Requirement Rationale

Wireless communication is a critical part of the exploration device because it allows the robot to be operated from a safe distance while providing real-time situational awareness. Features such as remote driving, live video feedback, and telemetry monitoring directly resulted in requirements for reliable two-way Wi-Fi communication and structured UART messaging with the rest of the system.

Power-related requirements were included to ensure stable operation of the ESP32 and prevent communication failures caused by voltage drops. Connection-loss handling was added as a safety measure so the system can respond predictably if wireless control is interrupted. Together, these requirements ensure that the wireless module reliably supports the overall exploration device while operating safely within the system architecture.

The video streaming requirement originated from the core mission of the Amphibot: providing a live FPV feed for remote reconnaissance. Although this stretch goal was not fully demonstrated on hardware, its implementation informed several hardware decisions, most notably the selection of the ESP32-S3-WROOM-1-N8R8 over the N4 variant, as the 8 MB PSRAM is a hard dependency of the OV5640 camera driver and cannot be worked around in software.