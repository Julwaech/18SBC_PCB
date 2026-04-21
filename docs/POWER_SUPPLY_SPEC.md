# Power Supply Specification — 18SBC

## Overview

The 18SBC includes two independent, isolated step-down (buck) voltage regulators to provide regulated power rails from the high-voltage battery bus. Both regulators are designed for high-current drone avionics loads with emphasis on efficiency and reliability.

## Input Requirements

| Parameter | Value |
|---|---|
| Operating Input Voltage | 9V – 85V DC |
| Component Voltage Rating | 100V DC minimum |
| TVS Protection | 85V TVS diode on input |
| Input Capacitance | Sufficient bulk + ceramic capacitance for ripple suppression |

> **Note:** The 14S Li-Ion pack operates between 42V (empty) and 58.8V (full), well within the 9–85V input range.

## Regulator 1 — Avionics Rail

| Parameter | Value |
|---|---|
| Output Voltage Options | 5.3V / 6V / 8V (selectable) |
| Output Voltage Tolerance | ≤ 1.5% |
| Topology | Synchronous buck |
| Typical Efficiency | 90% – 96% |
| Continuous Output Current | 15A typical |
| Peak Output Current | 22A |
| Output Ripple | ≤ 25mV @ 5.3V/5A, ≤ 35mV @ 5.3V/15A |
| Standby Current | 10–30 mA |
| Thermal Shutdown Threshold | 150°C |
| Short-Circuit Protection | Tolerant for 1 second, overcurrent with self-recovery |

> **Note:** Input voltage ≥ 15V required for sustained 15A output.

## Regulator 2 — Peripheral / High-Voltage Rail

| Parameter | Value |
|---|---|
| Output Voltage Options | 5.3V / 8V / 12V (selectable) |
| Output Voltage Tolerance | ≤ 1.5% |
| Topology | Synchronous buck |
| Typical Efficiency | 90% – 96% |
| Continuous Output Current | 15A typical |
| Peak Output Current | 22A |
| Output Ripple | ≤ 58mV @ 12V/5A |
| Standby Current | 10–30 mA |
| Thermal Shutdown Threshold | 150°C |
| Short-Circuit Protection | Tolerant for 1 second, overcurrent with self-recovery |

> **Note:** Input voltage ≥ 15V required for sustained 15A output.

## Isolation

Both regulators shall be galvanically isolated from the high-voltage battery bus. This ensures:
- Protection of downstream avionics from battery-side faults
- Clean ground separation between power domains
- Compliance with aerospace-grade power architecture best practices

## Voltage Sensing (ADC)

| Parameter | Value |
|---|---|
| Voltage Divider Ratio | 1K : 25K (built-in) |
| ADC Reference | 3.3V based |
| Purpose | Battery voltage monitoring via FC analog input |

## Physical Targets

| Parameter | Target |
|---|---|
| Board Area | ≤ 61 × 40 mm |
| Height (incl. heatsink) | ≤ 15 mm |
| Weight | ≤ 63 g |

## Protection Features Summary

- [x] 85V TVS input protection
- [x] Overcurrent protection with auto-recovery
- [x] Short-circuit tolerant (1s)
- [x] Thermal protection at 150°C
- [ ] Reverse input voltage protection (not required — battery connector is keyed)

## Integration Notes

- The power supply module connects to the main battery bus via the 18SBC board
- Kill switch (see [KILL_SWITCH_SPEC.md](KILL_SWITCH_SPEC.md)) operates upstream of the power supply — when MOSFETs are off, regulators receive no input power
- Both rails shall have enable pins accessible for sequenced power-up if needed
