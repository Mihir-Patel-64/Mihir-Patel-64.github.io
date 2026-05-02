---
title: Welcome
tags:
- Mihir's Individual Datasheet - Welcome Page
---
<center>
<font size= "6"> Mihir Patel — Individual Datasheet </font><br>
as part of<br>
<font size= "8"> R6 Recon Amphibot (Amphibot V1) </font><br>
for<br>
<font size= "5"> Team 302 </font><br>

**Submission: May 02, 2026**
</center>

## Introduction

This datasheet documents the hardware and firmware design for my individual 
subsystem: the **Internet-based Two-way Wireless Communication module** 
(ESP32 Gateway) for the Amphibot V1. Its purpose is to record clear, testable 
requirements, show the block-level hardware layout, list parts and interfaces, 
and explain how my work plugs into the team-level system (UART daisy-chain, 
power bus, and MQTT/Wi-Fi). This page is intended for teammates, instructors, 
and graders so they can quickly understand what I built and how it integrates 
into the overall project.

---

## Project Summary

The Amphibot V1 is a small, throwable amphibious exploration robot that streams 
live FPV video and telemetry to a remote operator while reporting a consolidated 
hazard score. My module provides the bi-directional Wi-Fi gateway between the 
robot and the operator's device. It:

- Hosts a reliable Wi-Fi connection and communicates with an external MQTT broker.
- Translates UART traffic from the onboard modular bus into MQTT topics (and vice versa).
- Implements safe fallback behaviors on connection loss — broadcasting a motor 
  safe-stop command over UART when WiFi is lost.
- Was designed to support live FPV video streaming via the OV5640 camera module, 
  with full software infrastructure implemented.

This gateway is crucial for remote control, live telemetry, and demo display 
functions of the Amphibot. For the full team-level context, visit the 
[Team 302 Report Website](https://egr314-s-2026-302.github.io/EGR314-Team302.github.io/).

---

## My Contribution

I (Mihir Patel) own the Wireless Communication subsystem (ESP32). My responsibilities were:

- **Hardware:** Selected and integrated the ESP32-S3-WROOM-1-N8R8 module, the 
  AP63203WU-7 3.3V switching regulator, barrel jack and power-jumper circuitry, 
  and the 2×4 IDC ribbon connector interface.
- **Firmware:** Implemented Wi-Fi + MQTT client using the `mqtt_as` async library, 
  MQTT↔UART packet bridge, reconnection and WiFi-loss failsafe behaviors.
- **Camera infrastructure:** Designed and implemented `camera_module.py` for OV5640 
  frame capture and chunked MQTT publishing, and `viewer.py` for laptop-side 
  frame reassembly and display.
- **System integration:** Defined and implemented the 64-byte UART packet format, 
  board ID routing, and inter-board message handling across the daisy-chain.
- **Documentation:** Supplied the module datasheet, BOM, block diagram, schematic, 
  PCB design, and API specification.

---

## Project Sections

You can navigate the main sections of this individual datasheet using the top 
menu or the links below. Each section documents a specific stage of the design 
and implementation of the **Wireless Communication (ESP32) subsystem**.

- **[Project Requirements](01-Requirements/Requirements.md)** – Module-level 
  requirements, thresholds, target goals, and final results

- **[Block Diagram](02-Block-Diagram/Block-Diagram.md)** – Subsystem-level block 
  diagram showing power domains, UART daisy-chain connections, ESP32 peripherals, 
  and external interfaces

- **[Component Selection](03-Component-Selection/Component-Selection.md)** – 
  Selected components and design rationale for the ESP32 wireless gateway

- **[Microcontroller Selection](04-Microcontroller-Selection/Microcontroller-Selection.md)** – 
  ESP32 microcontroller selection criteria, comparisons, and rationale

- **[Power Budget](05-Power-Budget/Power-Budget.md)** – Power consumption analysis, 
  supply requirements, and power domain design

- **[Bill of Materials (BOM)](06-BOM/BOM.md)** – Manufacturer part numbers, 
  footprints, quantities, and estimated costs

- **[Schematic](07-Schematic/schematic.md)** – Electrical schematics for the 
  wireless communication module

- **[PCB Layout](08-PCB/pcb.md)** – PCB layout, routing decisions, and design 
  considerations

- **[API](09-API/api.md)** – Firmware behavior, UART message specification, and 
  MQTT topic documentation

- **[Hardware V2.0](10-Hardware-v2/hardwarev2.md)** – What I would improve in 
  a second hardware revision

- **[Resources](11-Resources/resources.md)** – Final code, downloadable files, 
  and project videos

- **[Reflection](12-Reflection/reflection.md)** – Lessons learned, startup tips, 
  and recommendations for future students