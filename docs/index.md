---
title: Welcome
tags:
- ESP32 Wireless Communication Gateway - Welcome Page
---
<center>

<font size="6">ESP32 Wireless Communication Gateway</font><br>

**Mihir Patel**<br><br>

<font size="8">Team XPED - R6 Recon Amphibot (Amphibot V1)</font><br>

<font size="5">EGR314 Embedded Systems Design • Team 302</font>

</center>

## Introduction

This documentation presents the design and implementation of my **ESP32 Wireless Communication Gateway** for the Team XPED R6 Recon Amphibot. The gateway enables reliable wireless communication between the robot and a remote operator while integrating with the robot's modular UART-based architecture.

The documentation covers subsystem requirements, hardware architecture, component selection, firmware implementation, PCB design, communication interfaces, verification, and engineering decisions. It also explains how the subsystem integrates with the overall robotic platform and supports reliable operation during remote exploration missions.

---

## Project Overview

The **R6 Recon Amphibot** is a compact amphibious exploration robot designed for remote reconnaissance in challenging environments. The system streams live FPV video, transmits sensor telemetry, and allows an operator to remotely monitor and control the robot while evaluating environmental hazards.

The **ESP32 Wireless Communication Gateway** serves as the primary communication bridge between the robot and the operator. Its responsibilities include:

- Maintaining a reliable Wi-Fi connection with an external MQTT broker.
- Translating UART packets from the modular communication bus into MQTT topics and vice versa.
- Broadcasting a motor safe-stop command over the UART bus whenever wireless communication is lost.
- Providing the software infrastructure required for live FPV video streaming using the OV5640 camera module.
- Supporting real-time telemetry, command routing, and remote system monitoring.

The gateway provides the communication backbone that enables remote operation and coordination between every subsystem in the Amphibot.

For additional system-level documentation, visit the **Team XPED Project Website**:

**https://egr314-s-2026-302.github.io/EGR314-Team302.github.io/**

---

## My Contributions

I was responsible for the design and implementation of the **ESP32 Wireless Communication Gateway** subsystem.

My responsibilities included:

### Hardware Design

- Designed the subsystem around the **ESP32-S3-WROOM-1-N8R8** microcontroller.
- Selected and integrated the **AP63203WU-7** 3.3 V switching regulator.
- Designed the barrel-jack power input and power-jumper circuitry.
- Integrated the **2×4 IDC ribbon connector** for the modular UART communication bus.

### Firmware Development

- Implemented Wi-Fi connectivity and MQTT communication using the **mqtt_as** asynchronous library.
- Developed the MQTT-to-UART communication bridge.
- Implemented automatic reconnection logic and Wi-Fi loss failsafe behavior.
- Developed UART packet routing and subsystem message handling.

### Camera Infrastructure

- Developed `camera_module.py` for OV5640 image capture and MQTT-based frame transmission.
- Developed `viewer.py` for frame reconstruction and live image display on the operator's computer.

### System Integration

- Designed the 64-byte UART communication protocol.
- Implemented board identification and packet routing across the UART daisy-chain.
- Integrated the wireless gateway with the complete Team XPED communication architecture.

### Documentation

- Produced subsystem requirements.
- Created hardware block diagrams.
- Documented component selection decisions.
- Developed schematic and PCB documentation.
- Documented firmware APIs and communication interfaces.
- Performed subsystem verification and validation.

---

## Documentation Overview

The following sections describe each stage of the subsystem design and implementation.

| Section | Description |
|---------|-------------|
| **[Requirements](01-Requirements/Requirements.md)** | Functional and performance requirements, design objectives, and verification results. |
| **[Block Diagram](02-Block-Diagram/Block-Diagram.md)** | Hardware architecture showing power domains, UART interfaces, ESP32 peripherals, and external connections. |
| **[Component Selection](03-Component-Selection/Component-Selection.md)** | Component evaluation and selection rationale for the wireless communication subsystem. |
| **[Microcontroller Selection](04-Microcontroller-Selection/Microcontroller-Selection.md)** | Comparison of candidate microcontrollers and justification for selecting the ESP32-S3. |
| **[Power Budget](05-Power-Budget/Power-Budget.md)** | Power consumption analysis, regulator selection, and power distribution. |
| **[Bill of Materials](06-BOM/BOM.md)** | Complete list of components, manufacturers, footprints, quantities, and estimated costs. |
| **[Schematic](07-Schematic/schematic.md)** | Electrical schematic of the ESP32 Wireless Communication Gateway. |
| **[PCB Layout](08-PCB/pcb.md)** | PCB layout, routing decisions, and hardware design considerations. |
| **[API Documentation](09-API/api.md)** | UART protocol, MQTT topics, firmware interfaces, and software architecture. |
| **[Hardware Revision 2.0](10-Hardware-v2/hardwarev2.md)** | Proposed improvements for a future hardware revision. |
| **[Resources](11-Resources/resources.md)** | Source code, downloadable project files, and demonstration videos. |
| **[Reflection](12-Reflection/reflection.md)** | Engineering lessons learned, design reflections, and recommendations for future development. |
