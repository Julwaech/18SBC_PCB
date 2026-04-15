# 18SBC PCB

18-Series Battery Connector Board — a high-current PCB designed for 18S lithium battery pack integration in heavy-lift UAV systems.

## Overview

The 18SBC is a battery connector board that interfaces 18S smart battery packs (e.g. Tattu 4.0) with the aircraft electrical system. It converts the existing triangular BC-PCB design into a rectangular form factor while supporting significantly higher current requirements.

### Design Goals

- Convert existing triangular BC-PCB to rectangular shape
- Support 18S smart batteries (Tattu 4.0 compatible)
- Handle 100A continuous / 200A spike current
- Provide regulated 12V and 5V outputs to the main system PCB
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
| Board Layers | TBD (likely 4-layer, 2oz+ copper) |

### Current Rating Justification

Based on the motor data (Hobbywing XRotor X15):

| Throttle | Thrust | Current per Motor |
|---|---|---|
| 51% | 27,257 g | 44.7 A |
| 72% | 47,933 g | 101.6 A |
| 100% | 72,647 g | 197.1 A |

At 200 kg MTOW with 6 motors, hover throttle is ~50%. The 100A nominal rating covers normal operations; the 200A spike rating handles full-throttle bursts.

## Battery Connector

**Selected: [Prolanv EN60A](https://www.prolanv.com/2_2552837_5851637.html)**

| Parameter | Value |
|---|---|
| Current per Power Pair | 60A |
| Power Pairs | 5 positive + 5 negative |
| Total Current Capacity | 300A |
| Signal Pins | 12 |
| Contact Resistance | 0.6 mOhm |
| Mating Cycles | 10,000 |
| Temperature Range | -40C to +125C |
| Mounting | Right-Angle DIP (PCB mount) |

The EN60A provides 300A total capacity across 5 power pairs, giving comfortable headroom above the 200A spike requirement. The 12 signal pins are sufficient for cell balancing, CAN bus, and temperature sensor lines.

At 200A spike across 4 power pairs (50A each): P = I^2 x R = 50^2 x 0.0006 = 1.5W per pair = 6W total — well within thermal limits.

## Functional Requirements

### Power Path
- **High-side MOSFET switching** — N-channel MOSFETs with charge pump gate drive
- **Precharge circuit** — soft-start via resistor + relay/MOSFET to limit inrush current
- **Emergency kill switch** — hardware-level kill input, independent of MCU
- **Reverse polarity protection**
- **Current sensing** — high-side shunt resistor with dedicated current sense amplifier

### Voltage Regulation
- **12V regulated output** — buck converter from battery voltage, routed to main PCB
- **5V regulated output** — buck converter, routed to main PCB
- Both outputs need adequate filtering and protection

### Monitoring and Communication
- **MCU: STM32G4** — ARM Cortex-M4, hardware CAN-FD, 12-bit ADC, sufficient GPIOs
- **DroneCAN (UAVCAN v0)** — using [libcanard](https://github.com/dronecan/libcanard) for lightweight CAN implementation
- **NTC temperature sensors** — monitor battery connector, MOSFETs, and board hotspots
- **Cell voltage monitoring** — via battery smart interface or external balancer IC
- **Telemetry broadcast** — voltage, current, temperature, state-of-charge over CAN bus

### Protection
- **Overvoltage / Undervoltage lockout**
- **Overcurrent protection** (hardware + software)
- **Thermal cutoff**
- **Short circuit protection**

## Block Diagram

```
Battery (18S)
    |
    v
+------------------+
|  EN60A Connector | <-- 300A capacity, 12 signal pins
|  (Prolanv)       |
+--------+---------+
         |
    +----+----+
    | Precharge|
    | Circuit  |
    +----+----+
         |
    +----+----+
    | MOSFET  | <-- Emergency kill input
    | Switch  |
    +----+----+
         |
    +----+----+         +----------+
    | Current |-------->| STM32G4  |
    | Sense   |         | MCU      |
    +----+----+         |          |
         |              | - ADC    |
         +--------------| - CAN-FD |
         |              | - GPIOs  |
         |              +-----+----+
         |                    |
    +----+--------------------+---+
    |         Power Bus           |
    +----+----------+----------+--+
         |          |          |
    +----+---+ +----+---+     |
    | 12V    | | 5V     |     |
    | Buck   | | Buck   |     v
    +----+---+ +----+---+   Main
         |          |       Power
         v          v       Output
    To Main PCB  To Main PCB
```

## MCU Selection

| Feature | STM32G4 | ATtiny |
|---|---|---|
| Core | ARM Cortex-M4 | AVR 8-bit |
| CAN | Hardware CAN-FD | None (needs external MCP2515) |
| ADC | 12-bit, multiple channels | 10-bit, limited |
| Processing | 170 MHz | 20 MHz |
| DroneCAN | Native via libcanard | Possible but constrained |
| Cost | ~$3-5 | ~$1-2 |
| **Verdict** | **Selected** | Too limited |

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
4. **Gerber files** — manufacturing-ready output
5. **Documentation** — design notes and connector pinout

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