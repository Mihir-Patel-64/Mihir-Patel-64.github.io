---
title: Reflection
tags:
- Mihir's Individual Datasheet - Reflection
---

# Reflection

## Review of Module's Success

Looking back at the six requirements I defined for this subsystem, four core
requirements were fully met, one stretch goal was fully met, and one stretch
goal was only partially met.

The regulator worked exactly as expected. The AP63203WU-7 produced stable 3.3V
output throughout testing with plenty of headroom above the worst-case load. The
ESP32 booted reliably every session and held its MQTT connection without needing
manual resets. Two-way MQTT messaging worked end to end, telemetry received over
UART was published to the broker as JSON, and motor commands from the broker were
correctly parsed and forwarded as 64-byte UART packets to the actuator board. The
packet format was implemented as specified, with correct framing, board ID
routing, rate-limited transmission, and payload sanitization.

The WiFi failsafe stretch goal was also fully achieved. When the broker connection
dropped, the firmware detected it through the wifi callback, stopped any active
camera stream, broadcast an emergency stop packet over UART, and flashed all four
LEDs to signal the fault. This was tested and confirmed during development.

The one thing that did not get demonstrated was the camera. The full software
stack was built, camera_module.py handles OV5640 initialization and chunked MQTT
frame publishing, viewer.py reassembles and displays frames on a laptop using
OpenCV, and the three camera MQTT topics are defined in config_additions.py. The
issue was that standard MicroPython does not include the esp32-camera module. It
requires a custom firmware build with camera support compiled in, and getting that
build set up was not completed within the project timeline. The software was ready
but the hardware side of the camera bring-up was not finished in time.

---

## Microcontroller / Module Startup Tips

These are things I either found helpful, wish I had known earlier, or wish I had
followed more carefully:

1. Flash MicroPython with the right build from the start. Standard MicroPython
does not include the camera module. If your project involves the OV5640, you need
a custom firmware build before you write any camera code. Finding this out late
in the project is what caused the camera demo to be missed.

2. Test UART with a USB-to-serial adapter before connecting to teammates. Looping
back to yourself first confirms your TX and RX pins are correct and saves a lot
of confusion when things do not work in a multi-board setup.

3. The ESP32-S3 has two USB interfaces and they are not interchangeable. The
native USB on GPIO19/20 and the UART0 debug port are different. If you flash
via native USB and open a serial monitor expecting UART0 output, you will see
nothing. Make sure your flash tool and serial monitor are pointed at the same port.

4. Hold BOOT before pressing EN when entering download mode. This sounds obvious
but in a busy lab it is easy to forget. If esptool is not recognizing the chip,
this is almost always why.

5. Use mqtt_as instead of a basic MQTT library. It handles reconnection,
keep-alives, and clean sessions automatically. A blocking MQTT client will lock
up the entire firmware when the broker drops, which makes a WiFi failsafe nearly
impossible to implement.

6. Rate-limit UART transmissions. The ESP32 and PIC boards do not have the same
buffer sizes. Sending packets too fast dropped bytes on the receiving end during
early testing. The 500ms minimum interval between sends was necessary for
reliable communication.

7. Verify GPIO assignments against the actual datasheet pin table, not just the
module pin numbers. GPIO4 is not Pin 4 on the WROOM module. Cross-checking the
pin allocation table against Table 3-1 in the datasheet before routing the
schematic prevents a lot of errors.

8. Put test points on every signal you care about before ordering the PCB. They
were invaluable during bring-up. Any signal you might want to probe should have
a test point. It costs nothing in PCB area and saves significant time later.

9. The antenna keepout zone is not optional. The WROOM module has a defined
keepout area under the antenna. Placing copper there degrades RF performance.
Follow the datasheet layout guidelines exactly.

10. Start firmware development before the hardware is ready. The entire firmware
stack was developed and tested on a dev board before the custom PCB arrived. When
the PCB was assembled, there was already working firmware ready to flash. Starting
early is one of the best things you can do for your timeline.

---

## Lessons Learned

1. The gap between software-ready and hardware-verified is real. The camera
software was fully implemented but never verified on the final PCB. In embedded
systems a feature is not done until it runs on the actual target hardware.
Software completion and hardware demonstration are two different milestones and
both need time planned for them.

2. Component selection decisions have downstream consequences that are not always
obvious. Choosing the N8R8 over the N4 was a one dollar decision that made the
entire camera subsystem possible. If that choice had not been made early, the
OV5640 breakout would have been a component that could never work. Understanding
the full dependency chain before ordering parts matters a lot.

3. Async firmware architecture is worth the learning curve. Using uasyncio with
mqtt_as meant that UART polling, MQTT communication, heartbeat publishing, and
camera streaming could all run concurrently. A blocking approach would have made
the WiFi failsafe nearly impossible to implement cleanly.

4. Structured packet formats need to be agreed on and frozen early. The 64-byte
UART format worked well but there were early iterations where teammates had
different expectations. Any ambiguity in the protocol causes integration failures
that are hard to debug across boards. Lock it down in writing before anyone
starts coding.

5. Power budget analysis directly informs hardware decisions. Working through the
numbers confirmed that the AP63203WU-7 had adequate headroom for simultaneous
WiFi and camera operation. Without doing the math there would have been no way
to know if the regulator was actually sufficient.

6. PCB bring-up is its own phase of the project and needs dedicated time. Assembly,
soldering, and initial power-on each take longer than expected. The timeline needs
to budget for bring-up explicitly, not treat it as something that happens
automatically after the board arrives.

7. Debug hardware on the PCB is worth the effort. The five debug LEDs gave
immediate visual confirmation of UART RX, TX, MQTT activity, and WiFi status
during bring-up. Without them, figuring out whether a problem was in the firmware,
the wiring, or the MQTT connection would have required an oscilloscope every time.

8. Schematic net label naming should match firmware variable names. The USB
differential pair was labeled USB_N and USB_P in the schematic but referenced as
GPIO19 and GPIO20 everywhere else. That inconsistency caused confusion during
review. Keeping names consistent across the schematic, firmware, and datasheet
reduces errors.

9. Testing the failsafe explicitly is not optional. It would have been easy to
assume the WiFi failsafe worked without testing it. Actually verifying that the
emergency stop packet was broadcast over UART when the broker dropped gave real
confidence that the feature worked.

10. Documentation written during the design process is far better than
documentation written after. Pages written while design decisions were fresh were
more detailed and accurate than sections written later from memory. Keeping the
datasheet updated as decisions were made made the final report much easier to
complete.

---

## Recommendations for Future Students

1. Read the full assignment rubric before starting any design work. The final
report adds new required sections that are not in the early checkpoints. Knowing
what is coming from the beginning means you can structure your work to satisfy
those requirements as you go rather than adding missing sections at the end.

2. Get your development environment working on a commercial dev board before your
custom PCB arrives. Flash MicroPython onto an off-the-shelf ESP32-S3 dev board
and verify that WiFi, UART, and other peripherals work in software first. That
way you are debugging firmware and hardware separately, not at the same time.

3. Write down the UART protocol with your teammates before anyone starts coding.
Verbal agreements about packet formats lead to integration failures that are hard
to debug under time pressure. A shared document that everyone has reviewed and
agreed on is worth the thirty minutes it takes to write.

4. Order your PCB and components earlier than you think you need to. Shipping
delays and fabrication lead times are real. If your board arrives two weeks before
the showcase instead of four, you lose half your bring-up time. Build buffer into
your schedule deliberately.

5. Treat stretch goals as real goals with real schedules. The camera had a full
software stack built but was never demonstrated because the hardware bring-up
steps for the camera were not scheduled explicitly. If a feature matters enough
to implement, it matters enough to plan a verification timeline for it.