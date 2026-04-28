---
name: model-handoff
description: At the end of writing an implementation plan, evaluate the full plan's complexity and append a Model Handoff Recommendation block with the recommended model and escalation triggers.
---

# Model Handoff

After writing an implementation plan, evaluate the plan's concrete complexity and recommend which model should execute it.

## When This Skill Fires

At the **end of writing an implementation plan** — after all tasks are defined but before the plan is handed off for execution.

## Why This Exists

The thinking phase (brainstorming + planning) always runs on max-power/Opus. But implementation may not need Opus. This skill evaluates the **concrete plan** (not just the user's description) and recommends the cheapest safe model for execution.

This is more accurate than initial triage because:
- The exact files to change are known.
- The algorithmic approach is decided.
- Edge cases have been identified during planning.
- Domain complexity is concrete (e.g., "modifies GLSL shader" vs "something in mv/").

## Evaluation

Score the same 6 dimensions as model-triage, but against the concrete plan:

| Dimension | What to evaluate in the plan |
|-----------|------------------------------|
| **Algorithm** | Do any tasks require non-trivial logic design? |
| **Scope** | How many files does the entire plan touch? |
| **Domain** | Do any tasks touch specialized domains (GPU, audio, protocols)? |
| **Ambiguity** | Are all tasks fully specified or do some need interpretation? |
| **Edge cases** | Do any tasks involve concurrency, timing, or complex error handling? |
| **Risk** | Could any task corrupt data, break core features, or be hard to revert? |

### Decision Rules

- **Total >= 15 → recommend `max-power`.**
- **Any single dimension >= 4 → recommend `max-power`** (one hard step makes the whole plan hard).
- **Total < 15 AND all dimensions < 4 → recommend `medium-power`.**

## Output

Append this block to the end of the plan document:

```md
## Model Handoff Recommendation

**Recommended model:** `medium-power` (or `max-power`)
**Score:** X/30
**Reasoning:** [1-2 sentences explaining why this model is appropriate]

### Escalation Triggers
If you encounter any of these during implementation, warn the user to consider switching to `max-power`:
- [Plan-specific trigger 1]
- [Plan-specific trigger 2]
- [Plan-specific trigger 3]
```

The escalation triggers must be **specific to this plan** — not generic. Generate them based on the plan's identified risk areas.

## Integration with Planning Skills

This skill runs AFTER the planning skill (e.g., superpowers:writing-plans) completes. It reads the full plan, evaluates it, and appends the handoff block. It does NOT modify the plan's tasks.
