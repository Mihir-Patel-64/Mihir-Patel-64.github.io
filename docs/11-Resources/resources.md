---
title: Resources
tags:
- Mihir's Individual Datasheet - Resources
---

# Resources

This page contains all final code files and downloadable project assets for the
ESP32 Wireless Communication subsystem.

---

## Final Code

The complete firmware for this subsystem consists of four files. All four are
included in the ZIP download below.

**mihir_esp32_main.py** — Main firmware file. Handles WiFi connection, MQTT
publish and subscribe, UART packet reception and forwarding, LED indicators,
periodic sensor polling, and the WiFi-loss emergency stop failsafe.

**camera_module.py** — Camera subsystem module. Handles OV5640 initialization,
single-frame JPEG capture, chunked base64 MQTT frame publishing, and continuous
streaming mode with configurable frame rate.

**config_additions.py** — Defines the three camera-specific MQTT topics that
are added to the existing config.py file: TOPIC_CAM_CMD, TOPIC_CAM_FRAME, and
TOPIC_CAM_STATUS.

**viewer.py** — Laptop-side viewer script. Subscribes to TOPIC_CAM_FRAME over
MQTT, reassembles chunked JPEG frames, and displays them in a live OpenCV window.
Run this on your laptop, not on the ESP32.

### Download

[Download Final ESP32 Code ZIP](Mihir_ESP32_Code_Final.zip)

---

## Schematic and PCB Files

These are also available on the Schematic and PCB pages but are collected here
for convenience.

- [Download KiCad Project ZIP](Patel_EGR314_projectzip_4_11.zip)
- [Download PCB Gerber Files ZIP](Gerber_drill_4_10_26.zip)
- [Download High-Resolution Schematic PDF](Patel_EGR314_Schematic_MP_4_11.pdf)

---

## 3D Printed Parts

The team designed and printed a half-cut body model of the R6 Recon Amphibot
for the Innovation Showcase. This was not a functional enclosure but rather a
display model to help visitors understand how the internal electronics would be
arranged inside the chassis, where each board would sit, and how the overall
robot would look when assembled. It gave people at the showcase a physical
reference for the project without needing to see the actual robot fully built.