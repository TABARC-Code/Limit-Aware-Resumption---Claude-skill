# Limit-Aware Resumption — Description

Claude operates within a fixed context window and has no memory between sessions. When limits are reached, responses may truncate, earlier context is lost, and long-running tasks break down.

This project defines a structured skill that treats those limits as predictable failure points rather than errors..

---

## What It Does

Implements a **resumption protocol** that allows Claude to:

- Detect when it is approaching operational limits
- Pause execution at a clean boundary
- Capture the full task state in a structured format
- Enable continuation in a new session without repeating work

---

## What It Does Not Do

- Does not give Claude memory between sessions  
- Does not allow background execution or waiting  
- Does not create true automation without external tools  

Instead, it provide's a reliable **handover mechanism**.

---

## Core Idea

Rather than failing silently when limits are hit, Claude behaves like a system performing a controlled shutdown:

> Stop → Snapshot → Resume elsewhere

---

## How It Works

When a limit is reached, Claude emits a **RESUMPTION BLOCK** containing:

- Task definition.
- Completed progress.  
- Exact next step  .
- Minimal required context  

This block can be pasted into a new session to continue deterministically.

---

```mermaid
flowchart TD

A[Start Task] --> B[Claude Processes Task]

B --> C{Approaching Limit?}

C -- No --> B
C -- Yes --> D[Pause at Clean Boundary]

D --> E[Summarise State]

E --> F[Generate RESUMPTION BLOCK]

F --> G{User Choice}

G --> H[Stay-Alive Mode]
G --> I[Manual Resume Mode]
G --> J[Automated Cron Mode]

H --> K[User waits for reset]
K --> L[User sends 'continue']
L --> B

I --> M[User starts new chat]
M --> N[Paste RESUMPTION BLOCK]
N --> O[Claude reconstructs state]
O --> B

J --> P[Script scheduled externally]
P --> Q[API call with resumption prompt]
Q --> O

---

## Why It Matters

Without structure, long interactions degrade into:

- Repetition  
- Lost progress  
- Ambiguity  

This skill replaces that with:

- Continuity  
- Precision.  
- Predictable recovery.  

---

## Typical Use Cases

- Large code generation tasks  
- Multi-step reasoning workflows  
- Processing long documents  
- Iterative creative or technical builds  

---

## In One Line

A manual checkpointing system for a stateless AI.
