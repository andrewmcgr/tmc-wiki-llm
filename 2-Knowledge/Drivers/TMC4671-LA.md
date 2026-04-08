# TMC4671-LA

**Type:** knowledge
**Summary:** Hardware field-oriented control (FOC) servo controller IC for BLDC/PMSM, stepper, DC, and voice-coil motors; requires external power stage and gate driver.
**Tags:** #drivers #hardware #reference #tmc4671 #servo #foc
**Status:** draft
**Updated:** 2026-04-08
**Source:** TMC4671-LA_datasheet_rev2.08.pdf (Trinamic/Analog Devices, Rev 2.08, 2022-JUL-26)
**Related:** [[TMC2209]], [[TMC2240]], [[TMC5160A]]

---

## Overview

The TMC4671-LA is a **hardware FOC servo controller**, not a motor driver. It computes Clarke/Park transforms, runs PI current/velocity/position control loops, and outputs gate-drive PWM signals — but it has **no integrated power stage**. An external gate driver (e.g. TMC6100) and power MOSFETs are required for all applications.

The IC implements the complete closed-loop cascade in silicon at PWM rate, enabling deterministic torque response without MCU intervention. A 25 MHz external oscillator is mandatory; lower frequencies scale all timings proportionally, higher frequencies are not supported.

**Package:** QFN76, 0.4 mm pitch, 10.5 mm × 6.5 mm body, bottom exposed pad for thermal dissipation.

---

## Key Specifications

| Parameter | Value |
|---|---|
| Package | QFN76, 10.5 mm × 6.5 mm |
| Digital I/O supply (VCCIO) | 3.15 V – 3.45 V (3.3 V nominal) |
| Analog supply (V5) | 5 V recommended (3.3 V possible) |
| Core supply (VCC_CORE) | 1.8 V (internally generated from VCCIO) |
| Clock input (CLK) | 25 MHz external oscillator |
| Junction temperature | −40 °C to +125 °C |
| ADC input voltage (VAI) | 0 V – 5 V absolute max |
| Max VCCIO current (fCLK = 25 MHz) | 70 mA |
| Max V5 current (fCLK = 25 MHz) | 25 mA |
| Digital I/O output drive | 4 mA standard |
| ADC input impedance | 85–115 kΩ (typ 100 kΩ) |
| ESD protection (HBM) | 2 kV |
| PWM outputs | 8 (4 half-bridges: UX1 H/L, VX2 H/L, WY1 H/L, Y2 H/L) |
| SPI clock | Up to 8 MHz (with 500 ns pause on read) |
| Register address space | 128 addresses (0x00–0x7F) |

---

## Hardware FOC Architecture

The TMC4671 implements the complete FOC signal chain in dedicated hardware logic, updated each PWM period:

```
Position Target → [P] → Velocity Target → [PI] → Torque/Flux Targets
                                                        ↓
                  Mechanical → Clarke → Park → [PI×2] → iPark → iClarke → PWM
                  Feedback
```

**Three FOC modes** are selected via `MOTOR_TYPE_N_POLE_PAIRS` (register 0x1B):

| Mode | Value | Motor Type | Clarke/Park |
|---|---|---|---|
| FOC3 | 3 | 3-phase BLDC/PMSM | Full Clarke + Park |
| FOC2 | 2 | 2-phase stepper | iClarke bypassed (XY frame) |
| FOC1 | 1 | DC motor / voice coil | All transforms bypassed |
| — | 0 | None / disabled | — |

**Pole pairs:** For stepper motors, NPP = FullSteps / 4 (e.g. 200-step motor → NPP = 50). For DC motors, set NPP = 1.

**PI controller structures** (selected via `MODE_RAMP_MODE_MOTION` bit 31):
- **Classic (parallel):** P as Q8.8, I as Q0.15. Flux/torque loops run at 32 µs; velocity at 256 µs.
- **Advanced (sequential):** P selectable Q8.8 or Q4.12; runs at PWM frequency. Supports `MODE_PID_SMPL` downsampling.

Four biquad IIR filters (Q3.29 coefficients) are available for position target, velocity target, torque target, and flux target smoothing.

The IC does **not** support sensorless FOC — at least one feedback sensor is required for commutation.

---

## Motor Types Supported

**BLDC / PMSM (FOC3):** Full 3-phase Clarke and Park transforms. Hall, ABN encoder, or analog sin/cos encoder required for phi_e. SVPWM available for isolated star point motors (+12% voltage utilization).

**Stepper (FOC2):** 2-phase motor in XY frame. Clarke transform is replaced by direct XY current mapping. iClarke inverse is bypassed. Enables closed-loop microstepping and stall detection at full torque bandwidth.

**DC motor / voice coil (FOC1):** Single-phase. All transforms bypassed. Only one current measurement needed. Torque = current control only.

---

## Encoder / Feedback Support

| Sensor Type | Register Range | Notes |
|---|---|---|
| Open loop (no sensor) | 0x1A–0x1F | PHI_E_SELECTION = 2; for initial setup only |
| ABN incremental (primary) | 0x25–0x2B | Up to 2 MHz; 24-bit PPR (`ABN_DECODER_PPR` 0x26); N-pulse initialization |
| ABN incremental (secondary) | 0x2C–0x31 | Velocity/position only; **cannot** drive phi_e for FOC commutation |
| Digital Hall | 0x33–0x3A | 6-step; optional PWM-center sync sampling; blanking 0–4095 ns (10 ns steps); interim interpolation active >60 rpm |
| Analog Hall / SinCos | 0x3B–0x48 | AENC_UX/VN/WY differential inputs; hardware ATAN2 decoder; 0°/90° (2-phase) or 0°/120°/240° (3-phase) |
| External phi_e | 0x52 = 1 | 16-bit angle written directly by host |

**Reference switches:** REF_SW_L (pin 67), REF_SW_H (pin 68), REF_SW_R (pin 69) — position latched on edge.

**PHI_E_SELECTION register 0x52:**
- 1 = External (host-written)
- 2 = Open loop
- 3 = ABN primary encoder
- 5 = Digital Hall
- 6 = Analog encoder (AENC)
- 7 = phi_a from AENC

**Known errata (LA):** Hall interpolation + position mode may produce glitches on `PID_POSITION_ACTUAL` (0x6B) near zero speed. Workaround: disable hall interpolation (HALL_MODE 0x33 bit 8 = 0) when positioning.

---

## Interfaces

**Application SPI:** 40-bit datagrams (1 RW bit + 7 address bits + 32 data bits). SPI mode 3 (CPOL=1, CPHA=1). Write up to 8 MHz; read up to 8 MHz with 500 ns pause after address byte, or 2 MHz without pause.

**UART:** 3-pin (GND/RxD/TxD), 3.3 V, 1N8 format. Baud rate via register 0x79 `UART_BPS`:
- 0x00009600 → 9600 bps (default)
- 0x00115200 → 115200 bps
- 0x00921600 → 921600 bps
- 0x03000000 → 3 Mbps

**Step/Dir:** STP (pin 57) + DIR (pin 56) inputs. Step size set by `STEP_WIDTH` (0x78, s32).

**Debug SPI:** Shared with GPIO3–7 pins; multiplexed `DBGSPI_nSCS/SCK/MOSI/MISO/TRG`.

**RTMI (Real-Time Monitoring Interface):** High-speed telemetry output via Hirose DF20F-10DP-1V connector. Requires USB-2-RTMI adapter board for use with TMCL-IDE.

**Single Pin Interface (PWM_I):** Pin 58. Analog target input; motion mode selectable (torque / velocity / position via `AGPI_A` or `PWM_I` entries in MODE_RAMP_MODE_MOTION).

**GPIO0–GPIO7:** Pins 70, 71, 74–76, 1, 4, 5. Dual-function: GPIO or delta-sigma demodulator clocks/data (`MCLKI/MCLKO/MDAC`) or debug SPI.

**ENI / ENO:** ENI (pin 55) enables controllers and PWM outputs. ENO (pin 32) mirrors ENI when CLK is running and IC is not in reset.

---

## ADC Configuration

The TMC4671 uses internal delta-sigma (ΔΣ) ADCs with Sinc3 decimation filters. Two groups (A and B) have independent MCLK and MDEC decimation settings.

**Current sensing inputs:**
- `ADC_I0_POS/NEG` (pins 16/17) → phase I_U (3-phase) or I_X (2-phase)
- `ADC_I1_POS/NEG` (pins 18/19) → phase I_V or I_W (3-phase) or I_Y (2-phase)
- Third phase current calculated by Kirchhoff: I2 = –(I0 + I1)

**⚠ Shunt resistors must NOT connect directly to ADC pins.** Inputs are low-voltage differential; an external shunt amplifier (e.g. AD8418A or LT1999) is required to scale and level-shift the signal into the 0–5 V ADC range.

**Analog inputs:**
- `ADC_VM` (pin 20): motor supply voltage divider; drives brake chopper logic
- `AGPI_A` (pin 21): general purpose analog; usable as analog target via single-pin interface
- `AGPI_B` (pin 22): general purpose analog
- `AENC_UX/VN/WY` (pins 25–30): analog encoder / Hall inputs, differential

**Recommended input range:** 25%–75% of V5 (1.25 V – 3.75 V for 5 V supply) to avoid ΔΣ non-linearity near rails.

**Decoupling:** 100 nF at every supply pin; additional 4.7 µF at V5 and 4.7 µF + 470 nF at VCCIO.

**External ΔΣ modulators:** Supported on GPIO pins (AD7401, AD7402, or R-C-R-CMP comparator front end).

**Key ADC registers:**

| Address | Register | Default |
|---|---|---|
| 0x04 | dsADC_MCFG_B_MCFG_A | ΔΣ mode config |
| 0x07 | dsADC_MDEC_B_MDEC_A | Decimation (default 0x0100_0100) |
| 0x08 | ADC_I1_SCALE_OFFSET | I1 scale + offset |
| 0x09 | ADC_I0_SCALE_OFFSET | I0 scale + offset |
| 0x0A | ADC_I_SELECT | Map ADC channels to FOC inputs |
| 0x75 | ADC_VM_LIMITS | Brake chopper VM thresholds |

---

## PWM Modes

**Mode register:** `PWM_SV_CHOP` at 0x1A. Power-on default: `PWM_CHOP = 0` (PWM off, free-running). **Must set `PWM_CHOP = 7`** (centered PWM) before enabling FOC.

| `PWM_CHOP` | Mode |
|---|---|
| 0 | Off, free-running |
| 1 | Off, low-side permanent ON |
| 2 | Off, high-side permanent ON |
| 7 | Centered (required for FOC) |

**PWM frequency:**
```
fPWM = (4 × fCLK) / (PWM_MAXCNT + 1)
     = 100 MHz / (PWM_MAXCNT + 1)
Default: PWM_MAXCNT = 0xF9F = 3999 → fPWM = 25 kHz
```

`PWM_MAXCNT` is a 16-bit field at register 0x18.

**Break-before-make (BBM):** Register 0x19, fields `PWM_BBM_L` and `PWM_BBM_H`. Default 0x14 = 20 × 10 ns = 200 ns for each side.

**Space Vector PWM (SVPWM):** `PWM_SV` bit at 0x1A[8]. Default OFF. When enabled, provides ~12% higher effective voltage utilization. **Only suitable for 3-phase motors with isolated star point. Must remain OFF for FOC2/FOC1.** (LA fix: SVPWM was non-functional in ES; LA correctly adds +12% voltage in torque mode.)

**PWM outputs:** 8 pins for up to 4 half-bridges — PWM_UX1_H/L, PWM_VX2_H/L, PWM_WY1_H/L, PWM_Y2_H/L. Polarity configured via `PWM_POLARITIES`.

---

## Required External Components

| Component | Purpose |
|---|---|
| TMC6100 (or equivalent gate driver) | Level-shifted gate drive for power MOSFETs |
| Sense resistors | Phase current shunts |
| Shunt amplifier (e.g. AD8418A, LT1999) | Scale/shift shunt voltage into ADC input range |
| 25 MHz crystal oscillator | CLK input (mandatory) |
| 100 nF caps at every supply pin | Decoupling |
| 4.7 µF at V5; 4.7 µF + 470 nF at VCCIO | Bulk decoupling |
| 10 kΩ pull-up on nRST | Reset line when not actively driven |
| Voltage divider on ADC_VM | Scale VM to ADC range for brake chopper |
| Encoder interface protection (R+diodes, see Fig. 42) | Protect 3.3 V inputs from 5 V encoder signals |

The TMC4671 has no internal current limiting or protection. System protection (overcurrent, overtemperature) must be implemented externally.

---

## Motion Modes

Set via `MODE_RAMP_MODE_MOTION` (register 0x63):

| Value | Mode |
|---|---|
| 0 | Stopped |
| 1 | Torque mode |
| 2 | Velocity mode |
| 3 | Position mode |
| 4–7 | PRBS test (Flux/Torque/Velocity/Position) |
| 8 | UQ/UD external (open-loop voltage injection) |
| 10–12 | AGPI_A analog input (Torque/Velocity/Position) |
| 13–15 | PWM_I input (Torque/Velocity/Position) |

---

## Key Registers

| Address | Name | Description |
|---|---|---|
| 0x00 | CHIPINFO_DATA | Stacked; read chip ID and version |
| 0x04 | dsADC_MCFG_B_MCFG_A | Delta-sigma ADC mode configuration |
| 0x07 | dsADC_MDEC_B_MDEC_A | Decimation register (default 0x0100_0100) |
| 0x08 | ADC_I1_SCALE_OFFSET | I1 ADC scale (s16) + offset (s16) |
| 0x09 | ADC_I0_SCALE_OFFSET | I0 ADC scale (s16) + offset (s16) |
| 0x0A | ADC_I_SELECT | Map ADC channels to FOC current inputs |
| 0x18 | PWM_MAXCNT | PWM period register (default 0xF9F = 25 kHz) |
| 0x19 | PWM_BBM_H_BBM_L | Break-before-make times (default 0x1414 = 200 ns each) |
| 0x1A | PWM_SV_CHOP | PWM mode (bits[2:0]) + SVPWM enable (bit 8) |
| 0x1B | MOTOR_TYPE_N_POLE_PAIRS | Motor type (bits[19:16]) + NPP (bits[15:0]) |
| 0x25 | ABN_DECODER_MODE | ABN decoder configuration |
| 0x26 | ABN_DECODER_PPR | ABN encoder pulses per revolution (24-bit) |
| 0x33 | HALL_MODE | Hall mode; bit 8 = interpolation enable |
| 0x50 | VELOCITY_SELECTION | Source for velocity feedback |
| 0x51 | POSITION_SELECTION | Source for position feedback |
| 0x52 | PHI_E_SELECTION | Electrical angle source for FOC |
| 0x54 | PID_FLUX_P_FLUX_I | Flux current PI parameters |
| 0x56 | PID_TORQUE_P_TORQUE_I | Torque current PI parameters |
| 0x58 | PID_VELOCITY_P_VELOCITY_I | Velocity PI parameters |
| 0x5A | PID_POSITION_P_POSITION_I | Position P(I) parameters |
| 0x5D | PIDOUT_UQ_UD_LIMITS | Circular output voltage limiter (default 0x5A81) |
| 0x63 | MODE_RAMP_MODE_MOTION | Motion mode + PI structure selection (bit 31) |
| 0x64 | PID_TORQUE_FLUX_TARGET | Torque (bits[31:16]) + Flux (bits[15:0]) targets (s16 each) |
| 0x66 | PID_VELOCITY_TARGET | Velocity target (s32) |
| 0x68 | PID_POSITION_TARGET | Position target (s32) |
| 0x69 | PID_TORQUE_FLUX_ACTUAL | Actual torque + flux readback |
| 0x6A | PID_VELOCITY_ACTUAL | Actual velocity readback |
| 0x6B | PID_POSITION_ACTUAL | Actual position; write also sets target |
| 0x75 | ADC_VM_LIMITS | Brake chopper high/low thresholds |
| 0x78 | STEP_WIDTH | Step/Dir step size (s32) |
| 0x79 | UART_BPS | UART baud rate selector |
| 0x7C | STATUS_FLAGS | 32-bit status and interrupt flags |
| 0x7D | STATUS_MASK | Mask for STATUS output pin |

---

## Application Notes

Trinamic provides the following collateral at www.trinamic.com:

- **TMC4671 Application Note — How to turn a motor:** Step-by-step commissioning guide (SPI setup → PWM config → open loop → ADC calibration → sensor setup → PI tuning)
- **TMC API:** C source library for register abstraction
- **TMCL-IDE:** GUI with RTMI real-time plotting and automated PI controller tuning tools
- **TMC-UPS-10A70V-A-EVAL / TMC-UPS-2A24V-A-EVAL:** Reference power stage boards with matched current shunt circuitry

**Commissioning sequence:**
1. Verify SPI communication (read CHIPINFO_DATA 0x00)
2. Set MOTOR_TYPE_N_POLE_PAIRS (0x1B)
3. Configure PWM (0x18, 0x19, 0x1A); set PWM_CHOP = 7
4. Verify PWM outputs before connecting motor
5. Enable open-loop mode (PHI_E_SELECTION = 2, MODE = 8), rotate motor
6. Calibrate ADC offsets/gains (0x08, 0x09); verify current sign/phase matching
7. Configure feedback sensor; compare sensor angle vs open-loop angle
8. Tune flux/torque PI (0x54, 0x56), then velocity PI (0x58), then position P (0x5A)
9. Select motion mode and apply targets

---

## Comparison with Stepper Drivers

| Feature | TMC4671-LA | TMC2209 / TMC2240 / TMC5160A |
|---|---|---|
| Function | FOC servo controller (no power stage) | Integrated driver + controller |
| Power stage | External required (e.g. TMC6100) | Integrated MOSFETs |
| Motor types | BLDC, stepper, DC, voice coil | Stepper (some support DC) |
| Current control | Hardware FOC, torque/flux separated | Chopper current regulation |
| Position feedback | ABN encoder, Hall, analog sin/cos | Encoder input (basic) or none |
| Sensorless | Not supported | StallGuard (load-based, no position) |
| Control cascade | Position → Velocity → Torque in hardware | Velocity + current in hardware |
| Host interface | SPI + UART + Step/Dir | SPI + UART + Step/Dir |
| Typical application | Servo axis, BLDC spindle, robotics | Open-loop or simple closed-loop steppers |

---

## Evaluation Ecosystem

| Order Code | Description | Size |
|---|---|---|
| TMC4671-LA | IC only | QFN76, 10.5 × 6.5 mm |
| TMC4671-EVAL | Full evaluation board | 55 × 85 mm |
| TMC4671-BOB | Breakout board | 38 × 40 mm |
| Landungsbruecke | MCU controller board | 85 × 55 mm |
| TMC-UPS-2A24V-A-EVAL | 2 A / 24 V power stage | — |
| TMC-UPS-10A70V-A-EVAL | 10 A / 70 V power stage | — |
| TMC4671-2A24V-EV-KIT | Complete kit (2 A / 24 V) | — |
| TMC4671-10A70V-EV-KIT | Complete kit (10 A / 70 V) | — |
| USB-2-RTMI | RTMI USB adapter | 40 × 20 mm |

---

## Selection Notes

- Use TMC4671-LA when closed-loop servo performance is required: BLDC spindles, robotics joints, precision linear stages.
- **Not a drop-in replacement for stepper drivers** — requires external power stage, gate driver, current sense amplifiers, and a feedback sensor.
- For stepper-only applications without FOC, [[TMC2240]] or [[TMC5160A]] integrate driver + controller and are simpler to deploy.
- The -LA suffix denotes the production release; -ES was engineering sample (multiple errata); -ES2 was intermediate. LA fixes all 13 known ES errata including SPI read MSB corruption, SVPWM scaling, RTMI timing, and ADC crosstalk.
- TMC4671-LA is drop-in pin-compatible with TMC4671-ES/ES2 but firmware must be re-validated due to behavior changes (SVPWM +12%, advanced PI scaling, ENI/ENO function update).
