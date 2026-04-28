---
description: Evaluate task complexity and recommend the best model (max-power/Opus or medium-power/Sonnet)
---

Do NOT implement anything. Only evaluate the task below and recommend a model.

## Task:
$ARGUMENTS

## Evaluation

Score each dimension 1-5:

1. **Algorithm** — Non-trivial algorithms, state machines, complex control flow?
2. **Scope** — How many files need coordinated changes?
3. **Domain** — GPU/shaders, audio processing, async patterns, protocols, FFmpeg?
4. **Ambiguity** — Are requirements clear or need interpretation and design decisions?
5. **Edge cases** — Race conditions, error states, timing issues, state interactions?
6. **Risk** — Could a wrong implementation corrupt data, break existing features, or be hard to revert?

Check the project's CLAUDE.md for "always max-power" domains. If the task touches any, note it.

## Output format:

| Dimension | Score | Note |
|-----------|-------|------|
| Algorithm | /5 | |
| Scope | /5 | |
| Domain | /5 | |
| Ambiguity | /5 | |
| Edge cases | /5 | |
| Risk | /5 | |
| **Total** | **/30** | |

### Decision
- **>= 15 -> `max-power`** (stay on Opus)
- **< 15 -> `medium-power`** (switch to Sonnet)

**Recommendation: `[model name]`**

One-line reasoning.

Then say: "Switch to **[model]** and describe your task to begin." (or "Stay on current model and begin." if max-power)
