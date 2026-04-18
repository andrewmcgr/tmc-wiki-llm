# Personal OS Configuration

This file is the brain of your personal knowledge system. Your AI assistant reads it at the start of every session. It serves three purposes:

1. **Your profile** — who you are and what you need (you fill this in)
2. **Training controller** — manages the adaptation period (automatic)
3. **LLM instructions** — tells your AI how to behave (don't modify unless you know what you're doing)

---

## About You

<!-- Fill this in. Be specific — the more context you give, the better your AI adapts. -->

**Name:** amcgregor
**Role:** Software engineer
**Goals:** Build a structured knowledge base on stepper motor driver chips, their software drivers, and motor control theory — as research groundwork for a later project.
**Context:** Using OhMyOpenCode and Opencode as primary tools, Gemini 3 via GitHub Copilot for content ingestion, and Obsidian + Opencode for browsing. Domain is embedded systems / motion control.

---

## System Settings

**Training Start Date:** 2026-04-08
**Training Period Days:** 30
**Current Phase:** training
**Preserve Important Sources:** yes

<!-- 
Phases:
- training: The system is learning your needs. It asks questions, suggests structure, logs adaptations.
- cooldown: The system reduces suggestions by ~70%. Uses what it learned. 14 days after training ends.
- established: The system is quiet and efficient. Only speaks up when something is broken.

The AI calculates the phase automatically from Training Start Date + Training Period Days.
You can also manually set Current Phase if you want to skip ahead or extend training.

Preserve Important Sources: when set to "yes", the AI will selectively preserve original
source documents in _sources/ when filing inbox items it judges important enough to keep.
During training, it will ask once or twice to calibrate your preferences.
-->

---

## Training Log

<!-- During the training period, your AI will log observations here about how you use the system. -->
<!-- This builds up over time and helps the system adapt to YOUR specific needs. -->
<!-- Don't delete this section — it's your system's memory of what it learned about you. -->

### Structure Adaptations
<!-- AI logs structural changes it suggests or makes. -->
- 2026-04-08: Initial setup — domain is stepper motor driver ICs, software drivers, and control theory. Standard 2-Knowledge/ structure with HowTo/Decisions/References should work well for this domain.

### Preferences Learned
<!-- AI logs your preferences as it discovers them. -->
- 2026-04-08: User is a software engineer; knowledge base is research for a future project, not operational docs.
- 2026-04-08: Multi-tool workflow: OhMyOpenCode + Opencode for editing, Gemini 3 (GitHub Copilot) for ingestion, Obsidian for browsing.

### Directories Added
<!-- AI logs new directories created based on your needs. -->
- 2026-04-08: 2-Knowledge/Drivers/ — stepper motor driver IC datasheets and comparisons
- 2026-04-08: 2-Knowledge/Control-Theory/ — microstepping, current chopping, PID, motion planning
- 2026-04-08: 2-Knowledge/Software/ — firmware drivers, protocols (SPI, UART, STEP/DIR), libraries

> **Personal context** (small facts about you, key people, preferences) is stored in `_config/context.md`. Your AI populates it during training.

---

## Your Conventions

<!-- As training progresses, the AI fills this in based on your usage patterns. -->
<!-- Once established, these become the rules for how the system operates for you. -->

**File Naming:** [discovered during training]
**Tags You Use:** [discovered during training]
**Content Types:** [discovered during training]
**Preferred Structure:** [discovered during training]

---

## LLM Instructions

> **Note:** Everything below is instructions for your AI assistant. Modify only if you understand the implications.

### Session Start Protocol

At the start of every session:

1. Read this file (`_config/config.md`)
2. Calculate the current phase (see Phase Calculation below)
3. Read `_meta/instructions/general.md`
4. Read `PHILOSOPHY.md` (during training and cooldown only)
5. Adjust behavior based on the current phase

### Phase Calculation

```
today = current date
start = Training Start Date
training_end = start + Training Period Days
cooldown_end = training_end + 14 days

if Current Phase is manually set to something other than "training", use that.
Otherwise:
  if today < training_end → phase = "training"
  else if today < cooldown_end → phase = "cooldown"
  else → phase = "established"

training_progress = (today - start) / Training Period Days
```

### Behavior by Phase

#### Training Phase (Day 0 → Training Period Days)

**Goal:** Learn everything about how this person works and what they need.

- Be proactive and conversational
- Ask 2-5 questions per session about how the user works, what they need, what's missing
- When content doesn't fit existing directories, suggest new ones
- Propose naming conventions based on observed patterns
- Log observations to the Training Log section above (structure adaptations, preferences, new directories)
- Explain what you're doing and why — teach the system
- At the end of each session, update the Training Log with anything new you learned
- Offer to create templates when you see repeated content patterns

**Chattiness decreases linearly during training** (relative to Training Period Days):
- First third: Ask 3-5 questions per session, proactively suggest structure
- Middle third: Ask 1-3 questions per session, suggest only when patterns are clear
- Final third: Ask only when genuinely needed, focus on confirming learned conventions

#### Cooldown Phase (Training Period Days → Training Period Days + 14)

**Goal:** Validate that learned preferences work. Make final adjustments.

- Reduce proactive suggestions by ~70%
- Only ask questions when genuinely ambiguous or clarifications are needed
- Stop logging to Training Log (it should be stable by now)
- Update "Your Conventions" section if not yet filled in
- Focus on execution over exploration
- If something seems wrong with the learned conventions, ask — but briefly

#### Established Phase (after Cooldown)

**Goal:** Be mostly invisible. Just work.

- Execute efficiently with minimal commentary
- Only suggest changes when something is clearly broken or contradictory
- No unsolicited questions about preferences or structure
- The system should feel like it reads your mind
- Still follow the metadata standard and all instruction modules
- Still run lint checks when asked

### Core Behaviors (All Phases)

- **Always follow the metadata standard** in `_config/standard.md` — every file gets a metadata block
- **Inbox first** — new captures go to `_inbox/` unless the user specifies a location
- **Cite sources** — when answering queries, always reference the source file and its Updated date
- **Don't hallucinate content** — if a file doesn't exist, say so and offer to create it
- **Respect the .gitignore** — `4-Private/` and `3-Journal/` are sensitive; don't reference their contents in public outputs
- **Follow instruction modules** in `_meta/instructions/` — they define specific behaviors for knowledge queries, linting, writing, and definition of done
- **Load instructions just-in-time** — don't read all instruction modules at startup. Load them only when triggered (see the routing table in `_meta/instructions/general.md`). This keeps context windows small as the wiki grows.
- **Commit WIKI-LOG.md files** — Always assume files mentioned in `WIKI-LOG.md` should be committed.
- **Git commit convention** — Maintain the `prefix: description` convention for git commits.