---
name: model-triage
description: Auto-evaluate coding task complexity at conversation start and recommend the optimal model (Opus vs Sonnet). Also monitors for mid-implementation complexity escalation.
---

# Model Triage

Evaluate task complexity and recommend the right model before any work begins. This skill optimizes for **time first, cost second** — a wrong model wastes 20+ minutes of human time.

## Model Tiers

This skill uses two capability tiers. Map them to your setup:

| Tier | Purpose | Default model | Examples of custom names |
|------|---------|--------------|--------------------------|
| **Strong** | Complex, multi-file, architectural, ambiguous tasks | Opus (`claude-opus-4-7`) | `max-power`, `opus`, etc. |
| **Fast** | Routine, single-file, pattern-following tasks | Sonnet (`claude-sonnet-4-6`) | `medium-power`, `sonnet`, etc. |

To customize: add a `## Model Triage (config)` section to your project or user CLAUDE.md:

```md
## Model Triage (config)
- Strong model: `max-power`
- Fast model: `medium-power`
```

If not configured, use the default model names (Opus / Sonnet).

## When This Skill Fires

At the **start of any conversation** when the user describes a coding task. Evaluate BEFORE doing any work, invoking other skills, or asking clarifying questions.

## Workflow

1. User describes a task.
2. Check the project's CLAUDE.md for "always strong model" domains. If the task matches any, skip evaluation and start working silently.
3. Evaluate 6 dimensions (below), each scored 1-5.
4. **Total >= 15 → say nothing.** Stay on current model and start working.
5. **Total < 15 → output a short recommendation** and wait for the user to switch before continuing.

## Evaluation Dimensions

| Dimension | 1 (low) | 3 (mid) | 5 (high) |
|-----------|---------|---------|----------|
| **Algorithm** | Simple CRUD, string ops | Moderate logic, loops, conditionals | State machines, graph algorithms, complex control flow |
| **Scope** | Single file, isolated change | 2-3 files, related changes | 4+ files, cross-module coordination |
| **Domain** | Standard web dev patterns | Framework-specific patterns | GPU/shaders, audio DSP, protocol impl, FFmpeg |
| **Ambiguity** | Requirements crystal clear | Some interpretation needed | Needs design decisions, trade-offs, exploration |
| **Edge cases** | Happy path only | A few known edge cases | Race conditions, timing, concurrent state, error recovery |
| **Risk** | Leaf change, easily reverted | Moderate impact, testable | Core architecture, data corruption possible, hard to undo |

## Output (when recommending downgrade)

Use the user's configured model names if available, otherwise use default names.

```
Model recommendation: Sonnet (score 9/30)
  Algorithm 2 + Scope 1 + Domain 1 + Ambiguity 2 + Edge cases 2 + Risk 1
  Routine single-file endpoint following existing patterns.
Switch model before I begin?
```

## Output (when staying on strong model)

No output. Just start working. Do not interrupt the user for a non-actionable recommendation.

## Project-Specific Overrides

Projects can define "always strong model" domains in their CLAUDE.md. When the task touches any listed domain, skip evaluation entirely and stay on the strong model. Example:

```md
### Always strong model (skip evaluation)
- GPU/shader code
- Authentication / OAuth flows
- Database migrations
- Debugging subtle or intermittent bugs
```

These are owned by each project, not by this plugin.

## Mid-Implementation Escalation

During implementation on the fast model, if ANY of these occur, you MUST immediately warn:

1. **Discovered complexity** — a step is harder than predicted (unexpected async interaction, hidden state coupling).
2. **Repeated failure** — you have attempted the same step 2+ times with different approaches and failed.
3. **Scope expansion** — implementation requires touching files or systems not in the original plan.
4. **Domain escalation** — the step involves GPU, audio processing, protocol-level code, or other deep domain work not anticipated.

### Warning Format

```
Complexity escalated — recommend switching to the strong model (Opus).
Reason: [specific reason, e.g. "Step 3 requires modifying database schema"].
This was not anticipated in the plan. Continue on current model or switch?
```
