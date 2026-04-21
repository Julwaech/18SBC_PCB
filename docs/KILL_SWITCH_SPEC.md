# 18SBC Emergency Kill Switch Specification

## Overview
The 18SBC receives a **12V emergency kill signal** from the Flight Controller (FC) via a dedicated cable. This signal must be hardware-debounced and capable of overriding the ESP32 MOSFET control to cut battery power.

## Signal Input
- **Voltage:** 12V (amplified from FC GPIO on a separate board)
- **Active State:** HIGH = Kill requested
- **Connector:** TBD (dedicated pin on signal connector)

## Hardware Debounce (1s minimum)
To prevent false cutoffs during flight (which would cause a crash), the kill signal must be stable for **>=1 second** before triggering.

### Implementation: RC Filter + Schmitt-Trigger
- **R = 150k ohm, C = 10uF** -> tau = 1.5s
- **Schmitt-Trigger** (e.g. 74LVC1G17, 3.3V logic):
  - Voltage divider before Schmitt-Trigger to scale 12V -> 3.3V range
  - Triggers at ~70% of input -> effective delay approx 1.05s
- Short spikes and EMI noise are fully absorbed by the RC filter

**Output:** KILL_CONFIRMED -- HIGH only after >=1s stable kill signal

## MOSFET Control Logic (Hardware AND-Gate)
The MOSFETs are controlled by a **hardware AND-gate** (e.g. 74LVC1G08) that combines the ESP32 enable signal with the inverted kill signal:

```
MOSFET_ENABLE = ESP32_ENABLE AND (NOT KILL_CONFIRMED)
```

### Truth Table
| ESP32_ENABLE | KILL_CONFIRMED | MOSFETs | State            |
|--------------|----------------|---------|------------------|
| LOW          | LOW            | **OFF** | Boot / Error     |
| HIGH         | LOW            | **ON**  | Normal Operation |
| HIGH         | HIGH           | **OFF** | Emergency Kill   |
| LOW          | HIGH           | **OFF** | Kill + No Enable |

### Key Properties
- **Default state is OFF** -- safe during boot, reset, or ESP32 failure
- **Hardware override** -- ESP32 cannot block or bypass the kill signal
- **ESP32 enable is active-high** -- must be explicitly driven HIGH for MOSFETs to turn on
- The AND-gate operates purely in hardware, independent of any firmware

## Component Summary
| Component       | Part (Example)   | Function                        |
|-----------------|------------------|---------------------------------|
| R (debounce)    | 150k ohm         | RC time constant with C         |
| C (debounce)    | 10uF             | RC time constant with R         |
| Voltage divider | Resistors TBD    | Scale 12V kill signal to 3.3V   |
| Schmitt-Trigger | 74LVC1G17        | Clean digital output from RC    |
| Inverter        | 74LVC1G04        | Invert KILL_CONFIRMED           |
| AND-Gate        | 74LVC1G08        | Combine ESP32_ENABLE + NOT KILL |

## Design Priorities
1. **No false cutoff during flight** -- 1s debounce + noise-immune 12V bus
2. **Guaranteed kill when intended** -- pure hardware path, no firmware dependency
3. **Safe default state** -- MOSFETs OFF unless explicitly enabled
