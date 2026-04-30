# Changelog

## 2025-04-29 — Design Revision

### Changed
- **MCU:** ESP32-C3-MINI-1 replaced with **ESP32-S3-WROOM-1-N16R8** (LCSC C2913202)
  - Reason: ESP32-C3 has insufficient GPIOs for all required functions
  - ESP32-S3 provides 45 GPIOs, dual-core Xtensa LX7 @ 240 MHz, 16MB flash, 512KB SRAM
  - Native USB support (no external USB-UART bridge needed)
- **Battery connector:** Prolanv EN60A removed due to mechanical fit issues
  - **Power:** Now uses **AMT0650009DB0000G** screw terminals (same as propulsion output)
  - **Signal/CAN:** New **DB128V-5.08-4P-GN-S** 4-pin screw terminal block (LCSC C2915641)
  - PCB layout (top of board): Positive (left) | Signal (center) | Negative (right)

### Added
- **USB-C connector** for debugging, firmware flashing, and serial monitoring
  - Connects to ESP32-S3 native USB (GPIO19 D-, GPIO20 D+)
- **Hardware interlock documentation:** Pin1 and Pin2 on the battery signal connector must be bridged on the PCB to activate the battery
