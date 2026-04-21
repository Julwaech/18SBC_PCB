# Power Supply Specification

## Overview

The 18SBC includes two independent step-down voltage regulators to provide power to external systems (flight controller, peripherals, sensors). These regulators are part of the **External GND Domain** and are galvanically separated from the internal board logic.

## Input

| Parameter | Value |
|---|---|
| Source | 18S HV Battery Bus |
| Voltage Range | 54.0V - 75.6V (3.0V - 4.2V per cell) |
| Component Rating | 100V DC |
| TVS Protection | 85V |

## Regulators

### Regulator 1 - 5V Rail

| Parameter | Value |
|---|---|
| Output Voltage | 5V (+-1.5%) |
| Continuous Current | 3-5A |
| Topology | Synchronous Buck |
| Efficiency | 90-96% typical |
| Output Ripple | <=35mV at full load |
| Protections | Overcurrent (self-recovery), thermal cutoff (150C), short-circuit tolerant |

### Regulator 2 - 12V Rail

| Parameter | Value |
|---|---|
| Output Voltage | 12V (+-1.5%) |
| Continuous Current | 3-5A |
| Topology | Synchronous Buck |
| Efficiency | 90-96% typical |
| Output Ripple | <=60mV at full load |
| Protections | Overcurrent (self-recovery), thermal cutoff (150C), short-circuit tolerant |

## Ground Domain Architecture

The 18SBC has two electrically separated ground domains:

### Internal GND (GNDI)
- ESP32 MCU
- BMS logic
- MOSFET gate drivers
- Cell voltage sensing
- All internal signal processing

### External GND (GNDE)
- 5V and 12V regulator outputs
- Kill signal input (see [Kill Switch Spec](KILL_SWITCH_SPEC.md))
- All external connectors / interfaces

### Isolation Boundary

Signals crossing from External to Internal domain (or vice versa) **must be isolated**. Acceptable methods:
- Optocouplers
- Isolated gate drivers
- Isolated DC/DC converters

> **Rule:** DCDC regulator outputs may only be used internally if the signal path includes galvanic isolation before reaching the Internal GND domain.

## Battery Voltage Sensing

| Parameter | Value |
|---|---|
| Voltage Divider | 1k : 25k (input : ground) |
| Output | 0-2.91V for 0-75.6V input |
| Interface | 3.3V ADC compatible |
| Domain | Internal GND |

## Internal Power Supply

> **TBD** - The power source for the internal domain (ESP32, BMS logic, etc.) is yet to be defined. Options under consideration include a dedicated isolated DC/DC converter from the HV bus or an isolated tap from one of the external regulators.

## Integration Notes

- The **Kill Switch** ([spec](KILL_SWITCH_SPEC.md)) operates on the External GND domain (12V rail). The kill signal crosses into the Internal domain via an optocoupler to reach the hardware AND-gate that controls the MOSFET enable line.
- External connectors must clearly indicate which ground domain they reference (GNDE).
