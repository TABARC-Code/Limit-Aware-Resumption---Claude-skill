# Limit-Aware-Resumption---Claude-skill
This repository defines a structured behavioural protocol ("skill") for handling Claude's context limits, response cutoffs, and session resets.
---

### Author: TABARC-Code

---

This repository defines a structured behavioural protocol ("skill") for handling Claude's context limits, response cutoffs, and session resets. A bit of a work in progress. Use, abuse and rework as you find.

# Limit-Aware Resumption Skill for Claude

## Overview

This repository defines a structured behavioural protocol ("skill") for handling Claude's context limits, response cutoffs, and session resets.

Claude operates within a fixed context window and has no persistent memory between sessions. This skill does **not** attempt to bypass those constraints. Instead, it standardises how Claude:

- Detects approaching limits
- Pauses execution cleanly
- Emits a full task state snapshot
- Enables deterministic resumption in a new session

---

## Problem

Claude cannot:

- Persist state across sessions
- Wait in the background
- Resume autonomously after a reset
- Act on timers or scheduled triggers

As a result, long-running tasks are interrupted when:

- Context window fills
- Output is truncated
- Session expires or user leaves

---

## Solution

This skill implements a **Resumption Protocol**.

When nearing a limit, Claude must:

1. Stop at a logical boundary (never mid-sentence)
2. Summarise current progress
3. Emit a structured `RESUMPTION BLOCK`
4. Provide exact instructions for continuation

---

## Resumption Modes

### 1. Stay-Alive Mode (Interactive)

- Claude pauses and states expected reset time
- User keeps session open
- User sends a simple trigger message ("go", "continue")
- Claude resumes immediately

---

### 2. Fire-and-Forget Mode (Manual Resume)

- Claude outputs a copyable `RESUMPTION BLOCK`
- User starts a new chat later
- User pastes block + "continue"
- Claude reconstructs state and resumes

---

### 3. Cron Mode (Automated Resume)

- Claude generates a script (e.g. Python)
- Script is scheduled externally (cron, Task Scheduler, n8n, Zapier)
- Script sends resumption prompt via API
- Output is captured automatically

---

## Key Constraint

Claude **cannot actually wait or self-trigger**.

All "waiting" behaviour is simulated via:

- Explicit timestamps
- User or external system intervention

---

## Resumption Block Format

```text
=== RESUMPTION BLOCK ===
TASK: <description>
PROGRESS: <what's completed>
NEXT_STEP: <exact next action>
CONTEXT: <essential data only>
INSTRUCTION: Continue from NEXT_STEP without repeating prior work
=== END BLOCK ===
