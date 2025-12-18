---
description: Fix issue via Opus, then Opus reviews alignment with plan - with clarification checkpoints
argument-hint: <issue-description>
---

# Fix and Review Loop

The user has found an issue that needs fixing:

**Issue:** $ARGUMENTS

## Purpose

Handle iterative fixes while maintaining plan alignment. Includes clarification checkpoints to prevent quick patches from causing more bugs.

**CRITICAL: You (the main agent) must NEVER write fix code directly. Your job is to orchestrate agents and facilitate dialogue. All code changes go through the fix agent (Opus).**

## Pipeline with Checkpoints

```
You report issue
       ↓
┌─────────────────────────────────┐
│ CHECKPOINT 1: Is issue clear?   │
│ If ambiguous → clarify first    │
└─────────────────────────────────┘
       ↓
   Opus fixes
       ↓
   Opus reviews against plan
       ↓
┌─────────────────────────────────┐
│ CHECKPOINT 2: Review findings   │
│ If drift/ambiguity → clarify    │
└─────────────────────────────────┘
       ↓
   Done or next fix
```

---

## Checkpoint 1: Assess Before Fixing

**Before spawning any agent**, assess the issue on the main thread:

- Is the issue description clear and specific?
- Do you understand what "fixed" looks like?
- Is there any ambiguity about the expected behavior?

**If unclear:** Ask the user for clarification. Do NOT spawn the fix agent until you understand the issue.

Example clarifications:
- "When you say X fails, do you mean [A] or [B]?"
- "What should the expected behavior be here?"
- "Is this related to [specific part of plan] or something else?"

**If clear:** Proceed to Phase 1.

**If complex/unfamiliar code:** Consider a quick exploration first:

```
Task({
  description: "Explore context for fix",
  subagent_type: "Explore",
  model: "haiku",
  prompt: "Quick context gather for fixing [issue]. Find: the affected code, how it fits in the architecture, any related patterns. Be brief - this is prep for a fix, not a full exploration."
})
```

> **Context to include**: The specific issue, error messages or symptoms, which feature/component is affected.

Use this when the issue touches code you haven't recently explored, or when the fix location isn't obvious.

---

## Phase 1: Fix

Launch a **general-purpose agent** (Opus) to fix the issue:

```
Task({
  description: "Fix [issue]",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Fix this issue: [clear issue description].

  Context: Fixing an issue in a recently implemented feature. The approved plan is in .claude/ directory.

  Steps:
  1. Read the plan file to understand the intended architecture
  2. Locate the relevant code
  3. Make the minimal fix that aligns with the plan
  4. Run tests/typecheck to verify
  5. Return: what you changed, whether tests pass, how this aligns with the plan

  If you encounter ambiguity while fixing, note it clearly in your response rather than guessing."
})
```

> **Context to include**: Full issue details, relevant plan sections, any exploration findings, expected vs actual behavior.

---

## Phase 2: Review

Launch a **general-purpose agent** (Opus) to check plan alignment:

```
Task({
  description: "Review fix alignment",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Review this fix against the approved plan.

  The fix: [summary from fix agent]
  The plan is in .claude/ directory.

  Check for:
  1. Does this fix drift from the plan's architecture?
  2. Are there missing considerations from the original design?
  3. Is anything ambiguous that needs user clarification?
  4. Standard code quality (bugs, logic errors, security)
  5. Could this fix introduce new bugs?

  Be direct. Flag issues that need attention."
})
```

> **Context to include**: What the fix agent changed (files, logic), the original issue, plan location for reference.

---

## Checkpoint 2: Evaluate Review Findings

After review returns:

- **Clean** (fix aligns, tests pass, no issues) → Report success, ask "Anything else to fix?"
- **Issues found** (drift, ambiguity, side effects, new bugs) → Stop and discuss with user before proceeding. Present the specific concerns and let them decide next steps.

---

## Rules

1. **NEVER fix directly** - You are the orchestrator. Spawn Opus for all code changes.
2. **Clarify before fixing** - Don't spawn agents on ambiguous issues
3. **Plan is source of truth** - Reviewer checks alignment
4. **Stop on yellow/red** - Don't push forward when there's uncertainty
5. **User decides** - Present findings, let them make the call
6. **Note ambiguity, don't guess** - Agents should surface unknowns, not assume

---

## Begin

First, assess the issue at Checkpoint 1. Is this clear enough to fix, or do you need clarification?

Issue: $ARGUMENTS
