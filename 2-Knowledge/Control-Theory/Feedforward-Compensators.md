# Feedforward Compensators

**Type:** knowledge
**Summary:** Feedforward compensator design for process and motion control — ideal FF form, practical lead-lag realization, FOPDT-based tuning rules (Guzmán & Hägglund 2011), Lambda feedback tuning, ARX identification, and velocity/acceleration feedforward for servo systems.
**Tags:** #control-theory #feedforward #pid #tuning #motion
**Status:** draft
**Updated:** 2026-04-08
**Source:** Montoya-Ríos et al., "Simple Tuning Rules for Feedforward Compensators," Agronomy 2020; Simple_Tuning_Rules_for_Feedforward_Compensators_A.pdf; Guzmán & Hägglund, JPC 2011
**Related:** [[PID-Tuning-Methods]], [[SIMC-PID-Tuning]]

---

## Overview

Feedforward control uses knowledge of a measurable disturbance (or the setpoint trajectory) to act *before* the error appears, rather than reacting to it like feedback control. In motion control, feedforward is essential for achieving high tracking accuracy — a well-tuned PID alone cannot eliminate tracking error during acceleration/deceleration without excessive gains that compromise stability.

The combination of feedback (PID) + feedforward is standard practice in CNC, robotics, servo drives, and process control (e.g., greenhouse climate regulation).

## Architecture

```
           d (measurable disturbance)
           │
        [Gd(s)]
           │
           ▼
r ──►[Gff]──►(+)──►[Plant G(s)]──► y
  │              ▲
  └──►(−)──►[C(s)]┘
        ▲       (feedback)
        │
        y
```

- **C(s)** — feedback controller (PID / PI)
- **Gff(s)** — feedforward compensator
- **G(s)** — plant (process) transfer function
- **Gd(s)** — disturbance-to-output transfer function
- **r** — reference/setpoint
- **d** — measurable disturbance

The feedback loop handles unmeasured disturbances and model uncertainty. The feedforward path cancels the predictable, measurable disturbance component before it reaches the output.

## Ideal Feedforward

For perfect disturbance rejection, the ideal feedforward compensator is:

```
Gff_ideal(s) = −Gd(s) / G(s)
```

For setpoint feedforward (reference tracking):

```
Gff_ideal(s) = G(s)⁻¹
```

### Why Ideal is Impractical

- **Non-causal:** G⁻¹ often has more zeros than poles (requires predicting the future)
- **Unstable:** If G has right-half-plane zeros, G⁻¹ has RHP poles
- **Dead time conflict:** If dead time of Gd < dead time of G, `Gff = −Gd/G` contains `e^(+θs)` — non-realizable
- **Sensitivity:** Any model mismatch causes feedforward error that feedback must correct

In practice, the non-realizable delay terms are omitted and a realizable approximation is used instead.

## Practical Feedforward: Lead-Lag Filter Realization

Both G and Gd are modeled as FOPDT (First-Order Plus Dead Time) transfer functions:

```
G(s)  = k  · e^(−Ls)  / (τ·s + 1)
Gd(s) = kd · e^(−Ld·s) / (τd·s + 1)
```

The feedforward compensator is realized as a lead-lag filter:

```
Gff(s) = kff · (τd·s + 1) / (τp·s + 1) · e^(−(Ld − L)·s)   [if Ld ≥ L]
```

When `Ld < L` (dead time of disturbance is smaller than process dead time), perfect delay cancellation is impossible. The simple tuning rules below handle this case using the feedback controller to partially compensate.

## Guzmán & Hägglund Tuning Rules (JPC 2011)

The rules below are from Guzmán & Hägglund, *J. Process Control* 2011 (Ref [23] in Montoya-Ríos et al.), targeting **IAE minimization** and **no overshoot** in disturbance rejection.

**Assumptions:** G and Gd described by FOPDT models; PI feedback controller with parameters kp and Ti; feedforward realized as first-order lead-lag.

### Step 1 — Set the lead time constant

```
τd  (time constant of disturbance model Gd)
```

### Step 2 — Calculate the pole τp

```
τp = τd + (L − Ld)   if  0 < (L − Ld) < 1.7·τd
τp = τd               if  (L − Ld) ≥ 1.7·τd  (or Ld ≥ L)
```

Where L is the process dead time and Ld is the disturbance dead time. When `Ld < L` (causality problem), the excess delay `(L − Ld)` is absorbed into the pole, allowing the filter to take over partial compensation.

### Step 3 — Calculate the feedforward gain kff

From process parameters (k, τ) and PI tuning (kp, Ti):

```
kff = −(kd / k) · (τ / Ti) · (1 / (kp · kd))
```

More concisely, the gain is derived from the ratio of static gains adjusted for the PI reset time, ensuring steady-state disturbance cancellation.

The full compensator:

```
Gff(s) = kff · (τd·s + 1) / (τp·s + 1)
```

### SIMC Improvement Note

When using SIMC-based feedback tuning (see [[SIMC-PID-Tuning]]), the integral time Ti uses the "improved" SIMC rule: replace τ₁ by `τ₁ + θ/3`. This reduces overshoot in setpoint response and slightly modifies the FF gain calculation.

## Lambda Tuning for the Feedback PI Controller

Montoya-Ríos et al. used the **Lambda (λ) method** to tune the PI controller before designing the feedforward compensator. Given a FOPDT process model (k, τ, L):

```
kp = τ / (k · (τcl + L))
Ti = τ
```

Where `τcl` is the desired closed-loop time constant. Typical choice: `τcl = 0.3·τ` for fast disturbance rejection.

The PI (not PID) is sufficient when derivative action would amplify sensor noise — common in real facilities with noisy signals.

**Anti-windup:** Include back-calculation anti-windup with tracking constant `Tt = √Ti` to prevent integrator windup against actuator saturation limits.

## System Identification: ARX Method

When a first-principles model is unavailable, black-box identification from input-output data provides the FOPDT models needed for tuning.

### High-Order ARX Model

An Auto-Regressive with eXogenous input (ARX) model captures the complex dynamics:

```
A(z)·y(k) = B₁(z)·u₁(k−nk₁) + ... + Bₙ(z)·uₙ(k−nkₙ) + e(k)
```

Where A(z) and Bᵢ(z) are polynomials with orders na and nb. For a greenhouse (MISO), inputs are: ventilation opening, external solar radiation, external air temperature, external wind velocity.

**Procedure:**
1. Run open-loop step tests with varying input profiles
2. Record input-output data (sample time 30 s for greenhouse, faster for servo systems)
3. Fit ARX model using least-squares (MATLAB System Identification Toolbox or equivalent)
4. Validate on independent data — mean absolute error < 0.5 °C is acceptable for greenhouse

### Model Reduction to FOPDT

From the high-order ARX model, reduce to FOPDT transfer functions per input-output pair:

```
Gᵢ(s) = kᵢ · e^(−Lᵢ·s) / (τᵢ·s + 1)
```

Use the **reaction curve method**: apply a unit step to input i (zero all others), extract the ARX model response, fit k, τ, L from the transient. For the greenhouse example, four transfer functions resulted:

| Model | Input | Notes |
|---|---|---|
| Gu(s)  | Ventilation opening | Process model G(s) for PI tuning |
| Gd1(s) | Solar radiation | Disturbance model — FF compensator 1 |
| Gd2(s) | External air temperature | Disturbance model — FF compensator 2 |
| Gd3(s) | Wind velocity | Disturbance model — FF compensator 3 |

## Greenhouse Application: Experimental Results

Montoya-Ríos et al. (2020) applied this methodology to daytime temperature control of a 877 m² parral-type greenhouse in Almería, Spain.

**Identified models (representative, rounded):**

The PI controller was tuned with kp = −105.3 [%/°C] and Ti = 1601 s (τcl = 0.3·τ).

**Feedforward compensators (simple tuning rules vs. classical):**

| Disturbance | Classical Gff*(s) | Tuned Gff(s) [Guzmán-Hägglund] |
|---|---|---|
| Solar radiation | `−796.17s − 0.497 / (1586s + 1)` | `−741.9s − 0.4635 / (1509s + 1)` |
| Air temperature | `−10.2·10⁴s − 63.47 / (1440s + 1)` | `−8.6·10⁴s − 53.7 / (1267s + 1)` |
| Wind velocity | `5.7·10⁴s + 35.59 / (1259s + 1)` | `4.78·10⁴s + 29.88 / (1079s + 1)` |

The tuned rules yielded tighter poles (smaller τp) due to the causality problem — dead times of disturbances were shorter than the process dead time, so the excess delay was absorbed into the pole.

**IAE comparison (simulation):**

| Control strategy | IAE — 14 Mar 2020 | IAE — 1 May 2020 |
|---|---|---|
| PI only | 168.98 | 55.88 |
| PI + Classical FF | 115.12 | 18.67 |
| PI + Tuned FF (Guzmán-Hägglund) | **114.91** | **15.27** |

Both FF strategies dramatically outperformed PI alone. The tuned rules achieved the lowest IAE in both cases — the improvement is modest per day but accumulates significantly over a crop season.

**Real tests:** Three days of experimental validation in spring 2020. Control error contained within ±1 °C despite actuator resolution limitations (10% steps) and non-measurable disturbances. Best result on Day 2 (all three FF compensators active): inside temperature tracked setpoint through solar noon with only mild deviations from wind velocity transients.

**First experimental validation** of these Guzmán & Hägglund rules in a real process — prior work (including the 2011 JPC paper) was simulation only.

## Motion Control Feedforward

In servo/stepper motor position control, feedforward takes a different (more direct) form based on the kinematic trajectory:

### Velocity Feedforward

```
u_ff_vel = Kv · ṙ(t)
```

Where ṙ(t) is the commanded velocity (derivative of the position setpoint). Kv is the inverse of the velocity loop steady-state gain. Eliminates the following error proportional to velocity.

### Acceleration Feedforward

```
u_ff_acc = Ka · r̈(t)
```

Where r̈(t) is the commanded acceleration. Ka is set from motor inertia and torque constant. Eliminates the error proportional to acceleration.

### Combined

```
u_total = u_PID + Kv · ṙ(t) + Ka · r̈(t)
```

Most servo drives (including TMC4671-based systems) support velocity and acceleration feedforward natively. The PID handles residual errors, disturbances, and model mismatch.

### Tuning Velocity/Acceleration Feedforward

| Parameter | Ideal value | How to determine |
|---|---|---|
| **Kv** | 1/K_vel (inverse velocity loop gain) | Measure steady-state velocity per unit input, invert |
| **Ka** | J/k_t (inertia / torque constant) | From motor datasheet + load inertia calculation |

**Practical tuning procedure:**
1. Tune PID first (e.g., via SIMC with τc = θ)
2. Set Kv: run constant-velocity moves, increase Kv until following error at constant velocity → 0
3. Set Ka: run acceleration profiles, increase Ka until error during acceleration → 0
4. Re-check PID — feedforward may allow reducing PID gains for better stability

## Process Control vs. Motion Control FF: Key Distinction

| Aspect | Process control (Guzmán-Hägglund) | Motion control (velocity/acceleration) |
|---|---|---|
| **Disturbance type** | External measurable disturbances (temperature, radiation, load) | Known trajectory (position setpoint derivative) |
| **Model basis** | FOPDT from system ID (ARX → step response fit) | Motor kinematic model (inertia, gain) |
| **Compensator form** | Lead-lag filter Kff·(τd·s+1)/(τp·s+1) | Direct gain × trajectory derivative |
| **Tuning objective** | Minimize IAE for disturbance rejection | Minimize following error during motion |
| **Feedback interaction** | PI parameters affect kff calculation | PID tuned first; FF reduces required PID gain |

The Guzmán-Hägglund rules are directly applicable to process control disturbances in embedded systems — e.g., feedforward from a measurable load torque or supply voltage change — not just greenhouse climate control.

## When Feedforward Helps Most

Feedforward provides the largest benefit when:

- **Disturbance is measurable** (or the setpoint trajectory is known in advance)
- **Plant has significant dead time** — feedback alone cannot react quickly enough
- **High tracking accuracy is required** — following error during motion must be small
- **PID gains are limited** by stability (resonance, noise, delay)

Feedforward provides **no benefit** for:

- Unmeasured/unpredictable disturbances (use feedback or disturbance observer)
- Steady-state regulation (feedback handles this fine with integral action)
- Plants with highly uncertain dynamics (feedforward mismatch adds error)

## References

- Guzmán, J.L. & Hägglund, T. "Simple tuning rules for feedforward compensators." *J. Process Control* 21(1), 92–102, 2011. — Primary source for the tuning rules.
- Montoya-Ríos, A.P.; García-Mañas, F.; Guzmán, J.L.; Rodríguez, F. "Simple Tuning Rules for Feedforward Compensators Applied to Greenhouse Daytime Temperature Control Using Natural Ventilation." *Agronomy* 10(9), 1327, 2020. — Experimental validation; source paper for this page.
- Guzmán, J.L.; Hägglund, T.; Veronesi, M.; Visioli, A. "Performance indices for feedforward control." *J. Process Control* 26, 26–34, 2015.
- Åström, K.J. & Hägglund, T. *Advanced PID Control.* ISA, 2006. — Lambda tuning, anti-windup, PID theory.
- Skogestad, S. & Postlethwaite, I. *Multivariable Feedback Control.* Wiley, 2nd ed., 2005. (Chapter 10 — feedforward and ratio control)
- Ellis, G. *Control System Design Guide.* Academic Press, 4th ed., 2012. (Chapters on feedforward in servo systems)
