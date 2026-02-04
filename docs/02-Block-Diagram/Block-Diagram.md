---
title: Module's Block Diagram
tags:
- tag1
- tag2
---

## Overview
This needs to be updated with a brief purpose for having the block diagram.
Things to mention are:
* power levels
* sensor
* Actuator
* team connections
* Power source
* ...

To get some initial formatting help, one can view ["here"](https://embedded-systems-design.github.io/EGR304DataSheetTemplate/Appendix/basic-markdown-examples/) some basic techniques.


## Example Block Diagram 
Showing an example of how to import a screenshot of the block diagram created outside of git and brought into a page.

![Example of Indivial Block diagram ](individual-block-diagram.png)

```mermaid
flowchart TD
  %% Top cloud
  subgraph CLOUD[ ]
    direction TB
    MQTT[/"MQTT Server\n(Wi-Fi)"/]
  end

  %% Subsystem boundary
  subgraph SUBSYS["Mihir Patel — ESP32 Subsystem\nTeam 302 — R6 Recon Amphibot\n(Internet Two-way Wireless Gateway)"]
    direction TB
    style SUBSYS stroke-dasharray: 6 4, stroke:#888, fill:none

    %% Power blocks (left)
    BARREL[/"Barrel Jack Adapter\n(12V input)"/]
    PSU[/"3.3V Switching Regulator\n(3.3V, regulated, 1.5 A max)"/]
    BARREL -->|12 V DC| PSU

    %% 3.3V bus shown as horizontal rail
    PSU --- V3[/"3.3V Rail\n(regulated)"/]

    %% Main ESP32 block (center)
    subgraph MCU["ESP32 Wi-Fi Module"]
      direction TB
      ESP32BOX[["ESP32\n(MQTT Client)\nPublish / Subscribe\nUART (TX, RX)"]]
      ESP32BOX --- UART_PORT[/"UART\nTX, RX\nDigital - Serial (UART, 2 pins, 3.3V)"/]
      ESP32BOX --- USB[/"USB-C (ESP32 Programming)"/]
    end

    %% Connectors (top-left / top-right)
    CON_IN[/"Connector IN\n2×4 IDC (Upstream)\nUART (2 pins) + 3.3V sense"/]
    CON_OUT[/"Connector OUT\n2×4 IDC (Downstream)\nUART (2 pins) + 3.3V sense"/]

    %% Wiring between ESP32 and ribbon connectors (bidirectional UART)
    CON_IN -->|UART (2 pins,\nTX/RX, 3.3V)| UART_PORT
    UART_PORT -->|UART (2 pins,\nTX/RX, 3.3V)| CON_OUT

    %% Power rail to connectors (shared bus)
    V3 -->|3.3V power| CON_IN
    V3 -->|3.3V power| CON_OUT
    V3 -->|3.3V power| ESP32BOX

    %% Optional peripheral notes (placeholders for team)
    PERIPH_NOTE[/"(No camera on this board)\nPeripheral I/O reserved for\nUART diagnostics / status LEDs"/]

    %% USB programming
    ESP32BOX -->|USB serial| USB

    %% Links to outside cloud (Wi-Fi)
    ESP32BOX -->|Wi-Fi (MQTT)\nPublish / Subscribe| MQTT

  end

  %% External arrows / legend
  classDef perif stroke:#999,stroke-dasharray: 4 2;
  class PERIPH_NOTE perif;

  %% Legend block
  subgraph LEGEND["Legend"]
    direction LR
    L1["Solid line = wired connection"] 
    L2["Text on arrow = protocol / pin count / voltage"]
    L3["Dashed subsystem border = individual PCB boundary"]
  end
