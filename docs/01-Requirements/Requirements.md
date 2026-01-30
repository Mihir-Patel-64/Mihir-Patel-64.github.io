---
title: Module's Requirements
tags:
- EGR314
- Team XPED
---

# Project Requirements

The purpose of this section is to define the engineering requirements for the **Wireless Communication subsystem (ESP32)**. This module is responsible for enabling remote control of the exploration device, transmitting telemetry and video data to the user, and ensuring safe system behavior if the wireless connection is lost. Each requirement below describes the minimum functionality required for the module to be considered operational, along with target performance goals and optional stretch objectives.

---

## Module Requirements — Wireless Communication (ESP32)  
**Teammate: Mihir Patel**

| Requirement Description | Measure of Threshold | Target Measure | Stretch (Yes/No) |
|------------------------|------------------|-------------|------------------|
| Surface-mounted 3.3 V switching regulator | Output ≥ 3.2 V | Stable 3.3 V output | No |
| ESP32 microcontroller operation | Boots successfully | Runs Wi-Fi and MQTT | No |
| Wireless communication | One-way data transfer | Two-way MQTT messaging | No |
| UART communication with system bus | Basic TX/RX | Structured packets with CRC | No |
| Video data handling | Video stream present | Stable low-latency stream | Yes |
| Connection loss handling | Detect disconnection | Trigger safe-stop behavior | Yes |

---

## Feature-to-Requirement Rationale

Wireless communication is a critical part of the exploration device because it allows the robot to be operated from a safe distance while providing real-time situational awareness. Features such as remote driving, live video feedback, and telemetry monitoring directly resulted in requirements for reliable two-way Wi-Fi communication and structured UART messaging with the rest of the system.

Power-related requirements were included to ensure stable operation of the ESP32 and prevent communication failures caused by voltage drops. Connection-loss handling was added as a safety measure so the system can respond predictably if wireless control is interrupted. Together, these requirements ensure that the wireless module reliably supports the overall exploration device while operating safely within the system architecture.
