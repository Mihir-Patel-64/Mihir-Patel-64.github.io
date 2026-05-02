---
title: Module Bill of Materials
tags:
- EGR314 
- Team 302 
---

## Overview

This Bill of Materials (BOM) lists every electronic component required for Mihir Patel’s ESP32-S3 Vision & Wireless Communication Subsystem for the R6 Recon Amphibot (Team 302, EGR314 Spring 2026).

It includes active components, passives, connectors, protection devices, and interface hardware required for PCB fabrication and subsystem integration into the UART daisy-chain architecture.

All components are documented with manufacturer information, part numbers, supplier details, datasheet links, and schematic reference designators to ensure full traceability and verification compliance.

Spare quantities are included where necessary to ensure reliability during assembly, soldering, and system integration testing.

---

## Bill of Materials  

*Table 01: Items below represent the complete BOM associated with this subsystem.*

### Bill of Materials — ESP32-S3 Vision & Wireless Subsystem (Team 302)

| **Part Name / Description** | **Qty** | **Unit Cost** | **Total Cost** | **Manufacturer** | **Manufacturer #** | **Vendor / Source** | **Datasheet / Product Link** | **Reference** |
|:----------------------------|:------:|:-------------:|:--------------:|:----------------:|:------------------:|:-------------------:|:----------------------------:|:------------:|
| ESP32-S3-WROOM-1-N8R8 (WiFi + BT, 8MB Flash, 8MB PSRAM, PCB Antenna) | 1 | $6.13 | $6.13 | Espressif Systems | ESP32-S3-WROOM-1-N8R8 | Digi-Key | [Datasheet](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf) | U2 |
| 3.3V 2A Buck Regulator (TSOT23-6, SMD) | 1 | $0.71 | $0.71 | Diodes Inc. | AP63203WU-7 | Digi-Key | [Datasheet](https://www.diodes.com/assets/Datasheets/AP63200-AP63201-AP63203-AP63205.pdf) | U1 |
| OV5640 Camera Breakout (5MP, DVP Interface) | 1 | $17.50 | $17.50 | Adafruit | 5841 | Digi-Key | [Datasheet](https://www.adafruit.com/product/5841) | J6 (Daughterboard) |
| Micro USB Type-B Vertical THT Connector | 1 | $0.78 | $0.78 | GCT | USB3131-30-0230-A | Digi-Key | [Datasheet](https://gct.co/files/specs/usb3131-spec.pdf) | J1 |
| 2.1mm x 5.5mm Barrel Jack (THT) | 1 | $0.59 | $0.59 | Same Sky (CUI) | PJ-102A | Digi-Key | [Datasheet](https://www.sameskydevices.com/product/resource/pj-102a.pdf) | J2 |
| Schottky Diode 40V 3A (SMC SMD) | 4 | $0.70 | $2.80 | onsemi | MBRS340T3G | Digi-Key | [Datasheet](https://www.onsemi.com/pdf/datasheet/mbrs340t3-d.pdf) | D4, D8 |
| Polyfuse PTC 2.6A Hold 16V (1812 SMD) | 3 | $4.6 | $13.80 | Littelfuse | 1812L260/16MR | Digi-Key | [Datasheet](https://www.littelfuse.com/assetdocs/littelfuse-fuse-154-series-data-sheet) | F1 |
| Fixed Inductor 6.8µH 285mA 1.2Ω (1812 SMD) | 2 | $0.28 | $0.56 | Bourns | PM1812-6R8J-RC | Digi-Key | [Datasheet](https://www.bourns.com/docs/Product-Datasheets/pm1812_series.pdf) | L1 |
| 22µF 25V X5R Ceramic Capacitor (1812 SMD) | 2 | $1.09 | $2.18 | TDK | C4532X5R1E226M250KA | Digi-Key | [Datasheet](https://product.tdk.com/system/files/dam/doc/product/capacitor/ceramic/mlcc/catalog/mlcc_commercial_general_en.pdf) | C5, C6 |
| 10µF 25V X5R Ceramic Capacitor (1812 SMD) | 3 | $1.12 | $3.36 | TDK | C4532X5R1E106M250KA | Digi-Key | [Datasheet](https://product.tdk.com/system/files/dam/doc/product/capacitor/ceramic/mlcc/catalog/mlcc_commercial_general_en.pdf) | C2, C7, C9 |
| 0.1µF 630V X5R Ceramic Capacitor (1812 SMD) | 4 | $0.77 | $3.08 | TDK | C4532X5R2J104K230KA | Digi-Key | [Datasheet](https://product.tdk.com/system/files/dam/doc/product/capacitor/ceramic/mlcc/catalog/mlcc_commercial_general_en.pdf) | C1, C3, C4, C8 |
| 10kΩ 1% 1W Resistor (1812 SMD) | 4 | $0.36 | $1.44 | KOA Speer | WK73R2JTTE1002F | Digi-Key | [Datasheet](https://www.koaspeer.com/pdfs/WK73R.pdf) | R1, R5, R11, R12 |
| 220Ω 1% 1W Resistor (1812 SMD) | 5 | $0.37 | $1.85 | Yageo | RC1218FK-07220RL | Digi-Key | [Datasheet](https://yageogroup.com/content/datasheet/asset/file/PYU-RC_GROUP_51_ROHS_L) | R2, R3, R7, R8, R9 |
| 470Ω 1% 1W Resistor (1812 SMD) | 2 | $0.22 | $0.44 | Yageo | RC1218FK-07470RL | Digi-Key | [Datasheet](https://yageogroup.com/content/datasheet/asset/file/PYU-RC_GROUP_51_ROHS_L) | R4, R6 |
| Green LED Diffused 0805 SMD | 5 | $0.15 | $0.75 | ams-OSRAM | LG R971-KN-1-0-20-R18 | Digi-Key | [Datasheet](https://look.ams-osram.com/m/2da6f2ceeebf8fac/original/LG-R971.pdf) | D2–D7 |
| Tactile Switch SPST-NO SMD | 2 | $0.22 | $0.44 | CTS | 222AMVBAR | Digi-Key | [Datasheet](https://www.ctscorp.com/Files/DataSheets/Switches/Tactile-Switches/CTS-Switches-Tactile-222A-Series-Datasheet.pdf) | SW1, SW2 |
| Shunt Jumper 1.27mm Pitch | 4 | $0.16 | $0.64 | Adam Tech | HMSC-G | Digi-Key | [Datasheet](https://app.adam-tech.com/products/download/data_sheet/203317/hmsc-g-data-sheet.pdf) | J6 (Jumper) |
| 1×2 Pin Header 2.54mm THT | 1 | $0.00 | $0.00 | Various | PinHeader_1x02_2.54mm | Peralta | — | J4 |
| 2×4 Pin Header 2.54mm THT | 4 | $0.00 | $0.00 | Various | PinHeader_2x04_2.54mm | Peralta | — | J5, J7, J8, J9 |
| 2×9 Pin Header 2.54mm THT (Camera Connector) | 1 | $0.00 | $0.00 | Various | PinHeader_2x09_2.54mm | Peralta | — | J6 Header |
| Test Point | 8 | $0.00 | $0.00 | Various | Test Point | Peralta | — | T1, T2, T3, T4, T5, T6, T7, T8 |
| **Estimated Total Cost** |  |  | **$57.05** |  |  |  |  |  |
| **Tax** |  |  | **$4.71** |  |  |  |  |  |
| **Shipping Charges** |  |  | **$6.99** |  |  |  |  |  |
| **Tariff** |  |  | **~$1.07** |  |  |  |  |  |
| **Final TOTAL Cost** |  |  | **$69.82** |  |  |  |  |  |