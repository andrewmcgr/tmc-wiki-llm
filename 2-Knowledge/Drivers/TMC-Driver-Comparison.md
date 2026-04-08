# TMC Driver Comparison

**Type:** knowledge
**Summary:** Side-by-side comparison of Trinamic TMC2209, TMC2240, TMC5160A, and TMC4671-LA — electrical specs, feature matrices, interface options, and selection guidance for stepper motor and servo applications.
**Tags:** #drivers #hardware #reference #comparison
**Status:** draft
**Updated:** 2026-04-08
**Related:** [[TMC2209]], [[TMC2240]], [[TMC5160A]], [[TMC4671-LA]]

---

## Overview

This page compares four Trinamic/Analog Devices motor control ICs spanning the range from low-cost integrated stepper drivers to high-performance hardware FOC servo controllers. The first three (TMC2209, TMC2240, TMC5160A) are stepper motor drivers with integrated chopper control; the TMC4671-LA is a fundamentally different device — a hardware FOC servo controller that requires an external power stage and feedback sensor.

---

## At a Glance

| | TMC2209 | TMC2240 | TMC5160A | TMC4671-LA |
|---|---|---|---|---|
| **Function** | Integrated stepper driver | Integrated stepper driver | Stepper controller + gate driver | Hardware FOC servo controller |
| **Power stage** | Internal MOSFETs | Internal MOSFETs | External N-ch MOSFETs | External (e.g. TMC6100) |
| **Motor types** | 2-phase bipolar stepper | 2-phase bipolar stepper | 2-phase bipolar stepper | BLDC/PMSM, stepper, DC, voice coil |
| **VS range** | 4.75–29 V | 4.5–36 V | 8–60 V | N/A (power stage sets this) |
| **Max RMS current** | 2.0 A (duty-cycled) | 2.1 A | 20 A+ (MOSFET-limited) | Power-stage-limited |
| **Peak current** | 2.8 A | 3.0 A | MOSFET-limited | Power-stage-limited |
| **RDS(ON) (typ)** | 170 mΩ (HS+LS each) | 230 mΩ (HS+LS) | N/A (external FETs) | N/A |
| **Current sensing** | External sense resistors (or internal RDS(ON)) | Non-dissipative ICS (RDS(ON)) | External sense resistors | External shunt + amplifier |
| **Package** | QFN28, 5×5 mm | TQFN32 5×5 mm / TSSOP38 | eTQFP48 7×7 mm / QFN56 8×8 mm | QFN76, 10.5×6.5 mm |

---

## Interface Comparison

| | TMC2209 | TMC2240 | TMC5160A | TMC4671-LA |
|---|---|---|---|---|
| **SPI** | No | Yes (MODE 3, 10 MHz) | Yes (MODE 3, 4–8 MHz) | Yes (MODE 3, 8 MHz) |
| **UART** | Single-wire (up to 4 nodes) | Single-wire (up to 255 nodes) | Single-wire (up to 255 nodes) | 3-wire (RxD/TxD/GND) |
| **Step/Dir** | Yes | Yes | Yes | Yes |
| **Max UART baud** | 500 kBaud | fCLK/16 | fCLK/16 | 3 Mbps |
| **Daisy chain** | No | No | Yes (SPI) | No |
| **Internal step generator** | Yes (VACTUAL) | No | Yes (SixPoint ramp) | No (uses Step/Dir input) |
| **Ramp generator** | No | No | Yes (6-section hardware) | No |

---

## Motion Feature Comparison

| Feature | TMC2209 | TMC2240 | TMC5160A | TMC4671-LA |
|---|---|---|---|---|
| **StealthChop2** | Yes | Yes | Yes | N/A (FOC) |
| **SpreadCycle** | Yes | Yes | Yes | N/A (FOC) |
| **MicroPlyer (256× interp.)** | Yes | Yes | Yes | N/A |
| **Native microstepping** | Up to 256 | Up to 256 | Up to 256 | N/A (continuous FOC) |
| **StallGuard2 (SpreadCycle)** | No | Yes | Yes | N/A |
| **StallGuard4 (StealthChop)** | Yes | Yes | No | N/A |
| **CoolStep** | Yes (SG4-based) | Yes (SG2/SG4-based) | Yes (SG2-based) | N/A |
| **dcStep** | No | No | Yes | N/A |
| **Resonance dampening (TPFD)** | No | No | Yes | N/A |
| **Encoder input** | No | Yes (ABN on-chip) | Yes (ABN on-chip) | Yes (ABN, Hall, analog sin/cos) |
| **Sensorless operation** | Yes (StallGuard4) | Yes (StallGuard2/4) | Yes (StallGuard2) | No (sensor required) |
| **FOC control** | No | No | No | Yes (hardware cascade) |
| **Closed-loop position** | No | No | Encoder deviation check only | Yes (full PID) |
| **Torque control** | Current scaling only | Current scaling only | Current scaling only | Yes (direct torque PI) |

---

## Protection Feature Comparison

| Feature | TMC2209 | TMC2240 | TMC5160A | TMC4671-LA |
|---|---|---|---|---|
| **Overtemperature shutdown** | 143 °C (configurable) | 165 °C | 136/143/150 °C (selectable) | None (no power stage) |
| **Overtemperature warning** | 120 °C | Programmable (OTW_OV_VTH) | 120 °C | None |
| **Short-to-GND** | Yes (retry 3×) | Yes (per bridge) | Yes (retry 3×) | None |
| **Short-to-VS** | Yes | Yes (per bridge) | Yes | None |
| **Open load detect** | Yes (informational) | No | Yes (informational) | None |
| **Overcurrent protection** | Via sense resistor threshold | Hardware OCP (5 A IMAX) | Via sense resistor threshold | None (external responsibility) |
| **UVLO** | No | Yes (VS < 3.9 V) | Yes (VSA rising ~4 V) | No |
| **Overvoltage protection** | No | Yes (programmable OV pin) | No | No |
| **Charge pump UV** | Yes | Yes | Yes | N/A |
| **Temperature ADC** | No | Yes (on-chip) | No | No |

---

## Electrical Specifications Detail

### Supply and Power

| Parameter | TMC2209 | TMC2240 | TMC5160A | TMC4671-LA |
|---|---|---|---|---|
| VS operating | 4.75–29 V | 4.5–36 V | 8–60 V | N/A |
| VS absolute max | 33 V | 41 V | 64 V (short-time) | N/A |
| VCC_IO | 3.0–5.25 V | 2.2–5.5 V | 3.0–5.25 V | 3.15–3.45 V (3.3 V only) |
| Internal regulator | 5 V out | 5 V out | 5 V out + 12 V gate driver | 1.8 V core |
| Standby current | 160 µA typ | 4 µA (sleep mode) | 18 mA (idle, no motor) | N/A |
| Clock | 12 MHz internal (or ext. 4–16 MHz) | 12.5 MHz internal (or ext. 8–20 MHz) | 12 MHz internal (or ext. 4–18 MHz) | 25 MHz external (mandatory) |

### Thermal

| Parameter | TMC2209 | TMC2240 (TQFN32) | TMC2240 (TSSOP38) | TMC5160A (eTQFP48) |
|---|---|---|---|---|
| θ_JA | 30 K/W (2s2p) | 29 °C/W | 25 °C/W | 21 K/W (2s2p) |
| θ_JC | 6 K/W | 1.7 °C/W | 1.0 °C/W | 3 K/W |
| T_J operating | −40 to +125 °C | −40 to +125 °C | −40 to +125 °C | −40 to +125 °C |

---

## Package Comparison

| IC | Package(s) | Dimensions | Pin count | Pitch | Thermal pad |
|---|---|---|---|---|---|
| TMC2209 | QFN28 | 5×5 mm | 28 | 0.5 mm | Exposed GND slug |
| TMC2240 | TQFN32 | 5×5 mm | 32 | 0.5 mm | Exposed pad |
| TMC2240 | TSSOP38 | 9.7×4.4 mm | 38 | 0.5 mm | Exposed pad |
| TMC5160A | eTQFP48 | 7×7 mm body | 48 | 0.5 mm | Exposed die pad |
| TMC5160A | QFN56 WF | 8×8 mm | 56 | 0.4 mm | Wettable flanks |
| TMC4671-LA | QFN76 | 10.5×6.5 mm | 76 | 0.4 mm | Exposed pad |

---

## Current Sensing Approaches

| IC | Method | Sense element | Accuracy | Notes |
|---|---|---|---|---|
| TMC2209 | External shunt | R_sense (typ. 100–150 mΩ) | Good (resistor-dependent) | Also supports internal RDS(ON) (≤1.4 A, lower accuracy) |
| TMC2240 | Integrated Current Sense (ICS) | MOSFET RDS(ON) | ±5% (GLOBALSCALER=0) | No external resistors needed; RREF sets full-scale |
| TMC5160A | External shunt (Kelvin) | R_sense (typ. 22–220 mΩ) | ±5% full-scale | Four-wire Kelvin connection for accuracy |
| TMC4671-LA | External shunt + amplifier | R_shunt + AD8418A / LT1999 | External-dependent | ADC inputs are low-voltage differential; amplifier mandatory |

---

## Selection Guide

### Choose TMC2209 when:
- Coil current ≤ 1.4 A RMS continuous (or ≤ 2.0 A duty-cycled)
- Supply voltage 5–29 V
- UART-only communication is acceptable (no SPI needed)
- StealthChop silence and StallGuard4 sensorless homing are key requirements
- Lowest BOM cost and smallest footprint are priorities
- Target: 3D printers, light CNC, camera gimbals, office automation

### Choose TMC2240 when:
- Coil current ≤ 2.1 A RMS
- Supply voltage up to 36 V
- Eliminating sense resistors matters (ICS — no power wasted in sensing)
- Both SPI and UART are needed
- On-chip encoder interface and temperature ADC reduce BOM complexity
- Both StallGuard2 (SpreadCycle) and StallGuard4 (StealthChop) are desired
- Target: factory automation, lab equipment, medical devices, upgraded 3D printers

### Choose TMC5160A when:
- Coil current > 2 A, up to 20 A+ with appropriate external MOSFETs
- Supply voltage up to 60 V
- Hardware ramp generator (SixPoint) eliminates MCU real-time burden
- dcStep load-adaptive commutation is required (motor never loses steps)
- Resonance dampening (TPFD) is needed for mid-speed smoothness
- Encoder deviation monitoring is needed for safety-critical positioning
- Target: industrial automation, high-torque CNC, robotics, medical equipment

### Choose TMC4671-LA when:
- Closed-loop servo performance is required (true torque/velocity/position FOC)
- Motor type is BLDC/PMSM, or a stepper requiring FOC for maximum torque bandwidth
- Deterministic, hardware-only control loop execution is needed (no MCU jitter)
- Multiple motor types must be supported by one controller IC
- A feedback sensor (encoder, Hall, analog sin/cos) is available
- **Not suitable** when: sensorless operation is required, or when a simple integrated driver is preferred
- Target: BLDC spindles, robotics joints, precision linear stages, servo axes

---

## Architecture Decision Tree

```
Start
  │
  ├── Is the motor a BLDC/PMSM, or do you need true closed-loop FOC?
  │     └── YES → TMC4671-LA + external power stage (e.g. TMC6100)
  │
  ├── Do you need > 2.1 A RMS or > 36 V supply?
  │     └── YES → TMC5160A + external MOSFETs
  │
  ├── Do you need SPI, or encoder input, or lossless current sensing?
  │     └── YES → TMC2240
  │
  └── Is UART-only acceptable, current ≤ 2 A, voltage ≤ 29 V?
        └── YES → TMC2209
```

---

## BOM Complexity Comparison

| IC | External power components | Sense components | Feedback components | Total external passives (approx.) |
|---|---|---|---|---|
| TMC2209 | None (integrated FETs) | 2× sense resistors (optional: internal RDS(ON)) | None required | ~10 (caps, VREF resistor) |
| TMC2240 | None (integrated FETs) | RREF resistor only (ICS) | Optional encoder connector | ~8 (caps, RREF) |
| TMC5160A | 8× external MOSFETs + bootstrap caps | 2× Kelvin sense resistors | Optional encoder connector | ~25 (MOSFETs, caps, resistors) |
| TMC4671-LA | Full power stage (gate driver + MOSFETs) | Shunt resistors + amplifier ICs | Encoder or Hall sensor (mandatory) | ~40+ (power stage, amplifiers, sensors, caps) |

---

## Register Compatibility Notes

- **TMC2209 ↔ TMC2240:** Similar CHOPCONF, PWMCONF, DRV_STATUS layouts. TMC2240 adds THIGH, SG4 registers, and encoder registers. GCONF layout differs.
- **TMC2209 ↔ TMC5160A:** Motion controller registers (0x20–0x2D ramp, SW_MODE, RAMP_STAT) are unique to TMC5160A. Chopper registers are similar.
- **TMC5160A ↔ TMC5130:** Pin and software compatible; TMC5160A is the direct upgrade for higher current.
- **TMC4671-LA:** Completely different register map and architecture. Not compatible with the TMC2xxx/5xxx stepper driver family.

---

## Key References

- TMC2209 Datasheet Rev 1.09 (Trinamic/Analog Devices)
- TMC2240 Datasheet (Trinamic/Analog Devices)
- TMC5160A Datasheet Rev 1.17 (Trinamic/Analog Devices)
- TMC4671-LA Datasheet Rev 2.08 (Trinamic/Analog Devices, 2022)
- Individual wiki pages: [[TMC2209]], [[TMC2240]], [[TMC5160A]], [[TMC4671-LA]]
