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

## License

MIT
