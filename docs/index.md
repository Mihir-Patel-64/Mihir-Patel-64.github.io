---
title: Welcome
tags:
- tag1
- tag2
---
<center>
<font size= "6"> Mihir Patel — Individual Datasheet </font><br>
as part of<br>
<font size= "8"> R6 Recon Amphibot (Amphibot V1) </font><br>
for<br>
<font size= "5"> Team 302 </font><br>

**Submission: April, 03, 2026**
</center>

## Introduction

This datasheet documents the hardware and firmware plan for my individual subsystem: the **Internet-based Two-way Wireless Communication module** (ESP32 Gateway) for the Amphibot V1. Its purpose is to record clear, testable requirements, show the block-level hardware layout, list parts and interfaces, and explain how my work plugs into the team-level system (UART daisy-chain, power bus, and MQTT/Wi-Fi). This page is intended for teammates, instructors, and graders so they can quickly understand what I will build and how it integrates into the overall project.


### Project Summary

The Amphibot V1 is a small, throwable amphibious exploration robot that streams live FPV video and telemetry to a remote operator while reporting a consolidated hazard score. My module provides the bi-directional Wi-Fi gateway between the robot and the operator's device. It will:

- Host a reliable Wi-Fi connection and talk to an external MQTT broker (or local broker during demos).
- Translate UART traffic from the onboard modular bus into MQTT topics (and vice versa).
- Stream low-latency FPV video (target < 500 ms) to the operator app.
- Implement safe fallback behaviors on connection loss (motor safe-stop command via UART).

This gateway is crucial for remote control, live telemetry, and demo display functions of the Amphibot.

* Add context that ties into the link to your [team report.](https://egr314-s-2026-302.github.io/EGR314-Team302.github.io/)

### My Contribution

I (Mihir Patel) own the Wireless Communication subsystem (ESP32). My responsibilities are:

- Hardware: select and integrate the ESP32 module, the 3.3 V switching regulator, barrel jack and power-jumper circuitry, and the 2×4 IDC ribbon connector interface.
- Firmware: implement Wi-Fi + MQTT client, MQTT↔UART packet bridge with CRC checking, reconnection and fail-safe behaviors, and a basic web/socket interface for debugging.
- FPV stream handling: configure/stream camera video (ESP32-CAM or serial camera) and optimize to achieve target latency.
- System integration: define and verify UART packet formats, work with teammates to map ribbon pins, and ensure safe behavior when the wireless link is lost.
- Documentation & verification: supply the module datasheet, BOM, block diagram, and verification test log for integration.

### Project Sections

You can navigate the main sections of this individual datasheet using the top menu or the links below. Each section documents a specific stage of the design and implementation of the **Wireless Communication (ESP32) subsystem**.

- **[Project Requirements](01-Requirements/Requirements.md)** – Module-level requirements, thresholds, target goals, and stretch objectives  

- **[Block Diagram](02-Block-Diagram/Block-Diagram.md)** – Subsystem-level block diagram showing power domains, UART daisy-chain connections, ESP32 peripherals, and external interfaces  

- **[Component Selection](03-Component-Selection/Component-Selection.md)** – Selected components and design rationale for the ESP32 wireless gateway  

- **[Bill of Materials (BOM)](04-BOM/BOM.md)** – Manufacturer part numbers, footprints, quantities, and estimated costs  

- **[Schematic](05-Schematic/schematic.md)** – Electrical schematics for the wireless communication module  

- **[PCB Layout](06-PCB/pcb.txt)** – PCB layout, routing decisions, and design considerations  

- **[Reflection](07-Reflection/Reflection.txt)** – Lessons learned, challenges faced, and design insights from this subsystem
