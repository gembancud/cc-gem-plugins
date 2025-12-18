# gem-plugins

A Claude Code plugin marketplace by Gil Emmanuel Bancud.

```
/plugin marketplace add https://github.com/gembancud/cc-gem-plugins
```

---

<div align="center">

![arctan logo](arctan-logo.jpeg)

**Plan before you code. Fix without losing context.**

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

</div>

---

## The Name

**Tangent** — going off on a tangent, diving straight into code without a plan.
**Arctan** — the inverse: stepping back to plan before implementing.
**Tan** — refining what arctan produced.

---

## Commands

| | `/arctan <feature>` | `/tan <issue>` |
|---|---|---|
| **Purpose** | Build new features | Fix issues after implementation |
| **Pipeline** | Explore → Plan → Refine → Implement → Review | Clarify → Fix → Review → Iterate |
| **Interaction** | One planning session with you | Checkpoints when ambiguous |
| **Best for** | New features, multi-file changes | Bugs, refinements, edge cases |

**Examples:**
```
/arctan add user authentication with JWT tokens
/tan login fails when email contains a plus sign
```

---

## Workflow

```
/arctan ──→ Test ──→ /tan ──→ Done
   │                   │
   └── implement ──────┴── refine
```

1. **`/arctan`** — Plan and implement your feature
2. **Test** — Try it out, find edge cases
3. **`/tan`** — Fix issues while staying aligned with the plan

---

## Inspiration

Built on ideas from Anthropic's [feature-dev](https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev) plugin (7 phases, multiple checkpoints).

arctan optimizes for speed:

- **Single interaction point** — One planning dialogue via `EnterPlanMode` instead of 4+ checkpoints
- **Native planning mode** — Uses Claude Code's built-in planning tool
- **Faster iteration** — Less context-switching, same agent-driven context control

---

## License

[MIT](LICENSE)
