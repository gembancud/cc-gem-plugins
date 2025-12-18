# gem-plugins

A Claude Code plugin marketplace by Gil Emmanuel Bancud.

## Installation

```
/plugin marketplace add https://github.com/gembancud/cc-gem-plugins
```

---

## arctan

![arctan logo](arctan-logo.jpeg)

### The Name

**Tangent** represents going off on a tangent—diving straight into code without a plan. **Arctan** (arctangent) is the inverse: stepping back to plan before implementing. **Tan** then refines what arctan produced.

### Commands

#### `/arctan <feature-description>`

Full feature development pipeline. Use this when starting new features.

```
Explore (Haiku) → Plan (Opus) → Refine (You + Opus) → Implement (Opus) → Review (Opus)
```

**When to use:**
- New feature implementation
- Complex changes spanning multiple files
- When you want structured planning before coding

**Example:**
```
/arctan add user authentication with JWT tokens
```

---

#### `/tan <issue-description>`

Fix and review loop. Use this after `/arctan` to address issues found during testing.

```
Clarify → Fix (Opus) → Review against plan (Opus) → Done or iterate
```

**When to use:**
- Bugs discovered after implementation
- Behavior that doesn't match expectations
- Refinements to recently implemented features

**Example:**
```
/tan login fails when email contains a plus sign
```

---

### Workflow

1. **Start with `/arctan`** — Plan and implement your feature
2. **Test the implementation** — Try it out, find edge cases
3. **Use `/tan` for fixes** — Address issues while staying aligned with the original plan

### Inspiration

arctan is inspired by Anthropic's [feature-dev](https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev) plugin, which provides a thorough 7-phase workflow with multiple checkpoints for user input.

arctan takes a different approach:

- **Single interaction point** — feature-dev asks for input at discovery, clarification, architecture selection, implementation approval, and review decisions. arctan consolidates this into one dialogue during plan refinement via `EnterPlanMode`.
- **Uses native planning mode** — Rather than a custom multi-option architecture phase, arctan leverages Claude Code's built-in `EnterPlanMode` tool where you and the agent refine the plan together until you're satisfied.
- **Faster iteration** — Fewer touchpoints means less context-switching. The agent gathers context, proposes a plan, you refine it once, then it executes.

Both approaches use agents to manage context properly. arctan just optimizes for speed when you trust the exploration and want to get to the collaborative planning faster.

## License

MIT
