# PID Tuning Methods

**Type:** knowledge
**Summary:** Comparative reference for PID tuning methods — ZN, Cohen-Coon, Lambda/IMC, SIMC, relay autotune, Tyreus-Luyben, direct synthesis. Enriched with Pareto-optimality data, robustness margins, and method comparison from Skogestad & Grimholt (2012).
**Tags:** #control-theory #pid #tuning #motion #reference
**Status:** draft
**Updated:** 2026-04-08
**Source:** pidbook-chapter5.pdf (Skogestad & Grimholt 2012); Åström & Hägglund 2006
**Related:** [[SIMC-PID-Tuning]], [[Feedforward-Compensators]]

---

## Overview

PID controllers remain the workhorse of industrial and embedded control. The challenge is rarely the controller structure — it's choosing gains (Kp, Ki, Kd or equivalently Kc, τI, τD) that deliver acceptable performance. This page surveys the major tuning methods, their assumptions, and when to use each.

All methods below assume a standard FOPDT plant model unless otherwise noted:

```
G(s) = K · e^(−θs) / (τs + 1)
```

Where K is the static gain, τ is the dominant time constant, and θ is the dead time.

**Key insight from Skogestad & Grimholt:** ZN settings result in a very good disturbance response for integrating processes but are otherwise aggressive and give poor performance for processes with a dominant delay. IMC-derived settings are robust with good setpoint response but give poor disturbance rejection for integrating processes. SIMC was designed to work well across both cases.

## Method Comparison

| Method | Model needed? | Closed-loop test? | Robustness | Aggressiveness | Best for |
|---|---|---|---|---|---|
| Ziegler-Nichols (OL) | FOPDT | No | Low | High | Fast first pass |
| Ziegler-Nichols (CL) | No | Yes (Ku, Pu) | Low | High | Unknown plant model |
| Cohen-Coon | FOPDT | No | Medium | Medium | High dead-time plants |
| Lambda/IMC | FOPDT | No | High | Adjustable | Smooth setpoint tracking |
| SIMC | FOPDT or SOPDT | No | Adjustable (τc) | Adjustable | General purpose, near-optimal |
| Relay autotune | No | Yes (relay) | Medium | Medium | Online/adaptive tuning |
| Tyreus-Luyben | No | Yes (Ku, Pu) | High | Low | Stability-critical |
| Direct synthesis | Model | No | Depends | Depends | Known plant dynamics |

## Ziegler-Nichols (Open-Loop)

The original heuristic method. Perform an open-loop step test, fit K, τ, θ from the response.

| Controller | Kc | τI | τD |
|---|---|---|---|
| P | τ/(2Kθ) | — | — |
| PI | 0.9τ/(Kθ) | 3θ | — |
| PID | 1.2τ/(Kθ) | 2θ | 0.5θ |

**Pros:** Simple, widely known, no model beyond FOPDT.
**Cons:** Aggressive — designed for quarter-decay ratio, typically produces ~25% overshoot. Poor disturbance rejection for processes with large dead time. Only uses two model parameters (Ku, Pu) so cannot span the full range of FOPDT processes (which require three: K, τ, θ).

### ZN Robustness Data (Skogestad & Grimholt)

ZN closed-loop settings are known to result in poor robustness margins compared to SIMC at equivalent τc = θ:

- **Typical ZN:** Ms ≈ 2.0–2.5 (sensitivity peak), GM ≈ 1.5–1.8, PM ≈ 25–35°
- **SIMC at τc = θ:** Ms ≈ 1.59–1.70, GM ≈ 2.96–3.14, PM ≈ 46.9–61.4°

The SIMC `Ms < 1.7` threshold guarantees GM > 2.43 and PM > 34.2°. ZN frequently violates the typical minimum requirements of GM > 1.7 and PM > 30°.

## Ziegler-Nichols (Closed-Loop / Ultimate Gain)

Increase proportional gain with integral/derivative off until sustained oscillation. Record ultimate gain Ku and ultimate period Pu.

| Controller | Kc | τI | τD |
|---|---|---|---|
| P | Ku/2 | — | — |
| PI | 0.45·Ku | Pu/1.2 | — |
| PID | 0.6·Ku | Pu/2 | Pu/8 |

**Pros:** No plant model needed. Works on any stable plant.
**Cons:** Requires driving the plant to the stability boundary — dangerous for some systems. Same aggressiveness issues as OL method. The method only captures Ku and Pu, so it fundamentally cannot distinguish between two different FOPDT processes with the same Ku/Pu ratio but different θ/τ ratios — limiting its generality.

## Cohen-Coon

Designed for processes with significant dead time (θ/τ > 0.3). Uses FOPDT model but with modified correlations that account for the dead-time-to-lag ratio.

For P-only control:

```
Kp = (1/K) · (τ/θ + 1/3)
```

PI and PID formulas involve more complex polynomial expressions of the θ/τ ratio. Generally gives better disturbance rejection than ZN for dead-time-dominant processes, with somewhat less overshoot.

**Best for:** Chemical processes, thermal systems, or any plant where θ/τ > 0.3.

## Lambda / IMC Tuning

Model-based method where λ (lambda) is the desired closed-loop time constant. The controller is derived from internal model control (IMC) theory.

For PI control of FOPDT:

```
Kp ≈ (2τ + θ) / (K · 2λ)
τI = τ + θ/2
```

**Key property:** λ directly controls the speed-robustness tradeoff:
- Small λ → fast but fragile
- Large λ → slow but robust
- Rule of thumb: λ ≥ 0.8θ for robustness

**Known limitation (per Skogestad & Grimholt):** The analytically derived IMC settings of Rivera et al. are known to result in *poor disturbance response for integrating processes* (the τI = τ + θ/2 rule makes τI very large for lag-dominant plants, causing slow disturbance rejection). IMC performs well for setpoints but not for load disturbances.

SIMC (see [[SIMC-PID-Tuning]]) is a practical refinement of Lambda/IMC tuning with simpler rules and the τc parameter playing the same role as λ, but with a critical modification that caps τI at 4(τc + θ) to fix the integrating-process disturbance problem.

## SIMC (Skogestad IMC)

Two-step model-based procedure. Analytically derived, a single tuning parameter τc, near-optimal performance. Full detail in [[SIMC-PID-Tuning]].

**Step 1:** Obtain first- or second-order plus delay model from step response or closed-loop experiment.
**Step 2:** Apply rules:

For PI (first-order model):
```
Kc = (1/K) · τ1 / (τc + θ)
τI = min(τ1, 4(τc + θ))
```

For PID (second-order model, τ2 > θ):
```
Kc = (1/K) · τ1 / (τc + θ)
τI = min(τ1, 4(τc + θ))
τD = τ2   [series form]
```

**Tuning parameter:** τc = θ gives "tightest practical control" (Ms ≈ 1.59–1.70).

### SIMC Robustness Margins at τc = θ

| Process type | Kc | τI | GM | PM | Ms | Mt |
|---|---|---|---|---|---|---|
| First-order (τI = τ1) | 0.5·(τ1/Kθ) | τ1 | 3.14 | 61.4° | 1.59 | 1.00 |
| Integrating (τI = 8θ) | 0.5/(K'θ) | 8θ | 2.96 | 46.9° | 1.70 | 1.30 |

Maximum allowed time delay error: Δθ/θ = 2.14 (first-order), 1.59 (integrating). The system tolerates a 3.14× increase in delay before instability (first-order case).

### SIMC Pareto-Optimality

Skogestad & Grimholt (2012) compared SIMC PI against truly Pareto-optimal PI controllers across four benchmark processes. Performance metric J combines IAE for setpoint (IAEys) and input disturbance (IAEd), normalized by the IAE-optimal values at Ms = 1.59.

**SIMC PI (τc = θ) performance relative to Pareto-optimal (lower J is better):**

| Process | τ1/θ | SIMC Kc | SIMC τI | SIMC J | Pareto-optimal J | Ms |
|---|---|---|---|---|---|---|
| Pure time delay | 0 | 0 (pure I) | 0 | 1.35 | 1.00 | 1.59 |
| Small lag (τ1 = θ) | 1 | 0.5 | 1 | 1.03 | ~1.00 | 1.59 |
| Intermediate (τ1 = 8θ) | 8 | 4 | 8 | ~1.00 | 1.00 | 1.59 |
| Integrating | ∞ | 0.5 | 8θ | ~1.43 | 1.51 | 1.70 |

**Conclusion:** SIMC PI is within ~10% of Pareto-optimal for most processes. The only significant gap is the pure time delay process (40% above minimum J), which motivated the "Improved SIMC" rule.

### Improved SIMC Rule (for delay-dominant processes)

Replace τ1 by τ1 + θ/3 in the PI rules:
```
Kc = (1/K) · (τ1 + θ/3) / (τc + θ)
τI = min(τ1 + θ/3, 4(τc + θ))
```

This brings the pure time delay case nearly onto the Pareto-optimal curve (Kc ≈ 0.207, τI ≈ 0.333 vs. optimal Kc ≈ 0.20, τI ≈ 0.32 at Ms = 1.59).

The IMC PI rule (Rivera et al.) uses θ/2 for this correction — Skogestad found θ/3 gives faster settling and stays closer to the original rule structure.

## Relay Autotune (Åström-Hägglund)

A model-free method using relay feedback. Replace the controller with a relay (on/off with hysteresis). The plant oscillates at a limit cycle, from which Ku and Pu are extracted:

```
Ku ≈ 4d / (πa)
```

Where d is the relay amplitude and a is the oscillation amplitude. Pu is the oscillation period.

Then apply ZN-like rules (or preferably Tyreus-Luyben or SIMC rules) with the extracted Ku, Pu.

**Pros:** No plant model required. Safe — relay limits excitation amplitude. Can be automated.
**Cons:** Assumes the plant can tolerate oscillation. Quality of Ku/Pu extraction depends on noise.

**Motor control note:** Relay autotune is particularly useful for stepper/servo velocity loops where the plant model is hard to obtain analytically (friction, load-dependent dynamics).

## Tyreus-Luyben

A conservative variant of ZN closed-loop tuning, designed for stability-critical applications:

| Controller | Kc | τI | τD |
|---|---|---|---|
| PID | Ku/3.2 | Pu/0.45 | Pu/6.3 |

Compared to ZN: ~half the gain, ~4.4× the integral time. Sacrifices speed for significant stability margin improvement.

**Best for:** Systems where oscillation or overshoot is unacceptable (pressure control, safety-critical loops). Originally developed specifically for integrator/dead time processes.

## Direct Synthesis

Choose a desired closed-loop transfer function H(s), then solve for the required controller:

```
C(s) = H(s) / (G(s) · (1 − H(s)))
```

The result is model-dependent — the controller structure and order depend on the plant model. For FOPDT with first-order desired response, this yields PI or PID controllers similar to Lambda/IMC.

SIMC is a specific application of direct synthesis: the desired response is a first-order lag of time constant τc (following the unavoidable delay θ), applied to either a first- or second-order process model. SIMC's derivation is direct synthesis + the Taylor approximation e^(−θs) ≈ 1 − θs to convert the result into a realizable PID form.

**Pros:** Rigorous, principled.
**Cons:** Requires accurate plant model. Can produce non-standard controller structures.

## Performance/Robustness Tradeoff

The fundamental conflict in any tuning method:

- **Performance** (tight control): minimize IAE, fast disturbance rejection → favors small τc (or λ)
- **Robustness** (smooth control): minimize Ms, small input usage, noise insensitivity → favors large τc

Skogestad & Grimholt characterize this as a Pareto front in (Ms, J) space. Key observations:

1. **Operating region:** τc = θ places SIMC solidly in the negative-slope region of the Pareto front (genuine tradeoff). Reducing τc below θ gives diminishing performance returns with increasing robustness cost.
2. **Never go right of the knee:** If increasing τc improves *both* J and Ms simultaneously, you're overtuned — a smaller τc strictly dominates.
3. **Servo/regulator tradeoff for integrating processes:** An integrating plant cannot simultaneously achieve optimal setpoint *and* optimal disturbance response with a single PI parameter. SIMC c = 4 (in τI = min(τ1, 4(τc + θ))) gives a well-balanced compromise.
4. **ZN sits off the Pareto front:** ZN achieves neither good performance nor good robustness — the high gain buys some speed but at the cost of Ms > 2, which is worse than necessary.

### Minimum Controller Gain Constraint (Smooth Tuning)

For smooth control with acceptable disturbance rejection, the controller gain must satisfy:

```
Kc,min = |Δu0| / |Δymax|
```

Where Δu0 is the required input change to reject the disturbance and Δymax is the maximum allowed output deviation. This gives an upper bound on τc (τc,max), defining the practical operating window:

```
θ ≤ τc ≤ τc,max
```

## Practical Selection Guide for Motion Control

For stepper motor and servo applications:

1. **Start with SIMC** (τc = θ) — provides a rational first tuning based on minimal identification; near-optimal and robust (Ms ≈ 1.6)
2. **Use relay autotune** when plant model is unknown or load-dependent
3. **Avoid raw Ziegler-Nichols** — too aggressive for position/velocity loops, causes mechanical resonance excitation; robustness margins frequently below minimums
4. **Consider Tyreus-Luyben** for axes with compliance or backlash (overshoot causes position error accumulation)
5. **For delay-dominant motion loops** — use Improved SIMC (τ1 → τ1 + θ/3) to stay near Pareto-optimal
6. **Use τc > θ (smooth control)** when the loop must reject a known maximum disturbance but overshoot or large input transients are unacceptable — compute Kc,min first
7. **Add feedforward** after PID is reasonably tuned — see [[Feedforward-Compensators]]
8. **For high-performance requirements** — move beyond PID to sliding mode control or model-predictive control

## References

- Skogestad, S. & Grimholt, C. "The SIMC Method for Smooth PID Controller Tuning." in *PID Control in the Third Millennium* (Vilanova & Visioli, eds.), Springer, 2012. *(Chapter 5)*
- Skogestad, S. "Simple analytic rules for model reduction and PID controller tuning." *J. Process Control* 13, 291–309, 2003.
- Rivera, D.E., Morari, M., Skogestad, S. "Internal model control. 4. PID controller design." *Ind. Eng. Chem. Res.* 25(1), 252–265, 1986.
- Åström, K.J. & Hägglund, T. *Advanced PID Control.* ISA, 2006.
- O'Dwyer, A. *Handbook of PI and PID Controller Tuning Rules.* Imperial College Press, 3rd ed., 2009.
- Tyreus, B.D. & Luyben, W.L. "Tuning PI controllers for integrator/dead time processes." *Ind. Eng. Chem. Res.* 31, 2628–2631, 1992.
