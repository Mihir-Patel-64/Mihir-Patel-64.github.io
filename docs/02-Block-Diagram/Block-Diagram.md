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

## Block Diagram — Wireless Communication Subsystem (ESP32)

```mermaid
flowchart TB

  %% Cloud / Network
  MQTT[/"MQTT Server\n(Wi-Fi)"/]

  %% Subsystem Boundary
  subgraph SUBSYS["Mihir Patel — ESP32 Wireless Gateway\nTeam 302 · R6 Recon Amphibot"]
    style SUBSYS stroke-dasharray: 6 4

    %% Power Section
    BARREL["DC Barrel Jack\n(9–12 V DC)"]
    JUMPER["Power Jumpers\n(Local / Bus Select)"]
    REG["3.3 V Switching Regulator\n(3.3 V, 1.5 A max)"]

    BARREL --> JUMPER --> REG

    %% ESP32 Block
    subgraph ESP["ESP32 Wi-Fi Module"]
      UART["UART\nTX, RX\nDigital-Serial (2 pins, 3.3 V)"]
      WIFI["Wi-Fi\nMQTT Client\n(Publish / Subscribe)"]
      USB["USB\nProgramming / Serial"]
      GPIO["GPIO\nStatus / Debug"]
    end

    REG -->|3.3 V| ESP

    %% Ribbon Connectors
    IN["Connector IN\n2×4 IDC (Upstream)"]
    OUT["Connector OUT\n2×4 IDC (Downstream)"]

    IN <--> |UART (2 pins, 3.3 V)| UART
    UART <--> |UART (2 pins, 3.3 V)| OUT

    REG -->|3.3 V Bus| IN
    REG -->|3.3 V Bus| OUT

    %% USB Programming
    USBPORT["USB Port\n(ESP32 Programming)"]
    USBPORT --> USB

  end

  %% Wireless Link
  WIFI -.-> |Wireless (Wi-Fi)| MQTT
