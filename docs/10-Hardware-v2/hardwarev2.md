---
title: Hardware V2.0
tags:
- Mihir's Individual Datasheet - Hardware V2.0
---

# Hardware V2.0

If I were to design a second version of this board, there are several things I
would change based on what I learned during bring-up, integration, and the final
demonstration.

---

## Replace Micro USB with USB-C

The Micro USB connector worked fine for programming but it feels outdated and the
connector is more fragile than USB-C. A USB-C receptacle with the correct CC
resistors would be a straightforward swap that makes the board more durable during
repeated demo handling and more compatible with modern laptops that no longer ship
with Micro USB cables. The ESP32-S3 native USB differential pair would still be
used the same way, only the physical connector would change.

## Add a Dedicated Camera Power Rail

The OV5640 camera currently shares the main 3.3V rail with the ESP32, the debug
LEDs, and everything else on the board. During camera operation the current draw
spikes significantly. In a V2 design I would add a small secondary LDO, something
like the TLV70033, specifically for the camera module. This would isolate camera
noise from the ESP32 power rail and reduce the risk of brownouts during frame
capture. It would also make it easier to power-gate the camera when it is not in
use, which would help with overall power consumption.

## Fix the Camera Connector and Verify Pin Mapping

The J6 camera connector on V1 used a 2x9 pin header with a pin mapping derived
from the schematic. The mismatch between the schematic pin assignments and the
MicroPython esp32-camera library's expected pin ordering was one of the main
reasons the camera was not demonstrated. In V2 I would verify the exact pin
mapping required by the firmware library first, then design the connector and
schematic around that, instead of the other way around. I would also add a small
silkscreen label on the PCB for each camera pin to make bring-up easier.

## Add Decoupling Capacitors Closer to the ESP32 Module

The current design has decoupling capacitors on the 3.3V rail but they are not
placed as close to the ESP32 module pads as they should be. During WiFi
transmission the ESP32 draws current in short bursts and the decoupling needs to
be right at the module to be effective. In V2 I would place 100nF and 10uF
capacitors directly adjacent to the 3V3 and GND pins of the module following the
Espressif layout recommendations more strictly.

## Switch to a Higher Current Regulator

The AP63203WU-7 at 2A worked within the power budget, but the margin between the
worst-case load and the regulator limit was comfortable but not generous. If V2
adds a dedicated camera rail and any additional peripherals, upgrading to a 3A
regulator like the AP63205WU-7 in the same SOT-23-6 package would provide more
headroom without any layout changes.

## Add an LED for Camera Status

The current board has LEDs for MQTT activity, WiFi status, UART RX, and UART TX,
but nothing specifically for camera state. In V2 I would add a fifth indicator
LED on a dedicated GPIO to show whether the camera is actively streaming. During
demos this would make it immediately obvious whether the camera stream is running
without needing to check the laptop viewer.

## Improve the Antenna Keepout Zone Compliance

The ESP32-S3-WROOM-1 module requires a keepout area under the antenna section of
the module where no copper, traces, or components should be placed. The V1 layout
followed this generally but could be improved. In V2 I would be more careful about
enforcing the keepout on all layers including inner layers and the ground plane,
following the Espressif PCB design guidelines exactly.

## Add JTAG Debug Header

The current board relies entirely on USB serial for debugging. In V2 I would add
a small JTAG header broken out from the ESP32-S3 JTAG pins. This would allow
hardware-level debugging with OpenOCD if the firmware ever gets into a state where
USB serial is not responsive, which happened a few times during bring-up when the
firmware crashed before the USB stack initialized.