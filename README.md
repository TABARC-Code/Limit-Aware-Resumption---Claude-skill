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

---

Design Principles
Deterministic continuation
Zero ambiguity in next step
Minimal redundant context
No repeated work
Explicit failure handling
Use Cases
Long-form code generation
Large document processing
Multi-step reasoning tasks
Iterative design workflows
Limitations
Requires user or system to trigger continuation
Cannot recover lost context unless included in block
Dependent on correct block usage
Philosophy

This skill treats Claude like a stateless compute node.

Instead of fighting that constraint, it formalises:

"Pause → Snapshot → Resume"

Status

Experimental but practical.

Designed for power users working at the edge of context limits.


---

# 📄 `SKILL.md`

```markdown
# Limit-Aware Resumption Skill Definition

## Purpose

To enforce consistent behaviour when Claude approaches or hits operational limits, ensuring safe interruption and precise continuation.

---

## Trigger Conditions

This skill activates when:

- Context window is nearing capacity
- Output is truncated mid-response
- System signals rate or usage limits
- Task size risks incomplete execution

---

## Core Behaviour

When triggered, Claude MUST:

### 1. Stop Cleanly
- Never cut mid-sentence
- Never leave partial structures (code, lists, JSON)

---

### 2. Summarise State
Include:

- Task objective
- Completed work
- Remaining work

---

### 3. Generate RESUMPTION BLOCK

Format:


=== RESUMPTION BLOCK ===
TASK: <clear task definition>
PROGRESS: <completed steps>
NEXT_STEP: <single precise next action>
CONTEXT: <minimal required data>
CONSTRAINTS: <rules to maintain continuity>
INSTRUCTION: Continue from NEXT_STEP without repeating prior work
=== END BLOCK ===


---

### 4. Provide Resume Instructions

Claude MUST explicitly instruct:

- "Start a new chat"
- "Paste this block"
- "Say: continue"

---

## Behaviour After Resume

When a RESUMPTION BLOCK is detected:

Claude MUST:

1. Parse block immediately
2. Reconstruct task state
3. Continue from NEXT_STEP
4. NOT:
   - Re-explain context
   - Repeat prior outputs
   - Ask unnecessary clarification

---

## Resumption Modes

### Mode A — Interactive Wait

- Claude states reset time
- Waits for user input trigger
- Does not restart task

---

### Mode B — Manual Resume

- Outputs RESUMPTION BLOCK
- Ends execution cleanly

---

### Mode C — Automated Resume

If requested, Claude MAY:

- Generate API-compatible prompt
- Provide automation script (Python, curl, etc.)
- Include scheduling instructions

---

## Failure Handling

If continuation fails:

Claude MUST:

- Request missing context explicitly
- Avoid guessing prior state
- Ask for updated RESUMPTION BLOCK

---

## Strict Rules

Claude MUST NOT:

- Claim to persist memory across sessions
- Claim to resume autonomously
- Invent prior context
- Continue without explicit instruction

---

## Optional Enhancements

Claude MAY:

- Compress context intelligently
- Prioritise critical data
- Offer multiple resume strategies

---

## Summary

This skill enforces:

**Controlled interruption + deterministic recovery**

Instead of:

**Unpredictable truncation + lost context**
