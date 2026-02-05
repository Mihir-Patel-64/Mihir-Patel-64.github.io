---
title: Module's Block Diagram
tags:
- tag1
- tag2
---

## Overview

This block diagram represents the **ESP32-based Internet Two-way Wireless Communication subsystem** developed by **Mihir Patel** for **Team 302 – R6 Recon Amphibot**. The purpose of this subsystem is to act as the system’s wireless gateway, enabling remote communication with the operator while coordinating data exchange between the onboard subsystems using a UART daisy-chain architecture.

The diagram follows the EGR314 block diagram template and clearly separates this individual subsystem from the rest of the system using a dashed boundary. All major electrical components, communication interfaces, and power paths are shown with labeled signal directions, pin counts, and voltage levels.

---

### Subsystem Functionality

The ESP32 Wireless Gateway performs three primary functions within the overall exploration device:

1. **Two-Way Wireless Communication**  
   The ESP32 connects to an external MQTT server over Wi-Fi. Telemetry, system status, and debug information are published to defined MQTT topics, while control commands are received through subscribed topics. This satisfies the team requirement for duplex wireless communication using MQTT.

2. **UART Daisy-Chain System Interface**  
   The ESP32 communicates with the Sensor/HMI and Actuator subsystems through a standardized UART daisy-chain. Incoming and outgoing UART signals are routed through upstream and downstream **2×4 IDC connectors**, allowing structured messages to pass safely across all boards without direct coupling between subsystems.

3. **Local Debug and Status Indication**  
   Multiple GPIO-driven debug LEDs provide visual feedback for MQTT activity and UART transmission/reception. These indicators help with system bring-up, debugging, and demonstration clarity during lab testing and the Innovation Showcase.

---

### Power Architecture

Power is supplied to the subsystem through an external **12 V AC-DC wall adapter** connected via a **DC barrel jack**. This input feeds an onboard **3.3 V switching regulator rated at 1.5 A**, which provides regulated power to the ESP32 module. The regulated 3.3 V rail is also distributed to the system bus through the IDC connectors, supporting downstream modules when needed. All power connections are labeled with voltage levels and current capability in the diagram.

---

### Programming and Development Interface

The ESP32 is programmed and debugged locally using a **USB programming interface**, shown as a dedicated connection between the ESP32 module and the host computer. This interface is used exclusively for firmware upload and serial debugging and is intentionally kept separate from the UART system bus.

---

### Camera Integration

An onboard **ESP32-CAM / OV2640 camera module** is included within the ESP32 subsystem. The camera communicates internally with the ESP32 through the module’s native camera interface and does not use UART, SPI, or I²C buses. Captured video data is streamed wirelessly over Wi-Fi as part of the system’s live feedback capability, without affecting inter-board communication.

---

### Design Rationale and Standards Compliance

This block diagram meets the EGR314 requirements by:

- Clearly identifying the **individual subsystem boundary**
- Showing all **microcontroller peripherals** used (UART, GPIO, USB)
- Labeling **signal types, directions, voltages, and pin counts**
- Including both **upstream and downstream 2×4 IDC connectors**
- Separating **wireless communication** from wired system buses
- Supporting a **modular, standards-based system architecture**

Isolating wireless communication and system coordination within the ESP32 gateway reduces complexity in other subsystems and improves reliability, scalability, and ease of debugging.

---


## Block Diagram

![ESP32 Wireless Gateway Block Diagram](EGR314_team302_blockdiagram_Mihir.drawio.png)