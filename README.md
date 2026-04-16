# 18SBC PCB

18-Series Battery Connector Board — a high-current PCB designed for 18S lithium battery pack integration in heavy-lift UAV systems.

## Overview

The 18SBC is a battery connector board that interfaces 18S smart battery packs (e.g. Tattu 4.0) with the aircraft electrical system.

### Design Goals

- Support 18S smart batteries (Tattu 4.0 compatible)
- Handle 100A continuous / 200A spike current
- Provide regulated and isolated 12V and 5V outputs to the main system PCB (5V and 12V can share GND)
- Active power switching with precharge and emergency shutoff
- CAN bus communication (DroneCAN)

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
- **MCU: STM32G4** — ARM Cortex-M4, hardware CAN-FD, 12-bit ADC, sufficient GPIOs
- **DroneCAN (UAVCAN v0)** — using [libcanard](https://github.com/dronecan/libcanard) for lightweight CAN implementation. It will convert the battery protocol to DroneCAN, so the PCB needs two seperate CAN buses.
- **NTC temperature sensors** — monitor battery connector, MOSFETs, and board hotspots (2 sensors should be enough)

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
├── firmware/           # STM32G4 firmware (if applicable)
└── images/             # Board renders and photos
```

## Deliverables

1. **Schematic** — complete KiCad schematic with all subsystems
2. **PCB Layout** — rectangular board, routed for high current
3. **BOM** — full bill of materials with supplier links
4. **Gerber files** — manufacturing-ready output for JLCPCB
5. **Documentation** — kicad files with component/footprint libraries in a zip folder.

## Getting Started

1. Install [KiCad 10](https://www.kicad.org/)
2. Clone this repository
3. Open `kicad/18SBC.kicad_pro`

## Reference Links

- [DroneCAN](https://dronecan.github.io/) — CAN protocol for UAVs
- [libcanard](https://github.com/dronecan/libcanard) — lightweight DroneCAN implementation in C
- [Prolanv EN60A Connector](https://www.prolanv.com/2_2552837_5851637.html) — battery connector datasheet

## License

TBD
