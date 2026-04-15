# 18SBC PCB

18-Series Battery Connector Board – a PCB designed for high-voltage battery pack integration.

## Overview

The 18SBC is a battery connector board that interfaces an 18S lithium-ion battery pack with the rest of the electrical system.

## Specifications

| Parameter | Value |
|---|---|
| Configuration | 18S (18 series cell groups) |
| Max Voltage | 75.6V (4.2V × 18) |
| Nominal Voltage | 64.8V (3.6V × 18) |
| EDA Tool | KiCad 10 |
| Board Layers | TBD |

## Repository Structure

```
18SBC_PCB/
├── README.md
├── kicad/           # KiCad project files
│   ├── 18SBC.kicad_pro
│   ├── 18SBC.kicad_sch
│   └── 18SBC.kicad_pcb
├── docs/            # Documentation & datasheets
├── manufacturing/   # Gerber files & BOM
└── images/          # Board renders & photos
```

## Getting Started

1. Install [KiCad 10](https://www.kicad.org/)
2. Clone this repository
3. Open `kicad/18SBC.kicad_pro`

## License

TBD
