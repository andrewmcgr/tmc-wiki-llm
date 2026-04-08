# SIMC PID Tuning Method

**Type:** knowledge
**Summary:** Skogestad & Grimholt's two-step SIMC method for analytically derived PI/PID controller tuning — FOPDT model approximation via the half rule, direct synthesis derivation, improved SIMC rule (τ₁ → τ₁ + θ/3), Pareto-optimality analysis, robustness margins, and c factor discussion.
**Tags:** #control-theory #pid #tuning #simc #motion
**Status:** draft
**Updated:** 2026-04-08
**Source:** Skogestad & Grimholt, "The SIMC Method for Smooth PID Controller Tuning," PID Control in the Third Millennium, Springer 2012; pidbook-chapter5.pdf; improved-simc-method-for-pi-controller-tuning.pdf
**Related:** [[PID-Tuning-Methods]], [[Feedforward-Compensators]]

---

## Overview

The SIMC method is a two-step model-based PID tuning procedure developed by Sigurd Skogestad (NTNU). It satisfies three core objectives: (1) well-motivated, analytically derived rules; (2) simple, memorizable formulas; (3) good performance on a wide range of processes. The method bridges overly aggressive heuristics (Ziegler-Nichols) and overly conservative IMC tuning.

**Two-step procedure:**
1. Obtain a first- or second-order plus delay model
2. Derive controller settings using direct synthesis

A first-order model yields PI settings; a second-order model yields PID settings. The **sole tuning parameter** is τc — the desired closed-loop time constant.

The series (cascade, "interacting") PID form is used throughout:

```
c(s) = Kc · [1 + 1/(τI·s) + τD·s]
```

---

## Step 1: Model Approximation

### Model Forms

**First-order plus delay (FOPDT):**
```
g(s) = k·e^(−θs) / (τ₁s + 1)
```

**Second-order plus delay (SOPDT):**
```
g(s) = k·e^(−θs) / [(τ₁s + 1)(τ₂s + 1)]
```

Parameters to identify: gain k, effective delay θ, dominant time constant τ₁, optional second time constant τ₂.

### Obtaining the Model

**From open-loop step response:** Apply a step input Δu and read k = Δy(∞)/Δu, τ₁ from the 63% rise time, and θ from the apparent dead time. For lag-dominant processes (τ₁ > 8θ), one need not wait for full settling — approximate as an integrating process with slope k′ = k/τ₁.

**From closed-loop setpoint response (P-controller):** Use a P-only controller with gain Kc0 chosen to give ~30% overshoot (D ≈ 0.3). Record: Kc0, time to first peak tp, setpoint change Δys, peak output Δyp, first undershoot Δyu. Estimate steady state as:

```
Δy∞ ≈ (Δyp + Δyu) / 2       [or wait for settling]
```

Compute overshoot D = (Δyp − Δy∞)/Δy∞ and offset B = |Δys − Δy∞|/Δy∞, then extract FOPDT parameters (Shamsuzzoha & Skogestad 2010 formulas).

### The Half Rule for Model Reduction

For a detailed transfer function with time constants τ₁₀ ≥ τ₂₀ ≥ τ₃₀ … and original delay θ₀:

> **Half rule:** The largest neglected denominator time constant is split evenly — half added to the effective delay θ, half added to the smallest retained time constant.

**To obtain a first-order model:**
```
θ = θ₀ + τ₂₀/2 + τ₃₀ + τ₄₀ + ...   (all smaller time constants go fully into θ)
τ₁ = τ₁₀ + τ₂₀/2                     (half of τ₂₀ added to dominant lag)
```

**To obtain a second-order model:**
```
θ = θ₀ + τ₃₀/2 + τ₄₀ + ...
τ₁ = τ₁₀
τ₂ = τ₂₀ + τ₃₀/2
```

Sampling period h (digital implementation) is added fully to θ.

**Example E1:** Process g(s) = e^(−0.1s) / [(s+1)(0.2s+1)] → half rule gives θ = 0.1 + 0.2/2 = 0.2, τ₁ = 1 + 0.2/2 = 1.1. Actually: k=1, θ=0.1, τ₁=1.1 (the original delay was 0 here; the 0.2 constant is split).

**Example E2:** Process g₀(s) = (−0.3s+1)(0.08s+1) / [(2s+1)(1s+1)(0.4s+1)(0.2s+1)(0.05s+1)³]

- First-order model: k=1, τ₁=2.5, θ=1.47 (τ₂=1 split: +0.5 to τ₁, +0.5 to θ; then 0.4, 0.2, 3×0.05 go fully into θ; positive numerator 0.08 subtracts from θ)
- Second-order model: k=1, τ₁=2, τ₂=1.2, θ=0.77

Use PID (second-order model) only when τ₂ > θ (≈), since derivative action then gives meaningful improvement.

### Positive Numerator Time Constants

For a process with a lead term (Ts+1):
- **T >> τc (large lead):** Cancel against adjacent lag τ₀: if T/τ₀ > threshold use T2/T3 rules (approximate (Ts+1)/s ≈ T for integrating denominators)
- **T ≈ τ₀ (close lag):** May cancel approximately
- **T < τc (small lead):** Subtract from effective delay θ (Rule T3)

---

## Step 2: SIMC Tuning Rules — Derivation

### Direct Synthesis Basis

From the closed-loop setpoint response:
```
y/ys = gc / (1 + gc)
```

Specifying a desired first-order response (after the unavoidable delay):
```
[y/ys]_desired = e^(−θs) / (τc·s + 1)
```

Solving for the controller c(s) with a second-order process model and applying a first-order Taylor approximation e^(−θs) ≈ 1 − θs yields a series-form PID controller.

This derivation produces the "ideal" integral time τI = τ₁. However, for lag-dominant processes (τ₁ >> θ), τI = τ₁ leads to slow disturbance rejection. Skogestad's modification: cap τI at 4(τc + θ) to just avoid slow oscillations while maintaining the analytically derived Kc.

---

## Summary of SIMC Rules

### PI Rules (First-Order Model)

For g(s) = k·e^(−θs) / (τ₁s + 1):

| Parameter | Formula |
|-----------|---------|
| **Kc** | (1/k) · τ₁ / (τc + θ) |
| **τI** | min(τ₁, 4·(τc + θ)) |

### PID Rules (Second-Order Model, Series Form)

For g(s) = k·e^(−θs) / [(τ₁s + 1)(τ₂s + 1)]:

| Parameter | Formula |
|-----------|---------|
| **Kc** | (1/k) · τ₁ / (τc + θ) |
| **τI** | min(τ₁, 4·(τc + θ)) |
| **τD** | τ₂ |

The derivative time cancels the second-largest process time constant. To convert series-form to ideal (parallel) form: use factor f = 1 + τD/τI, giving Kc′ = f·Kc, τI′ = f·τI, τD′ = τD/f.

### Special Cases (Table 5.1)

| Process | g(s) | Kc | τI | τD |
|---------|------|-----|-----|-----|
| Pure time delay | k·e^(−θs) | 0 | 0 (pure I: KI = 1/[k(τc+θ)]) | — |
| First-order | k·e^(−θs)/(τ₁s+1) | (1/k)·τ₁/(τc+θ) | min(τ₁, 4(τc+θ)) | — |
| Integrating | k′·e^(−θs)/s | (1/k′)·1/(τc+θ) | 4(τc+θ) | — |
| Integrating + lag | k′·e^(−θs)/[s(τ₂s+1)] | (1/k′)·1/(τc+θ) | 4(τc+θ) | τ₂ |
| Double integrating | k″·e^(−θs)/s² | (1/k″)·1/[4(τc+θ)²] | 4(τc+θ) | 4(τc+θ) |
| IPZ process | k′·e^(−θs)·(Ts+1)/[s(τ₂s+1)] | (1/k′)·(T/τ₂)/(τc+θ) | min(τ₂, 4(τc+θ)) | — |

For integrating processes, the integral time τI = 4(τc+θ) comes from avoiding slow oscillations: the product Kc·τI must exceed 4/k′.

---

## Choosing τc

τc is the **single tuning knob** trading performance against robustness:
- **Small τc** → fast response, tight control, higher Ms, larger input moves
- **Large τc** → slow response, smooth control, lower Ms, smaller input moves

From (5.27), must have τc > −θ for positive Kc.

### Tight Control (Recommended Default)

```
τc = θ
```

This gives Ms between 1.59 and 1.70 — robust by most standards (GM > 2.96, PM > 46.9°). It is "tightest possible subject to maintaining smooth control."

### Robustness Margins at τc = θ (Table 5.2)

| Metric | First-order process (τI = τ₁) | Integrating process (τI = 8θ) |
|--------|-------------------------------|-------------------------------|
| Kc | 0.5 · (τ₁/θ) / k | 0.5 / (k′·θ) |
| τI | τ₁ | 8θ |
| Gain margin (GM) | 3.14 | 2.96 |
| Phase margin (PM) | 61.4° | 46.9° |
| Allowed delay error Δθ/θ | 2.14 | 1.59 |
| Sensitivity peak Ms | 1.59 | 1.70 |
| Complementary sensitivity Mt | 1.00 | 1.30 |
| Phase crossover freq ω₁₈₀·θ | 1.57 | 1.49 |
| Gain crossover freq ωc·θ | 0.50 | 0.51 |

These margins are better than typical minimums (GM > 1.7, PM > 30°). The same margins apply to second-order processes if τD = τ₂ is chosen per (5.29).

### Smooth Control Bound

If tight control is unnecessary, a lower bound on Kc ensures acceptable disturbance rejection:

```
Kc,min = |Δu₀| / |Δymax|
```

where Δu₀ = required input change to reject the disturbance, Δymax = maximum allowed output deviation. The corresponding τc,max is found by substituting Kc,min into the Kc formula. Recommended range:

```
θ ≤ τc ≤ τc,max
```

For example, for liquid level control there is typically no need for tight control, so τc >> θ is appropriate.

### Input Usage

Initial input jump for a setpoint change Δys with PI control: u(0⁺) = Kc·Δys. For a first-order process with τ₁ = 8, θ = 1, τc = θ: initial jump = τ₁/(τc+θ) = 4 times the steady-state value. If this is unacceptable, increase τc. With PID, derivative action causes even larger initial jumps for setpoint changes — avoid differentiating the setpoint (use D-on-output only).

---

## Pareto-Optimality Analysis

To assess how good the SIMC rules are, performance is quantified as:

```
J = (1/2) · [IAEys / IAEys° + IAEd / IAEd°]
```

where IAEys° and IAEd° are reference IAE values from IAE-optimal PI controllers (at Ms = 1.59) for setpoint and load disturbance respectively. J = 1 means optimal; J > 1 means suboptimal relative to that reference.

### Table 5.3 — IAE-Optimal PI Controllers (Ms = 1.59)

| Process | Kc (setpt) | τI (setpt) | IAEys° | Kc (dist) | τI (dist) | IAEd° | Kc (comb) | τI (comb) | J |
|---------|------------|------------|--------|-----------|----------|-------|-----------|----------|---|
| e^(−s) (pure delay, τ₁=0) | 0.20 | 0.32 | 1.607 | 0.20 | 0.32 | 1.607 | 0.20 | 0.32 | 1.00 |
| e^(−s)/(s+1) (τ₁=1) | 0.55 | 1.15 | 2.083 | 0.50 | 1.04 | 2.036 | 0.54 | 1.10 | 1.00 |
| e^(−s)/(8s+1) (τ₁=8) | 4.0 | 8 | 2.169 | 3.33 | 3.65 | 1.135 | 3.47 | 4.0 | 1.23 |
| e^(−s)/s (integrating) | 0.50 | ∞ | 2.169 | 0.40 | 5.8 | 15.09 | 0.41 | 6.3 | 1.51 |

For the pure delay process, setpoint and disturbance responses are identical so J = 1 with a single controller. For integrating processes, there is a fundamental servo/regulator trade-off (J_opt = 1.51).

### Table 5.4 — SIMC PI vs. Improved SIMC PI (τc = θ)

| Process | Kc (SIMC) | τI (SIMC) | J (SIMC) | Ms (SIMC) | Kc (imp) | τI (imp) | J (imp) | Ms (imp) |
|---------|-----------|----------|---------|----------|---------|--------|--------|--------|
| e^(−s) (τ₁=0) | 0 | 0 (pure I) | 1.35 | 1.59 | 0.17 | 0.33 | 1.21 | 1.45 |
| e^(−s)/(s+1) (τ₁=1) | 0.5 | 1 | 1.03 | 1.59 | 0.67 | 1.33 | 1.09 | 1.69 |
| e^(−s)/(8s+1) (τ₁=8) | 4 | 8 | 1.00 | 1.59 | 4.17 | 8 | 1.34 | 1.62 |
| e^(−s)/s (integrating) | 0.5 | 8 | 1.43 | 1.70 | 0.5 | 8 | 1.43 | 1.70 |

Key findings:
- For τ₁/θ = 1 and τ₁/θ = 8, SIMC is within ~3% of optimal (J ≈ 1.00–1.03)
- For the pure delay process (τ₁/θ = 0), SIMC is ~35% above optimal (J = 1.35)
- The improved SIMC rule reduces J for the pure delay case from 1.35 → 1.21
- Varying τc traces a near-Pareto-optimal curve for all processes except pure delay

---

## Improved SIMC Rule

### Problem with Pure Delay Processes

For a pure time delay g(s) = k·e^(−θs) (τ₁ = 0), the original SIMC rule gives Kc = 0 and τI = 0, degenerating to a pure integrating controller (KI = Kc/τI = 1/[k(τc+θ)]). The response is smooth but sluggish — about 35% worse than optimal in terms of J.

The IAE-optimal PI controller for a pure time delay process has τI ≈ θ/3 across a wide range of Ms values (1.4 to 1.7). This motivates the fix.

### The Improved SIMC Rule

Replace τ₁ by (τ₁ + θ/3) in the PI tuning formulas:

**Improved SIMC PI rules:**

| Parameter | Formula |
|-----------|---------|
| **Kc** | (1/k) · (τ₁ + θ/3) / (τc + θ) |
| **τI** | min(τ₁ + θ/3, 4·(τc + θ)) |

For a pure delay process this gives τI = θ/3 — close to the Pareto-optimal value. For the improved SIMC, the recommended τc shifts to τc ≈ 0.61θ for the pure delay case (giving Kc = 0.207, τI = 0.333, KI = 0.62).

**Impact by process type:**
- **Pure delay (τ₁ = 0):** Large improvement — J drops 1.35 → 1.21, near-optimal for small Ms
- **Small lag (τ₁ = θ):** Slight improvement for aggressive tuning, slight degradation for conservative tuning
- **Large lag (τ₁ = 8θ or integrating):** Negligible difference — the θ/3 correction is small relative to τ₁

**Note on IMC comparison:** Rivera et al. [8] proposed replacing τ₁ by τ₁ + θ/2 for their "improved PI" rule, but θ/2 gives a τI that is too large, resulting in slow settling. The SIMC value θ/3 gives faster settling and remains closer to the original SIMC structure.

---

## The c Factor for Integral Time

The original SIMC integral time cap uses c = 4:
```
τI = min(τ₁, c·(τc + θ))    with c = 4
```

This c parameter controls the servo/regulator trade-off. A larger c improves setpoint performance; a smaller c (e.g., c = 2) improves load disturbance rejection.

**Why c = 4 is recommended:**
1. It is close to the Pareto-optimal PI controller for the combined J cost metric
2. Reducing c to ~2.5 degrades robustness at τc = θ (higher Ms), requiring a different τc recommendation for integrating processes (adding complexity)
3. The near-optimal value from Table 5.3 for integrating processes is c ≈ 6.3/2 ≈ 3.15 for disturbance and c = ∞ for setpoint — c = 4 is a reasonable middle ground

The value c ≈ 2.6 is sometimes cited as near-Pareto-optimal for integrating processes when disturbance rejection is prioritized, but this advantage is modest and comes at the cost of reduced robustness.

---

## Special Topics

### Retuning Integrating Processes (Oscillations)

A common problem: integrating processes tuned with too-small Kc·τI product exhibit slow oscillations. Operators erroneously reduce Kc, making it worse. Rule:

> If slow oscillations with period P₀ occur (P₀ > 3·τI0), and the process is close to integrating, the product Kc·τI must be increased by factor **F ≈ 0.1·(P₀/τI0)²**.

This avoids requiring a model — just observe the oscillation period and apply the factor.

### Measurement Noise

The SIMC setting τc = θ is *less* sensitive to noise than Ziegler-Nichols (due to lower Ms). Practical recommendations:
1. Add measurement filter 1/(τFs+1) with τF up to ~0.5θ without major performance impact
2. If derivative action is used and noise is an issue, revert to first-order model and PI control
3. If noise still problematic, increase τc

### Controllability Insight

The effective delay θ (computed via half rule) is the primary limit on achievable control performance. PI control performance is fundamentally limited by (half of) τ₂ — the second-largest time constant. PID control performance is limited by (half of) τ₃.

---

## Comparison with Ziegler-Nichols

ZN settings are known to give aggressive tuning (Ms ≈ 2 or higher) with good disturbance response for integrating processes but poor performance for delay-dominant processes. SIMC with τc = θ typically gives:

- Ms ≈ 1.59–1.70 (vs ZN Ms often >2)
- Better robustness margins
- Better performance for delay-dominant processes
- Comparable disturbance rejection for integrating processes (with τI = 4(τc+θ) modification)

The IMC rules (Rivera et al.) give good setpoint response but poor disturbance rejection for integrating processes. SIMC avoids this by capping τI at 4(τc+θ).

---

## Worked Example (E2)

Process: g₀(s) = (−0.3s+1)(0.08s+1) / [(2s+1)(1s+1)(0.4s+1)(0.2s+1)(0.05s+1)³]

**Half rule → first-order model:** k=1, τ₁=2.5, θ=1.47

SIMC PI with τc = θ = 1.47:
```
Kc = (1/1) · 2.5 / (1.47 + 1.47) = 0.85
τI = min(2.5, 4·2.94) = 2.5
```

**Half rule → second-order model:** k=1, τ₁=2, τ₂=1.2, θ=0.77

SIMC PID with τc = θ = 0.77:
```
Kc = (1/1) · 2 / (0.77 + 0.77) = 1.30
τI = min(2, 4·1.54) = 2
τD = 1.2  (series form)
```

Convert to parallel form (f = 1 + τD/τI = 1 + 1.2/2 = 1.60):
```
Kc' = 1.30 · 1.60 = 2.08
τI' = 2 · 1.60 = 3.20
τD' = 1.2 / 1.60 = 0.75
```

The PID controller shows notably better disturbance rejection than PI for this process (τ₂ = 1.2 > θ = 0.77 satisfies the condition for beneficial derivative action).

---

## Relevance to Motion Control

SIMC is directly applicable to stepper/servo motor tuning:

- **Velocity loops** approximate FOPDT well — electrical time constant τ₁, communication/computation delay θ. Use PI with τc = θ as starting point.
- **Position loops** are integrating processes — use the integrating-process rules: Kc = (1/k′)/(τc+θ), τI = 4(τc+θ).
- **Cascaded loops** (position → velocity → current) — tune inner loops tight (small τc), outer loops looser.
- **The τc knob** maps directly to the smoothness/responsiveness trade-off in motion applications: increase τc to reduce vibration/ringing at the cost of slower settling.
- **Improved SIMC rule** applies when the velocity loop has near-zero lag (motor + driver with very fast electrical response) — the θ/3 correction prevents the degenerate pure-I controller.

---

## References

- Skogestad, S. "Simple analytic rules for model reduction and PID controller tuning." *Journal of Process Control* 13, 291–309, 2003.
- Skogestad, S. & Grimholt, C. "The SIMC Method for Smooth PID Controller Tuning." In *PID Control in the Third Millennium*, Springer, 2012 (Chapter 5).
- Grimholt, C. & Skogestad, S. "The improved SIMC method for PI controller tuning." *IFAC PID'12*, Brescia, Italy, March 2012.
- Rivera, D.E., Morari, M., Skogestad, S. "Internal model control. 4. PID controller design." *Ind. Eng. Chem. Res.* 25(1), 252–265, 1986.
- Shamsuzzoha, M. & Skogestad, S. "The setpoint overshoot method: a simple and fast method for closed-loop PID tuning." *J. Process Control* 20, 1220–1234, 2010.
