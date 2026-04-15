# 18SBC PCB

18-Series Battery Connector Board — a high-current PCB that interfaces 18S smart batteries with a drone power system.

## Overview

The 18SBC converts an existing battery connector PCB design (originally for 14S / triangular form factor) to work with **18S smart batteries** and significantly higher current requirements. The board handles high-current power switching, precharge logic, emergency shutoff, isolated auxiliary supplies, CAN-based battery monitoring (DroneCAN), and on-board temperature sensing.

### Design Goals

- Compatible with **18S Tattu smart batteries**
- **Rectangular PCB** shape (previous design was triangular)
- **100A continuous** / **200A spike** current handling
- 12V and 5V regulated outputs for downstream electronics
- Full DroneCAN telemetry integration

---

## Specifications

| Parameter | Value |
|---|---|
| Max System Voltage | 90V (18S LiPo, 75.6V nominal) |
| Battery | Tattu 4.0 18S 30000mAh |
| Battery Connector | ACES 59615-0149D001 |
| Nominal Current | 100A continuous |
| Peak Current | 200A (short duration) |
| Motor Output Terminals | AMT0860001TA0000G (Amphenol Anytek, M6, 200A) |
| MCU | STM32G4 series (e.g. STM32G431, LQFP48) |
| EDA Tool | KiCad 10 (mandatory) |

---

## Current Requirements Rationale

Based on the motor reference data (Hobbywing XRotor X15):

| Throttle | Thrust | Current per Motor |
|---|---|---|
| 51% | 27,257 g | 44.7 A |
| 72% | 47,933 g | 101.6 A |
| 100% | 72,647 g | 197.1 A |

At 200 kg MTOW with 6 motors, hover throttle is ~50%. The 100A nominal rating covers normal flight operations; 200A spike handles full-throttle bursts.

---

## Functional Requirements

### Power MOSFETs
- Vds rated **≥200V**
- Low Rds(on), parallel MOSFETs sized for 100A+ continuous
- Robust gate drive circuit
- Hardware latch to keep MOSFETs ON reliably (independent of MCU state)

### Precharge Logic (MCU-controlled)
- On battery power-up → current flows through **precharge resistor** to charge downstream capacitors
- After **2 seconds** → MCU closes main MOSFETs
- Redundant latch mechanism: MOSFETs must stay closed even if MCU temporarily glitches
- Hardware watchdog on MCU

### Emergency Shutoff
- **No physical button** (battery has built-in switch)
- FC GPIO input (3.3V logic level)
- Signal must be held HIGH for **≥1 second** before MOSFETs open
- Debounce/timing handled in MCU firmware
- Prevents accidental shutoff from noise or transients

### CAN Bus — DroneCAN Node
- MCU connects to **Tattu battery CAN bus** via dedicated CAN transceiver
- Reads BMS data: SoC, voltage, current, temperature, cell voltages
- **Translates Tattu proprietary protocol → DroneCAN** (UAVCAN v0)
- Publishes `uavcan.equipment.power.BatteryInfo` + cell-level data
- Second CAN transceiver connects to **FC-side DroneCAN bus**
- Recommended CAN stack: **libcanard**

### On-Board Temperature Sensors
- **2x NTC thermistors** on PCB (suggested: MOSFET area + connector area)
- Read via MCU ADC
- Published as `uavcan.equipment.device.Temperature` on DroneCAN bus
- Sampling rate: ~1 Hz

### Isolated DC-DC Power Supplies
- **5V / 30W** (6A) — galvanically isolated
- **12V / 30W** (2.5A) — galvanically isolated
- Input voltage range: 50–90V

---

## Block Diagram

```
[Tattu 4.0 18S Battery]
    │
    ├─ Power ──→ [ACES 59615] ──→ [Precharge R] ──→ [Main MOSFETs] ──→ [AMT086] ──→ Motors
    │                                    ↑
    │                              [STM32G4 MCU]
    │                            ↑    ↑    ↓    ↓
    │                            │    │    │    └──→ DroneCAN ──────→ FC
    └─ CAN (Tattu BMS) ─────────┘    │    │         (BatteryInfo +
                                      │    │          Temperature)
                          FC GPIO ────┘    │
                          (Emergency OFF)  └──→ [Isolated DC-DC]
                                                 ├─ 5V / 30W ──→ FC
                          [NTC 1] ──→ ADC        └─ 12V / 30W ──→ FC
                          [NTC 2] ──→ ADC
```

---

## Interfaces to Flight Controller

| Interface | Direction | Details |
|---|---|---|
| 12V | 18SBC → FC | Isolated, 30W max |
| 5V | 18SBC → FC | Isolated, 30W max |
| CAN | 18SBC → FC | DroneCAN (translated from Tattu BMS + temperature data) |
| GPIO | FC → 18SBC | 3.3V input — Emergency Shutoff trigger (1s hold) |

---

## MCU Selection: STM32G4

| Requirement | STM32G4 | ATtiny |
|---|---|---|
| Native CAN peripherals | ✅ 2x FDCAN | ❌ None |
| Flash / RAM | 128KB / 32KB | 2–8KB / limited |
| DroneCAN stack (libcanard) | ✅ Fits easily | ❌ Too large |
| ADC for NTC sensors | ✅ Multiple channels | ✅ |
| Hardware Watchdog | ✅ IWDG + WWDG | ✅ WDT |
| Cost | ~2–3€ | ~1–2€ |

---

## Repository Structure

```
18SBC_PCB/
├── README.md
├── kicad/                # KiCad 10 project files
│   ├── 18SBC.kicad_pro
│   ├── 18SBC.kicad_sch
│   └── 18SBC.kicad_pcb
├── libs/                 # Project-local KiCad libraries
│   ├── 18SBC.kicad_sym
│   └── 18SBC.pretty/
├── 3d_models/            # STEP/WRL files
├── docs/                 # Documentation & datasheets
├── firmware/             # STM32G4 firmware source
├── manufacturing/        # Gerber files, drill files, BOM
└── images/               # Board renders & photos
```

---

## Deliverables

1. **KiCad 10 project** (`.kicad_pro`, `.kicad_sch`, `.kicad_pcb`)
2. **Symbol & footprint libraries** (project-local, all custom parts included)
3. **3D models** (`.step` / `.wrl`) if created
4. **BOM** (`.csv`) with manufacturer part numbers, quantities, suppliers
5. **Gerber + drill files** (manufacturing-ready)
6. **Schematic PDF** export
7. **STM32G4 firmware** (precharge, emergency shutoff, Tattu→DroneCAN, temperature)
8. **Documentation** — build instructions, firmware flashing guide, pinout docs

---

## Design & Tooling Requirements

- **KiCad 10** — mandatory, no exceptions
- All custom symbols/footprints as **project-local libraries**
- Do NOT rely on global/system KiCad libraries for custom parts

---

## References

- [DroneCAN](https://dronecan.github.io/)
- [libcanard](https://github.com/dronecan/libcanard)
- Tattu 4.0 CAN protocol documentation — provided separately

---

## Getting Started

1. Install [KiCad 10](https://www.kicad.org/)
2. Clone this repository
3. Open `kicad/18SBC.kicad_pro`

## License

TBD
