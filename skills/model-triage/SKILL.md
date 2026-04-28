---
name: model-triage
description: Auto-evaluate coding task complexity at conversation start and recommend the optimal model (max-power/Opus vs medium-power/Sonnet). Also monitors for mid-implementation complexity escalation.
---

# Model Triage

Evaluate task complexity and recommend the right model before any work begins. This skill optimizes for **time first, cost second** — a wrong model wastes 20+ minutes of human time.

## When This Skill Fires

At the **start of any conversation** when the user describes a coding task. Evaluate BEFORE doing any work, invoking other skills, or asking clarifying questions.

## Workflow

1. User describes a task.
2. Check the project's CLAUDE.md for "always max-power" domains. If the task matches any, skip evaluation and start working silently.
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

```
Model recommendation: `medium-power` (score 9/30)
  Algorithm 2 + Scope 1 + Domain 1 + Ambiguity 2 + Edge cases 2 + Risk 1
  Routine single-file endpoint following existing patterns.
Switch model before I begin?
```

## Output (when staying on max-power)

No output. Just start working. Do not interrupt the user for a non-actionable recommendation.

## Project-Specific Overrides

Projects can define "always max-power" domains in their CLAUDE.md. When the task touches any listed domain, skip evaluation entirely and stay on max-power. Example:

```md
### Always `max-power` (skip evaluation)
- mv/ folder (renderer, effects, shaders)
- CDP / Suno integration
- SSE streaming endpoints
```

These are owned by each project, not by this plugin.

## Mid-Implementation Escalation

During implementation on `medium-power`, if ANY of these occur, you MUST immediately warn:

1. **Discovered complexity** — a step is harder than predicted (unexpected async interaction, hidden state coupling).
2. **Repeated failure** — you have attempted the same step 2+ times with different approaches and failed.
3. **Scope expansion** — implementation requires touching files or systems not in the original plan.
4. **Domain escalation** — the step involves GPU, audio processing, protocol-level code, or other deep domain work not anticipated.

### Warning Format

```
Complexity escalated — recommend switching to `max-power`.
Reason: [specific reason, e.g. "Step 3 requires modifying GPU shader uniforms"].
This was not anticipated in the plan. Continue on medium-power or switch?
```
