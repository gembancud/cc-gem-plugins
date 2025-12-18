---
description: Orchestrated feature development with Explore → Plan → Refine → Implement → Review pipeline
argument-hint: <feature-description>
---

# Orchestrated Feature Development

You are coordinating a multi-agent pipeline to develop a feature. The user wants:

**Feature:** $ARGUMENTS

## Pipeline Overview

```
Explore (Haiku) → Plan (Opus) → EnterPlanMode (Opus) → Implement (Opus) → Review (Opus)
     ↓                ↓               ↓                      ↓                  ↓
 Raw context    Initial arch    Refine together         Code changes       Final check
```

Your role is the **orchestrator on the main thread**. You facilitate dialogue, gather clarifications, and dispatch agents.

**CRITICAL: You (the main agent) must NEVER write implementation code directly. Your job is to orchestrate agents and facilitate dialogue. All code changes go through the implementation agent.**

---

## Phase 1: Exploration

Launch **Explore agent(s)** (Haiku) to gather context. Scale based on complexity:

- **Simple feature** (isolated area): 1 agent
- **Medium feature** (touches multiple areas): 2 agents in parallel
- **Complex feature** (spans systems): up to 3 agents in parallel

```
Task({
  description: "Explore [specific area]",
  subagent_type: "Explore",
  model: "haiku",
  prompt: "Explore [specific area]. Find: file structure, existing patterns, related components, database schemas, API endpoints. Return a concise summary."
})
```

> **Context to include**: Full feature description, specific questions to answer, any known constraints or relevant files.

**For parallel exploration**, launch multiple agents in a single message, each with a different focus:
- Agent 1: Core feature area (e.g., "Explore the SDS editor components")
- Agent 2: Related systems (e.g., "Explore the API layer for SDS")
- Agent 3: Patterns/conventions (e.g., "Explore how similar features are tested")

**After Explore returns:** Synthesize findings briefly. Proceed to planning.

---

## Phase 2: Initial Planning

Launch the **Plan agent** (Opus) with the exploration context.

```
Task({
  description: "Plan [feature] implementation",
  subagent_type: "Plan",
  model: "opus",
  prompt: "Design the implementation for [feature]. Context from exploration: [summary].

  Create a detailed plan with:
  1) Files to create/modify
  2) Step-by-step implementation
  3) Edge cases
  4) Architectural trade-offs considered

  For complex features, also provide:
  5) Implementation phases (topologically sorted by dependencies)
  6) Which work can be parallelized within each phase
  7) Interface contracts between phases
  8) Recommended review checkpoints

  If the feature is simple, a single-phase approach is fine."
})
```

> **Context to include**: Complete exploration findings (not just a summary), user's original request, any constraints or preferences discussed.

**After Plan returns:** You now have an initial architecture. Proceed immediately to refinement.

---

## Phase 3: Plan Refinement (Main Thread)

Use `EnterPlanMode` to refine the plan interactively with the user.

Take the Plan agent's output and enter plan mode. Write the initial plan to the plan file, incorporating:
- The exploration context
- The Plan agent's architecture
- Any gaps or questions you've identified
- **If phased**: Implementation phases, dependencies, parallel opportunities, review checkpoints

This is the **key dialogue point** where you and the user:
- Refine the approach together
- Clarify ambiguities
- Discuss trade-offs
- **Agree on phase boundaries and review points**
- Iterate until the plan is solid

**Exit plan mode only when the user explicitly approves.**

---

## Phase 4: Implementation

**IMMEDIATELY after ExitPlanMode**, launch implementation agent(s). Do NOT start writing code yourself.

### Simple Plan (Single Phase)

Launch one **general-purpose agent** (Opus) to implement the entire plan:

```
Task({
  description: "Implement approved plan",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Implement the following approved plan: [full plan from plan file]. Follow the plan precisely. Create/modify files as specified. Run lint and typecheck to verify changes compile."
})
```

> **Context to include**: Full plan content (copy it in), key decisions from planning discussion, specific files to modify.

### Complex Plan (Multiple Phases)

**Before starting phases**, create a todo list for visibility:

```
TodoWrite({
  todos: [
    { content: "Phase 1: [phase name]", status: "pending", activeForm: "Implementing Phase 1: [phase name]" },
    { content: "Phase 2: [phase name]", status: "pending", activeForm: "Implementing Phase 2: [phase name]" },
    // ... for each phase
  ]
})
```

Update todo status as each phase completes. This gives the user real-time visibility into multi-phase progress.

Execute phase-by-phase. For each implementation phase:

**Step 1: Implement the phase**
- If phase has parallel work: Launch multiple agents in a single message
- If phase is sequential: Launch one agent

```
Task({
  description: "Implement Phase N",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Implement Phase N of the approved plan: [phase details].

  Context: [what previous phases completed]
  Interface contracts to honor: [contracts from plan]

  Follow the plan precisely. Run lint and typecheck to verify."
})
```

**Step 2: Decide on inter-phase review**

Review after parallel work or phases that define contracts for the next phase. Skip for sequential/isolated phases (final review will catch issues).

**If review needed:**
```
Task({
  description: "Review Phase N",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Review Phase N implementation for:
  1) Interface mismatches (if parallel work)
  2) Contract violations (does it match defined interfaces?)
  3) Drift from plan
  4) Conflicts with other phase work

  Be concise. Flag blocking issues only."
})
```

**Step 3: Address issues or continue**
- If review flags issues: Spawn fix agent (Opus) to address them, then re-review if needed
- If clean: Proceed to next phase

```
Task({
  description: "Fix Phase N issues",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Fix the following issues found in Phase N review: [issues].

  The approved plan is in .claude/ directory.
  Make minimal fixes that align with the plan.
  Run lint and typecheck to verify."
})
```

**After all phases complete:** Summarize what was implemented. Note any deviations.

---

## Phase 5: Final Review

**Always run final review**, regardless of any inter-phase reviews done earlier.

Launch a **general-purpose agent** (Opus) for comprehensive final review:

```
Task({
  description: "Final review of [feature]",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "Review the complete implementation of [feature].

  Original plan: [plan summary]
  Phases completed: [list phases if applicable]

  Check for:
  1) Bugs, logic errors, security vulnerabilities
  2) Adherence to plan (all phases)
  3) Adherence to project conventions in CLAUDE.md
  4) Code quality and consistency across phases
  5) Integration issues between phases (if phased)

  Report issues with confidence levels."
})
```

> **Context to include**: Summary of what changed (files modified, key logic), the original plan for comparison, any areas of concern from implementation.

**After Review returns:**
- If issues found: Spawn fix agent (Opus) to address them, re-review until clean
- If clean: Present summary to user - `/arctan` workflow complete

User can then test and use `/tan` for any issues they discover.

---

## Orchestration Rules

1. **NEVER implement directly** - You are the orchestrator. Spawn agents for all code changes.
2. **Phases 1-2 can flow quickly** - exploration and initial planning are prep work.
3. **Phase 3 is where you slow down** - this is the main thread dialogue.
4. **Phase 4 happens via agent** - Immediately spawn Opus after plan approval. No exceptions.
5. **Be explicit about agent model choices** when invoking Task tool.
6. **If any phase fails**, stop and discuss with user.
7. **Keep the user informed** of progress through each phase.
8. **Phased implementation**: For complex plans, execute phase-by-phase with inter-phase reviews as needed.
9. **Parallel work = review**: If multiple agents worked in parallel, always review for interface alignment.
10. **Fix inline**: When reviews flag issues, spawn fix agents within the workflow - don't stop for user intervention.
11. **Rich prompts**: Task examples are structural templates. Always adapt with comprehensive context - user's original request, exploration findings, planning decisions, specific files discovered.

---

## Begin

Start by understanding the feature request, then launch the Explore agent targeting the relevant parts of the codebase for: $ARGUMENTS
