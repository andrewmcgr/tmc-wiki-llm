# Wiki Log

Append-only log of significant changes to the wiki. Each entry captures what changed and why.

---

## [2026-04-07] init | Wiki bootstrapped from llm-context-base template
## [2026-04-08] config | Profile configured — stepper motor driver IC research wiki for amcgregor
## [2026-04-08] structure | Added 2-Knowledge/Drivers/, Control-Theory/, Software/ — domain subdirectories
## [2026-04-08] config | Enabled source preservation in _sources/
## [2026-04-08] ingest | TMC2209 datasheet → 2-Knowledge/Drivers/TMC2209.md (via web research, PDF not extractable)
## [2026-04-08] ingest | TMC2240 datasheet → 2-Knowledge/Drivers/TMC2240.md (via web research, PDF not extractable)
## [2026-04-08] ingest | TMC5160A datasheet → 2-Knowledge/Drivers/TMC5160A.md (via web research, PDF not extractable)
## [2026-04-08] ingest | TMC4671-LA datasheet → 2-Knowledge/Drivers/TMC4671-LA.md (via web research, PDF not extractable)
## [2026-04-08] ingest | SIMC PID tuning → 2-Knowledge/Control-Theory/SIMC-PID-Tuning.md (from Skogestad JPC 2003 + web research)
## [2026-04-08] ingest | PID tuning methods overview → 2-Knowledge/Control-Theory/PID-Tuning-Methods.md (from pidbook-chapter5 + web research)
## [2026-04-08] ingest | Sliding mode control → 2-Knowledge/Control-Theory/Sliding-Mode-Control.md (from Yu et al. 2023 MDPI review)
## [2026-04-08] ingest | Feedforward compensators → 2-Knowledge/Control-Theory/Feedforward-Compensators.md (partial — source paywalled, general knowledge used)
## [2026-04-08] reingest | TMC2209 rewrite from Docling PDF extraction → 2-Knowledge/Drivers/TMC2209.md (Rev 1.09 datasheet; full register map, pin config, current sensing, motion features)
## [2026-04-08] reingest | TMC2240 rewrite from Docling PDF extraction → 2-Knowledge/Drivers/TMC2240.md (full ICS, SG2/SG4, encoder, protection, register map)
## [2026-04-08] reingest | TMC5160A rewrite from Docling PDF extraction → 2-Knowledge/Drivers/TMC5160A.md (Rev 1.17; SixPoint ramp, external MOSFET guide, dcStep, layout)
## [2026-04-08] reingest | TMC4671-LA rewrite from Docling PDF extraction → 2-Knowledge/Drivers/TMC4671-LA.md (Rev 2.08; hardware FOC, ADC config, PWM modes, commissioning)
## [2026-04-08] reingest | SIMC PID tuning rewrite from Docling PDF extraction → 2-Knowledge/Control-Theory/SIMC-PID-Tuning.md (Skogestad JPC 2003; τc rules, FOPDT/SOPDT, integrating processes)
## [2026-04-08] reingest | PID tuning methods rewrite from Docling PDF extraction → 2-Knowledge/Control-Theory/PID-Tuning-Methods.md (ZN, Cohen-Coon, Lambda/IMC, relay autotune, direct synthesis)
## [2026-04-08] reingest | Sliding mode control rewrite from Docling PDF extraction → 2-Knowledge/Control-Theory/Sliding-Mode-Control.md (Yu et al. 2023; PMLSM model, all SMC variants, stepper relevance)
## [2026-04-08] reingest | Feedforward compensators rewrite from Docling PDF extraction → 2-Knowledge/Control-Theory/Feedforward-Compensators.md (velocity/accel FF, IMC-based, FB+FF architecture)
## [2026-04-08] create | TMC driver comparison page → 2-Knowledge/Drivers/TMC-Driver-Comparison.md (cross-reference of TMC2209, TMC2240, TMC5160A, TMC4671-LA)
## [2026-04-08] create | Kalico TMC driver support → 2-Knowledge/Software/Kalico.md (6 TMC drivers, per-driver config, Kalico-specific additions vs. stock Klipper, sensorless homing)
## [2026-04-08] create | klipper_tmc_autotune → 2-Knowledge/Software/klipper_tmc_autotune.md (physics-based TMC register tuning from motor constants, design decisions, algorithm walkthrough)
## [2026-04-08] create | tmc-4671 driver → 2-Knowledge/Software/tmc-4671.md (closed-loop FOC servo driver for Klipper, TMC4671 hardware PID, S-IMC autotuning, encoder integration)
## [2026-04-08] ingest | AN-001 → 2-Knowledge/Drivers/AN-001-SpreadCycle-Parameterization.md (SpreadCycle chopper tuning: TBL, TOFF, hysteresis, behavioral/oscilloscope methods, low-L motors, 3-phase)
## [2026-04-08] ingest | AN-002 → 2-Knowledge/Drivers/AN-002-StallGuard2-CoolStep.md (StallGuard2/4 parameterization, CoolStep current scaling, load angle theory, dual-motor testing)
## [2026-04-08] ingest | AN-003 → 2-Knowledge/Drivers/AN-003-dcStep.md (dcStep load-adaptive velocity control, DC_TIME/DC_SG/VDCMIN tuning, TMCL-IDE wizard)
## [2026-04-08] ingest | AN-015 → 2-Knowledge/Drivers/AN-015-StealthChop-Performance.md (StealthChop vs SpreadCycle qualitative comparison: noise, vibration, power, mode switching)
## [2026-04-08] ingest | AN-026 → 2-Knowledge/Drivers/AN-026-Adaptive-Microstep-Table.md (custom microstep waveform optimization, laser pointer calibration, incremental encoding scheme)
## [2026-04-08] ingest | AN-064 → 2-Knowledge/Drivers/AN-064-TMC4671-Linear-Motor.md (TMC4671 FOC for linear motors, ABN encoder PPR, register config, linear scaling)
## [2026-04-08] revise | klipper_tmc_autotune.md — traced empirical constants (1.46, 374, 248×32) to AN-001 register encoding and sinusoidal averaging derivation; replaced speculative app note citations with verified derivations
