---
title: Module's Power Budget
tags:
- Mihir's Individual Datasheet - Power Budget
---

# Power Budget

This page presents the Power Budget for Mihir Patel’s ESP32 Wireless Communication subsystem, verifying that all voltage rails, regulators, and the selected external power supply can reliably meet the total current demand of every major component in the design.

---

## Overview

The Power Budget includes:

- Power requirements of all major active components.
- Assigned voltage rail and total current consumption.
- Regulator and external source selections with 25% safety margins.

This subsystem operates primarily from a single regulated +3.3V rail derived from a 12V wall adapter.

---

## Power Budget Table

![Power Budget Table](PowerBudget_MP.jpg)

**Figure 01:** ESP32 Wireless Communication Subsystem Power Budget

---

### How the Power Budget Informed Design Choices

- The power budget was used to total the worst-case current draw of all major active components on the +3.3V rail, including the ESP32-S3-WROOM-1-N4 microcontroller, OV2640 camera module, USB-UART interface (if populated), debug LEDs, and supporting logic overhead. By summing each device’s peak current and adding a 25% safety margin, the spreadsheet confirmed a maximum required load of approximately **1.19 A** on the +3.3V rail.

- This requirement directly informed the regulator selection. A TPS62840 switching regulator rated for **1.5 A** was selected to provide adequate overhead above the 1.19 A requirement. This ensures stable operation during Wi-Fi transmission spikes and camera streaming bursts without approaching the regulator’s limits.

- The same budget was used to verify that the external +12V wall adapter can supply the regulator without being overstressed. With an estimated input current of approximately **0.36 A** at 12V (after accounting for conversion efficiency), and a 12V / 2A adapter selected, more than **1.6 A of headroom** remains available for startup transients or additional subsystem loads.

- From these calculations, the key conclusion is that a single +3.3V regulated rail powered from a +12V wall adapter through a high-efficiency buck regulator is sufficient for all wireless, camera, and communication functions in Mihir’s subsystem. No additional voltage rails are required.

---

### Downloadable Files

- **Power Budget EXCEL:**  
[Download Power Budget (Excel)](PowerBudget_MP.xlsx)

- **Power Budget ZIP:**  
[Download Power Budget (ZIP)](PowerBudget_MP.xlsx.zip)

- **Power Budget PDF:**  
[Download Power Budget (PDF)](PowerBudget_MP.pdf)

---

## Power Rails and Regulators

| **Power Rail** | **Regulator / Source** | **Part Number** | **Output Voltage** | **Max Current (mA)** | **Notes** |
|----------------|------------------------|------------------|--------------------|----------------------|------------|
| +3.3V | Switching Buck Regulator | TPS62840 | +3.3V | 1500 | Powers ESP32, camera, USB interface, debug LEDs |

> All major components, including the ESP32-S3-WROOM-1-N4, OV2640 camera module, CP2102N USB-UART interface (if used), and debug LEDs, are powered from a single +3.3V regulated supply. This simplifies distribution, reduces conversion losses, and improves system stability.

---

## Power Source Verification

| **Power Source** | **Part Number** | **Output** | **Max Current (mA)** | **Used (mA)** | **Remaining (mA)** |
|------------------|----------------|-----------|---------------------|---------------|-------------------|
| Wall Supply | 12V DC Adapter | +12V, 2A | 2000 | 363 | 1637 |

> The +12V adapter feeds the TPS62840 buck regulator, supplying the entire +3.3V rail with significant overhead.

> No battery supply is required for this subsystem.

---

### Power Budget Summary

This power budget directly accounts for the electrical demands of all wireless and communication components, assigning each to a single +3.3V logic rail. Current estimates were based on worst-case peak consumption and include a 25% safety margin as required by the design review guidelines.

The selected TPS62840 regulator exceeds the required current with safe overhead, and the 12V / 2A wall adapter provides more than sufficient capacity to support continuous operation, Wi-Fi transmission bursts, and camera streaming activity.

No additional voltage rails or battery supply are necessary for normal operation. The chosen power architecture simplifies the system while maintaining appropriate electrical headroom and thermal safety margins.
