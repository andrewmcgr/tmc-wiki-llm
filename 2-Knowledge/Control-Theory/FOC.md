# Field-Oriented Control (FOC)

**Type:** knowledge
**Summary:** Synthesis of Field-Oriented Control (FOC) theory and its application to stepper motors and PMSMs. Covers d-q frame modeling, Clarke/Park transforms, hardware implementation via TMC4671, and advanced control strategies like SMC.
**Tags:** #control-theory #foc #motion #pmsm #stepper #reference
**Status:** active
**Updated:** 2026-04-17
**Related:** [[TMC4671-LA]], [[Sliding-Mode-Control]], [[SMC-PMSM-Survey]], [[tmc-4671]], [[k4671]], [[GESO-SMC-RL-Feed]]

---

## Overview

Field-Oriented Control (FOC), also known as vector control, is a control strategy that allows for the independent control of magnetic flux and electromagnetic torque in AC motors. By transforming the three-phase (or two-phase) stationary currents into a two-axis rotating coordinate system (the d-q frame), FOC enables AC motors to be controlled with the same simplicity and performance as separately excited DC motors.

In the context of this wiki, FOC is the bridge that allows **stepper motors** to be treated as high-pole-count **Permanent Magnet Synchronous Motors (PMSM)**, unlocking high-speed performance, reduced heat, and silent operation.

---

## Theoretical Basis

### Coordinate Transformations

FOC relies on two primary mathematical transforms to map physical phase currents to the rotating reference frame:

1.  **Clarke Transform (α-β frame):** Converts the stationary three-phase (a-b-c) or two-phase (X-Y) currents into a stationary two-axis orthogonal system (α-β).
2.  **Park Transform (d-q frame):** Rotates the α-β frame to align with the rotor's magnetic flux vector using the electrical angle ($\phi_e$).
    *   **d-axis (Direct):** Aligned with the rotor flux; controls magnetization.
    *   **q-axis (Quadrature):** Orthogonal to the rotor flux; controls electromagnetic torque.

### d-q Frame System Model

The standard model for a surface-mounted PMSM (and FOC-driven stepper) in the d-q frame is:

**Voltage Equations:**
$$u_d = R_s \cdot i_d + L_d \cdot \frac{di_d}{dt} - \omega_e \cdot L_q \cdot i_q$$
$$u_q = R_s \cdot i_q + L_q \cdot \frac{di_q}{dt} + \omega_e \cdot L_d \cdot i_d + \omega_e \cdot \psi_f$$

**Electromagnetic Torque:**
$$T_e = \frac{3}{2} \cdot p_n \cdot \psi_f \cdot i_q$$

Where:
*   $R_s, L_s$: Winding resistance and inductance.
*   $\psi_f$: Permanent magnet flux linkage.
*   $p_n$: Number of pole pairs.
*   $\omega_e$: Electrical angular velocity.

### Cascade Control Architecture

FOC is typically implemented as a cascaded control loop:
1.  **Current Loop (Inner):** Two PI controllers regulate $i_d$ (usually to zero for maximum torque efficiency) and $i_q$ (torque). Runs at the highest frequency (PWM rate).
2.  **Velocity Loop (Middle):** Regulates motor speed by adjusting the $i_q$ target.
3.  **Position Loop (Outer):** Regulates rotor position by adjusting the velocity target.

---

## Application in Stepper Motors

FOC transforms a traditional open-loop stepper into a high-performance closed-loop servo.

*   **Stepper as PMSM:** A 2-phase stepper motor is modeled as a PMSM with a high number of pole pairs ($NPP = \text{FullSteps} / 4$).
*   **FOC2 Mode:** Hardware like the [[TMC4671-LA]] implements a specific "FOC2" mode that bypasses the Clarke transform, mapping the two physical phases directly to the X-Y frame.
*   **Benefits:**
    *   **Efficiency:** Current is only drawn as needed for the load, drastically reducing heat.
    *   **High Speed:** Eliminates the "mid-band resonance" and torque drop-off typical of open-loop stepping.
    *   **Silence:** Continuous sinusoidal commutation eliminates the audible noise of discrete steps.

---

## Implementation Reference

### Hardware: TMC4671-LA
The [[TMC4671-LA]] is the primary hardware reference in this wiki. It implements the entire FOC signal chain (transforms, PI loops, SVPWM) in silicon, updating at the PWM frequency (~25-100 kHz). It supports:
*   **FOC3:** 3-phase BLDC/PMSM.
*   **FOC2:** 2-phase Stepper.
*   **FOC1:** DC/Voice Coil.

### Software & Firmware
*   **[[tmc-4671]] (Klipper):** A Python extension that delegates real-time control to the TMC4671 hardware while providing a "Virtual Stepper" interface to the motion planner.
*   **[[k4671]] (Rust/Embassy):** A firmware implementation that uses a hybrid approach, combining hardware FOC with software-based feedforward compensation to overcome hardware limitations.

---

## Advanced Control Strategies

Beyond standard PI-based FOC, several advanced techniques are documented in the wiki for handling disturbances and non-linearities:

*   **[[Sliding-Mode-Control]] (SMC):** A robust non-linear control method that replaces the PI velocity/position loops to provide better rejection of load disturbances and parameter uncertainties.
*   **[[GESO-SMC-RL-Feed]]:** Combines SMC with a Generalized Extended State Observer (GESO) to estimate and compensate for both matched (cogging torque) and mismatched (load) disturbances.
*   **[[Feedforward-Compensators]]:** Used to improve tracking performance by injecting predicted torque requirements (based on acceleration/velocity) directly into the current loop.

---

## References to Wiki Pages

*   **Hardware:** [[TMC4671-LA]], [[TMC-Driver-Comparison]]
*   **Theory:** [[Sliding-Mode-Control]], [[SMC-PMSM-Survey]], [[Feedforward-Compensators]]
*   **Firmware/Software:** [[k4671]], [[tmc-4671]], [[k4671-Feedforward]]
*   **Advanced Research:** [[GESO-SMC-RL-Feed]], [[SMC-PI-Cascade]]
