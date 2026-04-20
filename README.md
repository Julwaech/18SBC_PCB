# 18SBC PCB

18-Series Battery Connector Board — a high-current PCB designed for 18S lithium battery pack integration in heavy-lift UAV systems.

## Overview

The 18SBC is a battery connector board that interfaces 18S smart battery packs (e.g. Tattu 4.0) with the aircraft electrical system.

### Design Goals

- Support 18S smart batteries (Tattu 4.0 compatible)
- Handle 100A continuous / 200A spike current
- Provide regulated and isolated 12V and 5V outputs to the main system PCB (5V and 12V can share GND)
- Active power switching with precharge and emergency shutoff
- Dual CAN bus communication (battery protocol + DroneCAN)
- Wireless connectivity (WiFi / BLE) for diagnostics and configuration

## Specifications

| Parameter | Value |
|---|---|
| Configuration | 18S (18 series cell groups) |
| Max Voltage | 75.6V (4.2V x 18) |
| Nominal Voltage | 64.8V (3.6V x 18) |
| Continuous Current | 100A |
| Spike Current | 200A (short duration) |
| Board Shape | Rectangular |
| EDA Tool | KiCad 10 |
| Board Layers | TBD |

### Current Rating Justification

Based on the motor data (Hobbywing XRotor X15):

| Throttle | Thrust | Current per Motor |
|---|---|---|
| 51% | 27,257 g | 44.7 A |
| 72% | 47,933 g | 101.6 A |
| 100% | 72,647 g | 197.1 A |

The Hover throttle is ~50%. The 100A nominal rating covers normal operations; the 200A spike rating handles full-throttle bursts.

## Battery Connector

**Selected: [Prolanv EN60A](https://www.prolanv.com/2_2552837_5851637.html)**

| Parameter | Value |
|---|---|
| Current per Power Pair | 60A |
| Power Pairs | 5 positive + 5 negative |
| Total Current Capacity | 300A |
| Signal Pins | 6 |
| Contact Resistance | 0.6 mOhm |
| Mating Cycles | 10,000 |
| Temperature Range | -40C to +125C |
| Mounting | Right-Angle DIP (PCB mount) |

The EN60A provides 300A total capacity across 5 power pairs, giving comfortable headroom above the 200A spike requirement. From the 6 signal pins, only 4 are used.

At 200A spike across 4 power pairs (50A each): P = I^2 x R = 50^2 x 0.0006 = 1.5W per pair = 6W total — well within thermal limits.

## MCU and Communication

### MCU: ESP32-C3-MINI-1

The ESP32-C3-MINI-1 module is a compact RISC-V based microcontroller with integrated wireless connectivity. It includes a PCB antenna — no external antenna components are required. A keepout zone must be maintained around the antenna area on the PCB layout (no copper / ground plane underneath).

| Parameter | Value |
|---|---|
| Core | 32-bit RISC-V, 160 MHz |
| Flash | 4 MB (integrated) |
| SRAM | 400 KB |
| WiFi | 802.11 b/g/n (2.4 GHz) |
| Bluetooth | BLE 5.0 |
| GPIOs | 22 |
| SPI | 3x (1 used for dual MCP2515) |
| ADC | 6 channels, 12-bit |
| Operating Voltage | 3.0 – 3.6V |
| Deep Sleep Current | 5 uA |
| Antenna | Integrated PCB antenna |

### Dual CAN Bus (2x MCP2515 + Transceiver)

Two independent CAN networks using external MCP2515 controllers on a shared SPI bus. This provides identical timing behavior on both buses and clean separation of the battery-side and drone-side CAN domains.

| CAN Bus | Purpose | Controller | Transceiver |
|---|---|---|---|
| CAN 1 — Battery | Battery protocol (smart battery communication) | MCP2515 (SPI, CS1) | SN65HVD230 (3.3V) |
| CAN 2 — Drone | DroneCAN (UAVCAN v0) interface to aircraft | MCP2515 (SPI, CS2) | SN65HVD230 (3.3V) |

Each MCP2515 requires an 8 MHz crystal + 2x 22pF load capacitors. 120 Ohm termination resistors are optional (directly connected or selectable via solder jumper depending on bus topology).

The board converts between the battery protocol (CAN 1) and DroneCAN (CAN 2) using [libcanard](https://github.com/dronecan/libcanard).

### GPIO Allocation

| Function | GPIOs | Notes |
|---|---|---|
| SPI Bus (shared) — SCK, MOSI, MISO | 3 | Directly routed to both MCP2515 |
| MCP2515 #1 — CS1, INT1 | 2 | CAN 1 (Battery) |
| MCP2515 #2 — CS2, INT2 | 2 | CAN 2 (Drone) |
| Temperature Sensors | 2 | 1-Wire (e.g. DS18B20) or analog NTC via ADC |
| Circuit Switching | 4 | MOSFET gate drive (e.g. precharge, kill switch, power enable) |
| **Total Used** | **13** | |
| **Remaining / Reserve** | **9** | Available for future expansion |

Pin count provides comfortable headroom for all current functions with 9 GPIOs in reserve.

## Functional Requirements

### Power Path
- **High-side MOSFET switching** — N-channel MOSFETs with charge pump gate drive
- **Precharge circuit** — soft-start via resistor + relay/MOSFET to limit inrush current
- **Emergency kill switch** — hardware-level kill input, independent of MCU

### Voltage Regulation
- **12V regulated output** — buck converter from battery voltage
- **5V regulated output** — buck converter from battery voltage
- Both outputs need adequate filtering, protection and isolation

### Monitoring and Communication
- **Dual CAN bus** — see [MCU and Communication](#mcu-and-communication) section above
- **Temperature sensors** — 2 pins reserved for monitoring battery connector and MOSFET hotspots (1-Wire or NTC/ADC)
- **WiFi / BLE** — integrated in ESP32-C3 for wireless diagnostics, configuration, and telemetry

### Protection
- **Short circuit protection via fuse**

## Repository Structure

```
18SBC_PCB/
├── README.md
├── kicad/              # KiCad 10 project files
│   ├── 18SBC.kicad_pro
│   ├── 18SBC.kicad_sch
│   └── 18SBC.kicad_pcb
├── docs/               # Documentation and datasheets
├── manufacturing/      # Gerber files and BOM
├── firmware/           # ESP32-C3 firmware (ESP-IDF / Arduino)
└── images/             # Board renders and photos
```

## Deliverables

1. **Schematic** — complete KiCad schematic with all subsystems
2. **PCB Layout** — rectangular board, routed for high current, antenna keepout zone respected
3. **BOM** — full bill of materials with supplier links
4. **Gerber files** — manufacturing-ready output for JLCPCB
5. **Documentation** — kicad files with component/footprint libraries in a zip folder.

## Getting Started

1. Install [KiCad 10](https://www.kicad.org/)
2. Clone this repository
3. Open `kicad/18SBC.kicad_pro`

## Reference Links

- [ESP32-C3-MINI-1 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-c3-mini-1_datasheet_en.pdf) — MCU module
- [MCP2515 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf) — CAN controller
- [SN65HVD230 Datasheet](https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf) — 3.3V CAN transceiver
- [DroneCAN](https://dronecan.github.io/) — CAN protocol for UAVs
- [libcanard](https://github.com/dronecan/libcanard) — lightweight DroneCAN implementation in C
- [Prolanv EN60A Connector](https://www.prolanv.com/2_2552837_5851637.html) — battery connector datasheet

## License

TBD
