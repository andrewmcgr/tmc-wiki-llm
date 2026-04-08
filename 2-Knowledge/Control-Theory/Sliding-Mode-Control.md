# Sliding Mode Control

**Type:** knowledge
**Summary:** Sliding mode control (SMC) theory and application to permanent magnet linear synchronous motors (PMLSM) — covers PMLSM state-space model, chattering mitigation, all major SMC variants (boundary layer, reaching law, disturbance observer, TSMC, NTSMC, FNTSMC, integral, super-twisting, adaptive, intelligent), reaching law formulations, disturbance observer designs, and neural network/fuzzy combinations. Based on Yu et al. 2023 review paper.
**Tags:** #control-theory #smc #motion #advanced #pmlsm
**Status:** draft
**Updated:** 2026-04-08
**Source:** Yu et al., "Sliding Mode Control for PMLSM Position Control — A Review," Actuators 12(1), 2023; sliding-mode-control-for-pmlsm-position-control-a-review.pdf
**Related:** [[PID-Tuning-Methods]], [[SIMC-PID-Tuning]], [[TMC4671-LA]]

---

## Overview

Sliding mode control (SMC) is a nonlinear, variable-structure control technique that drives system states onto a predefined surface in state space (the **sliding surface**) and forces them to remain there regardless of disturbances or model uncertainty. Once on the sliding surface, system dynamics are governed entirely by the surface design — independent of matched disturbances. This gives SMC exceptional robustness.

SMC originated in the late 1950s; widespread development began after Utkin's 1977 survey. The tradeoff is **chattering** — high-frequency oscillation caused by the discontinuous switching action. Modern SMC variants address this through boundary layers, higher-order algorithms, observer-based estimation, or adaptive/intelligent gain tuning.

SMC has been used extensively in PMLSM position control because PMLSMs are multivariable, strongly coupled nonlinear systems affected by friction, load change, and force ripple that defeat linear controllers.

---

## PMLSM Dynamic Model

### d-q Frame Equations

Three reference frames are used: a-b-c three-phase stationary, α-β stationary, and d-q rotating. The motor equations in d-q coordinates are:

**Stator current (voltage) equations:**
```
u_d = R_s·i_d + L_d·(di_d/dt) − ω·L_q·i_q
u_q = R_s·i_q + L_q·(di_q/dt) + ω·L_d·i_d + ω·ψ_f
```

**Electromagnetic thrust** (surface-mounted PMLSM, L_d = L_q = L_s):
```
F_e = (3/2)·(π/τ)·ψ_f·i_q  =  k_f · i_q
```

**Mechanical motion equation:**
```
F_e = m·(dv/dt) + B·v + F_W
```

Where:
- R_s, L_s — winding resistance and inductance
- ψ_f — permanent magnet flux linkage
- k_f — thrust force constant  
- m — mover mass
- B — viscous friction coefficient
- v — linear velocity
- F_W — lumped uncertainties (nonlinear friction, load disturbance, force ripple)
- τ — pole pitch, n_p — number of pole pairs

### State-Space Form for SMC Design

Define states x₁ = x (position), x₂ = v (velocity), control u = i_q, output y = x:

```
ẋ₁ = x₂
ẋ₂ = k₁·u + k₂·x₂ + D(t)
y   = x₁
```

Where:
```
k₁ = k_f / m       (force constant / mass)
k₂ = −B / m        (viscous friction term)
D(t) = −F_W / m    (lumped disturbance, |D(t)| ≤ D_max)
```

The q-axis current i_q is the control input. SMC determines the desired i_q reference; the inner current loop (PI or hysteresis) tracks it.

---

## SMC Theory

### Fundamental Structure

For a nonlinear system ẋ = f(x,t) + g(x,t)·u + d(x,t), define a switching function **s(x) ∈ Rᵐ** such that s(x) = 0 is the sliding surface. The SMC law takes the form:

```
u_i = u_i⁺(x)  if  s_i(x) > 0
u_i = u_i⁻(x)  if  s_i(x) < 0
```

System motion consists of two phases:
1. **Reaching phase** — states driven to s = 0 from arbitrary initial conditions in finite time
2. **Sliding phase** — states constrained to s = 0; dynamics governed by sliding surface design

### Sliding Surface Design

**Linear surface** (simplest, asymptotic convergence):
```
s(x) = C·x(t)    where C = [c₁, c₂, …, c_{n-1}, 1]ᵀ
```
Choose C so that the characteristic polynomial pⁿ⁻¹ + c_{n-1}·pⁿ⁻² + … + c₁ is Hurwitz.

For tracking control with error e = x_d − x:
```
s = (d/dt + λ)^(n-1) · e    (n = system order)
```
For second-order (n=2): s = ė + λ·e, λ > 0.

### SMC Law Design — Equivalent Control Method

```
u = u_eq + u_sw
```

- **u_eq** (continuous): keeps system on surface when s = 0. Derived from ṡ = 0 ignoring disturbances:

```
u_eq = −[∂s/∂x · g(x)]⁻¹ · [∂s/∂x · f(x)]
```

- **u_sw** (discontinuous): enforces reachability condition ṡ·s < 0:

```
u_sw = −k · sgn(s) / [∂s/∂x · g(x)]    where k = D_max + η, η > 0
```

Lyapunov function V = ½s² confirms stability: V̇ = s·ṡ < 0 outside s = 0.

---

## Reaching Law Approach

Gao (1993) proposed specifying the switching function dynamics directly as a differential equation. General form:

```
ṡ = −L·sgn(s) − K·f(s)
```

Where L = diag[l₁,…,lₘ], lᵢ > 0; K = diag[k₁,…,kₘ], kᵢ > 0; sᵢ·fᵢ(sᵢ) > 0, fᵢ(0) = 0.

### Special Cases

**1. Constant rate reaching law:**
```
ṡ = −K · sgn(s)
```
Reaches surface in t = |s(0)|/K. Chattering proportional to K.

**2. Constant plus proportional rate (exponential) reaching law:**
```
ṡ = −K · sgn(s) − L·s
```
The −L·s term accelerates convergence when s is large, reduces chattering rate near surface. Most common in practice.

**3. Power rate reaching law:**
```
ṡ = −K · |s|^α · sgn(s)    (0 < α < 1)
```
Slows as s → 0, further reducing chattering. Used in variable-rate approaches for PMLSM.

**Variable-rate reaching law with sigmoid/power function** (for PMLSM observers): a term (1 − e^(−a|e₁|)) ties reaching speed to state error e₁, suppressing chattering while maintaining rapidity.

---

## Chattering Problem and Mitigation

Chattering arises from the discontinuous sgn(·) function. In physical systems: actuator wear, audible noise, excitation of unmodeled dynamics.

### Overview of Approaches

| Approach | Mechanism | Tradeoff |
|---|---|---|
| **Boundary layer** | Replace sgn(s) with sat(s/Δ) | Continuous control; steady-state error ∝ Δ |
| **Reaching law** | Shape ṡ dynamics to slow near surface | Reduced robustness |
| **Disturbance observer** | Estimate D(t), reduce required K | Depends on observer accuracy |
| **Terminal SMC** | Nonlinear surface, finite-time convergence | Singularity risk; more complex |
| **Super-twisting** | 2nd-order SMC with continuous u | More complex; needs careful gain tuning |
| **Adaptive SMC** | Tune K online | Possible overestimation |
| **Intelligent SMC** | NN/fuzzy approximate disturbance | Requires training data, complex |

---

## Conventional SMC Methods

### Boundary Layer Approach

Replace the sign function with a saturation function inside a layer of width Δ around s = 0:

```
sat(s) = sgn(s)       if |s| ≥ Δ
sat(s) = s/Δ          if |s| < Δ
```

The boundary layer delivers linear feedback inside (proportional to s/Δ) and switching outside. This ensures continuity at the cost of asymptotic (not finite-time) convergence to a residual set of size ∝ Δ.

**Double boundary layer** (Cupertino et al., 2009): uses two layers with widths Φ_h (outer) and Φ_l (inner), switching behavior between layers, to simultaneously suppress chattering and address static friction:

```
sat(s_x) = sgn(s_x)              if |s_x| ≥ Φ_h
sat(s_x) = s_x/Φ_l + β·sgn(s_x) if Φ_h > |s_x| > Φ_l
sat(s_x) = (1+β)·s_x/Φ_l        if |s_x| ≤ Φ_l
```

**Complementary SMC with approach-angle saturation** (Jin & Zhao, 2019): the boundary layer dynamically shrinks as the state trajectory approaches the surface, preserving asymptotic stability unlike fixed-Δ saturation.

### Reaching Law Approach in Practice

PMLSM implementations use the exponential reaching law ṡ = −L·sgn(s) − K·s with an added adaptive disturbance observer (ADO) to reject external disturbances and parameter uncertainties. A variable-rate law (Lu et al.) combining exponential and power terms achieved excellent dynamic characteristics. A discrete-time load thrust observer (Wang et al., 2022) with an improved reaching law effectively suppressed chattering for linear motor load thrust mutations.

### Disturbance Observer-Based SMC

Architecture: the disturbance observer (outer loop) estimates lumped disturbances from measurable variables, feeds the estimate D̂(t) to the SMC inner loop. SMC switching gain K need only exceed the estimation error (not D_max), substantially reducing chattering.

**Reduced-order disturbance observer** (Aschemann et al., 2018): state equation extended with integrator disturbance model ḞW = 0; estimated force fed to integral SMC. Steady-state accuracy and chattering elimination confirmed by simulation.

**Finite-time disturbance observer (FTDO)** + FNTSMC (Gao et al., 2018): FTDO estimates time-varying disturbances with Lyapunov-verified finite-time convergence; FNTSMC position controller guarantees error reaches zero in finite time. Better tracking, faster dynamics, stronger robustness than PID and linear SMC in simulation.

**DFOB-MI** (disturbance force observer with mass identification): feeds disturbance compensation directly to current regulator rather than position SMC loop; mass identified online using discrete MRAI method. Implemented on DSP TMS320F28335.

**NDOB-backstepping SMC** (Song et al., 2019): nonlinear disturbance observer (improved sliding-mode observer) combined with backstepping SMC and command filter for good tracking.

**Recursive SMC with adaptive disturbance observer (RSM-ADO)** (Shao et al., 2021): ADO uses sliding-mode structure with nested adaptation — no upper bound on disturbance or derivative required. Peak error ~5 μm tracking 1000 μm triangular wave at 0.2 ms sampling. Implemented on DSPACE DS1103.

**Double disturbance observer** (Zhang et al., 2022): matched + mismatched disturbance observers combined with high-order fast NTSMC. Control law: u_q = u_q0 + u_q1 + u_q2 (u_q2 = disturbance compensation). Implemented on TMS320F2812 DSP.

---

## Advanced SMC — Terminal SMC (TSMC)

Terminal SMC introduces a nonlinear term to guarantee **finite-time convergence** (vs. asymptotic for linear surfaces).

### Basic Terminal Sliding Surface

```
s(t) = ẋ + β·x^(q/p) = 0    (β > 0; p, q positive odd integers; 1 < p/q < 2)
```

Convergence time from x(0) = x₀:
```
t_s = p / (β·(p−q)) · |x₀|^((p−q)/p)
```

### Fast Terminal Sliding Surface

Combines linear and nonlinear terms so far-from-origin uses terminal attractor, close-to-origin uses linear (faster):

```
s = ẋ + α·x + β·x^(q/p) = 0    (α, β > 0; p > q, positive odd)
```

### Nonsingular Terminal SMC (NTSMC)

Avoids the singularity (ẋ → ∞ when x → 0) of basic TSMC by rewriting the surface:

```
s = x + (1/β)·ẋ^(p/q)    (sig^γ(x) = sgn(x)·|x|^γ, α,β > 0, 0 < γ < 1)
```

Or equivalently:
```
s = ẋ + β·sig^γ(x)
```

Finite convergence time maintained without singularity.

### Fast NTSMC (FNTSMC)

Combines fast convergence with nonsingularity. Sliding surface:

```
s = e + (1/α)·ė^(p/q)    where e = y − y_r
```

Control law: u = u_eq + u_sw, where u_sw = −m₀[k₁·s + k₂·sig^p(s)]. Verified on dSPACE-DS1103; outperforms NTSMC and H∞ with and without load changes.

### Integral Terminal SMC

Sliding surface includes integral term for eliminating steady-state error and removing reaching phase:

```
s = k₁·e + k₂·∫e dt + ė^(2α₁/(1+α₁))    (k₁, k₂ > 0, α₁ > 0)
```

Sign function replaced with novel saturation function to reduce chattering. Effective for finite-time tracking in presence of disturbances.

### Discrete-Time TSMC

For direct DSP implementation using Euler discretization. Discrete terminal surface:

```
s(k) = e₁(k) + c₁·e₁(k−1) + c₂·|e₂(k)|^γ·sgn(e₂(k))
```

Where e₁(k) = x_r(k) − x₁(k), e₂(k) = [e₁(k+1) − e₁(k)]/h, 0 < h·c₁ < 1, c₂ > 0, 0 < γ < 1.

Tracking error converges to O(h³). Delayed estimation (TDE) compensates disturbance. Validated on DSP; smaller steady-state error and better robustness vs. PID and discrete linear SMC.

**Discrete fractional-order terminal SMC**: surface includes fractional-order term D^α·e for global memory and improved transient performance; implemented on real linear motor with higher precision and robustness than chattering-free DTSMC.

---

## Advanced SMC — Super-Twisting Algorithm (STA)

Super-twisting is a 2nd-order sliding mode algorithm. It produces a **continuous control signal** (only the internal state derivative is discontinuous), eliminating first-order chattering while retaining simplicity and robustness.

### Algorithm

```
u   = −α·|s|^(1/2)·sgn(s) + v
v̇  = −β·sgn(s)
```

Control u is continuous (integral of sgn term). Conditions for finite-time convergence: α² ≥ 4β·D_max/(β − D_max) approximately (depends on disturbance bound D_max and its derivative).

### Variants Applied to PMLSM

**STA with defined boundary layer** (Tan et al., 2021): standard linear sliding surface, STA control law within defined boundary. Higher steady-state accuracy and less chattering vs. conventional SMC. Int. J. Mech. Sci.

**Super-twisting NTSMC (STNTSMC)**: STA switching law combined with nonsingular terminal surface:
```
u_sw = −α·|s|^(1/2)·sgn(s) + v,   v̇ = −β·sgn(s)
```
Ensures s = ṡ = 0 in finite time. Combined with high-order sliding-mode observer (HOSTO) for uncertainty estimation → OSTNTSMC (Fu et al., IEEE TPE 2021).

**NFTSM + high-order super-twisting observer** (Xu et al., IEEE/ASME Trans. Mech. 2022): observer estimates lumped disturbances from incomplete state information; NFTSM controller guarantees tracking convergence. Faster response and higher precision vs. conventional NFTSM and NTSM.

**Self-adaptive STA for speed tracking** (Li et al., 2019): gains k_p and k_i tuned by adaptive algorithm:
```
u = k_p·|s|^(1/2)·sgn(s) + k_i·∫sgn(s)dt
```
Implemented on linear motor platform. Tracking error < 0.04 mm/s (Wang et al., 2018).

---

## Advanced SMC — Adaptive SMC

Adaptive SMC tunes switching gains online to track disturbance amplitude, avoiding overestimation. The adaptive law adjusts K̂ in real time:

```
u_sw = −K̂(t)·sgn(s)
K̂̇ = γ·|s|    (γ > 0, adaptation rate)
```

K̂ grows until the reachability condition is met, then stabilizes.

### Variants Applied to PMLSM

**Adaptive fractional-order terminal SMC (AFOTSM)** (Sun & Ma, 2018): FO terminal surface gives higher tracking precision than integer terminal surfaces. Adaptive term in switching control weakens uncertainty effects. Errors < 0.005 mm without load, < 0.01 mm with 30% load. Outperformed FOTSM, FNTSM, PID, and OSMC. IEEE/ASME Trans. Mech.

**Adaptive recursive terminal SMC (ARTSM)** (Shao, 2020): recursive structure with fast NTSM function and recursive integral terminal sliding mode. System starts on sliding surface (no reaching phase). Tracking error < 1.7% of reference amplitude. IEEE Trans. Ind. Electron.

**Barrier function adaptive SMC (BFASM)** (Shao et al., 2021): barrier function in adaptive law ensures gain remains bounded (prevents overestimation). No disturbance upper bound needed. Tracking errors within 40 μm under payload-varying disturbances. IEEE Trans. Ind. Inf.

**Discrete adaptive SMC** (Chen & Lu, 2014): real-time gain tuning for linear PMISM. Fast response and strong robustness under uncertainties and noise. IEEE Trans. Ind. Inf.

**Adaptive NTSMC with time delay estimation (TDE)** (Fu et al., 2022): TDE estimates unmodeled dynamics and bounded control input gain; adaptive algorithm tunes robust control gain. Higher tracking accuracy, verified on real PMLSM with DSP. Int. J. Control Autom.

---

## Advanced SMC — Intelligent SMC

Intelligent SMC replaces or augments the switching gain with a neural network or fuzzy system that approximates disturbances/uncertainties online. SMC guarantees stability; the intelligent component reduces conservatism and chattering.

### Neural Network Variants

**Intelligent complementary SMC + RBFN** (Lin et al., 2010): RBFN trained online by adaptive learning approximates lumped uncertainties. FPGA-based implementation for PMLSM. Effective for different strokes, test signals, load conditions. IEEE Trans. Power Syst.

**Adaptive RBF-NN NFTSMC** (Zhao & Fu, 2020): RBF neural network approximates unknown PMLSM dynamics; adaptive law estimates uncertainty upper bound. Control law:
```
u = u_eq_NN + u_sw_adaptive
```
u_eq estimated by RBF, u_sw gain tuned by adaptive law. Higher accuracy, faster response, less chattering vs. conventional NFTSMC. IEEE Access.

**SOSMC + recurrent RBFNN (RRBFNN)** (Zhao et al., 2020): second-order SMC eliminates chattering; RRBFNN uncertainty observer estimates unknowns. Control law:
```
u = u_eq + u_NN + u_rc_compensator
```
Best chattering reduction and tracking accuracy vs. SOSMC and SOSMC+RBFNN. IET Electr. Power Appl.

**FNTSMC + extreme learning machine (ELM)** (Zhang et al., 2020): ELM adaptively estimates equivalent control term online; sat(s) replaces sgn(s). Control:
```
u = u_eq_ELM + u_sw_sat + u_re_reaching
```
Neural Comput. Appl.

**IBTSMC + RBF** (Fu et al., 2021): backstepping terminal SMC with tanh(·) replacing sgn(·); RBF compensates for uncertainty. Avoids high switching gain. IEICE Electron. Express.

**Multi-kernel NN SMC (MNN)** (Wang et al., 2021): MNN approximates lumped disturbances; dynamic boundary layer (saturation function) reduces chattering. Better than ASMC-RBF under load variation. IEEE Access.

### Fuzzy Logic Variants

**Adaptive fuzzy fractional-order SMC + PFNN** (Chen et al., 2019): probabilistic fuzzy neural network (PFNN) estimates lumped uncertainties online; adaptive fuzzy reaching regulator (AFRR) compensates for PFNN observation deviations while smoothing hitting control. Lyapunov-derived adaptive tuning. Best precision and robustness in comparison vs. PID, FO-SMC sign, FO-SMC sat, adaptive self-tuning PID fuzzy SMC. IEEE/ASME Trans. Mech.

---

## Comparative Summary

From Yu et al. Table 2:

| Approach | Advantages | Disadvantages |
|---|---|---|
| **Boundary layer** | Simple; reduced chattering | Large control gain; steady-state error ∝ Δ; no robustness guarantee inside layer |
| **Reaching law** | Convergence rate adapts to distance from surface; reduced chattering | Reduced robustness vs. conventional SMC |
| **Disturbance observer SMC** | Reduced chattering; disturbance rejection without worsening chattering | Control performance depends on observer accuracy |
| **Terminal SMC** | Finite-time convergence; stronger robustness; less steady-state error | Complex computation; singularity in basic TSMC (not in NTSM/NFTSM) |
| **Super-twisting SMC** | Continuous control signal; finite-time convergence; retains first-order simplicity; less steady-state error | Complex computation and gain tuning |
| **Adaptive SMC** | Gains tuned online; better disturbance rejection; reduced chattering | Possible overestimation (except barrier function variants) |
| **Intelligent SMC** | Universal disturbance approximation; strongest robustness; adaptive performance | Most complex; requires training data, fuzzy rules, or NN architecture |

---

## Design Procedure (Practical)

1. **Identify plant parameters:** m (moving mass), B (viscous friction), k_f (force/torque constant); measure or estimate D_max from load analysis and friction characterization
2. **Choose sliding surface type:** Start with s = ė + λ·e (linear); upgrade to FNTSMC if finite-time convergence is needed
3. **Set λ:** Based on desired error decay rate τ_c ≈ 1/λ
4. **Design control law:** Equivalent control from ṡ = 0 assumption; switching control u_sw = −K·sgn(s)/k₁
5. **Set switching gain:** K > D_max with margin η. Start conservatively, reduce if chattering is excessive
6. **Mitigate chattering:** First try boundary layer (sat) or STA. If disturbance bound is large/uncertain, add disturbance observer
7. **Consider advanced variants:** FNTSMC for fast convergence; STA for continuous u; Adaptive SMC if disturbance amplitude varies widely; NN/fuzzy SMC only if simpler methods are insufficient
8. **Discretize carefully:** STA discretizes more cleanly than basic SMC (continuous u). Euler discretization introduces O(h) error; use h ≤ 0.2 ms for motor control
9. **Simulate first:** SMC is sensitive to discrete-time effects and sensor noise. Validate with realistic disturbance models
10. **Validate on hardware:** Key metrics are position tracking error (μm), disturbance rejection time (ms), control signal chattering amplitude

---

## Relevance to Stepper Motor Control

While the Yu et al. review focuses on PMLSM (linear motors), the theory transfers directly to rotary stepper motors operated under FOC:

- **Stepper motors as PMSMs:** Modern FOC-driven steppers (via [[TMC4671-LA]] or firmware FOC) use the same dq-frame current control as PMLSM — the state-space model (ẍ = k₁·i_q + k₂·v + D) applies directly to the position/velocity loop
- **Discrete-time implementation:** Firmware SMC requires discretization. The super-twisting algorithm (continuous u) discretizes more cleanly than basic SMC; discrete terminal SMC has O(h³) error convergence
- **Load disturbance rejection:** SMC's invariance property on the sliding surface makes it attractive for CNC/3D printer axes subject to cutting forces or varying filament back-pressure — linear PID cannot match this
- **Chattering vs. microstepping:** The chattering frequency must be well above mechanical bandwidth but below the microstepping/FOC update rate to avoid audible interaction. STA or DOB+SMC preferred over plain switching SMC for audio-sensitive applications
- **Friction and force ripple:** The lumped uncertainty D(t) in the PMLSM model maps directly to stepper cogging torque and static friction — the same DOB designs suppress both
- **FNTSMC for point-to-point moves:** Finite-time convergence guarantees the axis reaches target in bounded time, useful for sequenced multi-axis motion

---

## Key References

Selected from Yu et al. 2023 citation list (most significant):

- **Yu et al. (2023):** "Sliding-Mode Control for PMLSM Position Control — A Review." *Actuators* 12(1), 31. https://doi.org/10.3390/act12010031 — primary source for this page
- **Utkin (1977):** "Variable Structure Systems with Sliding Modes." *IEEE Trans. Autom. Control* 22, 212–222 — original SMC survey
- **Young, Utkin & Ozguner (1999):** "A control engineer's guide to sliding mode control." *IEEE Trans. Control Syst. Technol.* 7, 328–342 — engineering perspective, chattering analysis
- **Gao & Hung (1993):** "Variable Structure Control of Nonlinear Systems: A New Approach." *IEEE Trans. Ind. Electron.* 40, 45–55 — reaching law framework
- **Slotine & Sastry (1983):** "Tracking control of nonlinear systems using sliding surfaces." *Int. J. Control* 38, 465–492 — boundary layer approach
- **Feng, Yu & Man (2002):** "Non-singular terminal sliding mode control of rigid manipulators." *Automatica* 38, 2159–2167 — NTSMC foundation
- **Yu & Man (2002):** "Fast terminal sliding-mode control design for nonlinear dynamical systems." *IEEE Trans. Circuits Syst.* 49, 261–264 — fast TSM
- **Shtessel, Taleb & Plestan (2012):** "A novel adaptive-gain super twisting sliding mode controller." *Automatica* 48, 759–769 — adaptive STA
- **Du, Chen et al. (2018):** "Discrete-Time Fast Terminal Sliding Mode Control for Permanent Magnet Linear Motor." *IEEE Trans. Ind. Electron.* 65, 9916–9927 — discrete TSMC reference
- **Fu, Zhao & Zhu (2021):** "A Novel Robust Super-Twisting Nonsingular Terminal SMC for PMLSM." *IEEE Trans. Power Electron.* 37, 2936–2945 — OSTNTSMC
- **Edwards & Spurgeon (1998):** *Sliding Mode Control: Theory and Applications.* CRC Press — standard textbook
