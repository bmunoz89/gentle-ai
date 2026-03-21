# Status Protocol (shared across all SDD skills)

## Overview

All SDD skills MUST return one of exactly four terminal status values. No other status values are permitted. This file defines what each status means and what payload fields are required.

## Status Enum

| Status | Meaning |
|--------|---------|
| `DONE` | Phase complete, artifact saved |
| `DONE_WITH_CONCERNS` | Phase complete but notable risks or deviations exist |
| `BLOCKED` | Cannot proceed, requires external resolution |
| `NEEDS_CONTEXT` | Requires user input before the phase can continue |

## Required Payload Fields by Status

### DONE

The phase completed successfully and its artifact was saved to the backend.

**Required fields:**
- `executive_summary` — one-sentence summary of what was produced
- `artifacts` — list of artifacts saved (each with `type` and `engram_topic_key` or `file_path`)
- `next_recommended` — the next SDD phase the orchestrator should offer to run

**Example:**
```
status: DONE
executive_summary: "Proposal created with 4 in-scope deliverables and a low-risk approach."
artifacts:
  - type: proposal
    engram_topic_key: "sdd/{change-name}/proposal"
next_recommended: "sdd-spec"
```

---

### DONE_WITH_CONCERNS

The phase completed and its artifact was saved, but notable risks, gaps, or deviations were identified that the orchestrator SHOULD surface to the user.

**Required fields:**
- `executive_summary` — one-sentence summary of what was produced
- `artifacts` — list of artifacts saved
- `next_recommended` — the next SDD phase
- `risks` — non-empty list of concerns (each item is a one-sentence description)

**Example:**
```
status: DONE_WITH_CONCERNS
executive_summary: "Brainstorm completed but intent remains partially unclear."
artifacts:
  - type: brainstorm
    engram_topic_key: "sdd/{change-name}/brainstorm"
next_recommended: "sdd-propose"
risks:
  - "Constraint on backward compatibility was mentioned but not fully elaborated."
  - "Two conflicting approach directions emerged — sdd-propose will need to resolve the tension."
```

---

### BLOCKED

The phase cannot proceed and requires external resolution before it can be re-launched.

**Required fields:**
- `blocker_description` — one-sentence description of what is blocking
- `resolution_needed` — concrete description of what is needed to unblock

**Prohibited fields:** Do NOT include `next_recommended` — there is no next step until the block is resolved.

**Example:**
```
status: BLOCKED
blocker_description: "No proposal artifact found for change 'add-payment-gateway'."
resolution_needed: "Run sdd-propose first to create a proposal before running sdd-spec."
```

---

### NEEDS_CONTEXT

The phase requires a specific piece of information from the user before it can continue. The orchestrator relays the question to the user, collects the answer, and re-launches the skill with the answer appended to the context.

**Required fields:**
- `question` — the exact text to present to the user; MUST be a single complete question ending in "?"; MUST be addressed directly to the user; MUST NOT be a statement or an instruction to the orchestrator
- `why` — one-sentence explanation of why this information is needed (for the orchestrator only — do NOT show this to the user)

**Example:**
```
status: NEEDS_CONTEXT
question: "What is the core goal you want to achieve with this change?"
why: "Understanding the primary intent is required before asking about constraints."
```

---

## Orchestrator Behavior for Each Status

| Status | Orchestrator Action |
|--------|---------------------|
| `DONE` | Surface `executive_summary` to user, offer `next_recommended` phase |
| `DONE_WITH_CONCERNS` | Surface `executive_summary` and `risks` to user, offer `next_recommended` |
| `BLOCKED` | Immediately surface `blocker_description` and `resolution_needed` to user — do NOT continue pipeline |
| `NEEDS_CONTEXT` | Present `question` verbatim to the user (do NOT rephrase), collect answer, re-launch skill with answer appended |

## Validation Rules

1. `question` in NEEDS_CONTEXT MUST end in "?"
2. `risks` in DONE_WITH_CONCERNS MUST be a non-empty list
3. BLOCKED MUST NOT include `next_recommended`
4. NEEDS_CONTEXT MUST NOT include `artifacts` or `next_recommended`
5. Any skill returning NEEDS_CONTEXT without a `why` field is malformed — the orchestrator MUST treat this as a warning before relaying the question

## Usage

Each skill's SKILL.md references this file in its "Execution and Persistence Contract" section:
```
Read and follow `skills/_shared/status-protocol.md` for the return status contract.
```

Sub-agents retrieve this file from `~/.claude/skills/_shared/status-protocol.md` (installed by `gentle-ai install`).
