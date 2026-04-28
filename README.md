# claude-wise

Smart model selection for Claude Code.

## Why This Exists

Using the right model for the right task is critical for efficient agentic coding:
- **Cost problem:** Using `max-power` (Opus, ~$15/Mtok) for simple CRUD tasks or styling changes is overkill and expensive.
- **Time problem:** Using `medium-power` (Sonnet, ~$3/Mtok) for complex algorithmic tasks or deep refactoring often leads to failed implementations, loops of debugging, and wasted time (often 20+ minutes of human time lost).

`claude-wise` introduces a systematic triage framework to auto-evaluate task complexity and route to the optimal model.

## How It Works

The workflow uses a **Thinking vs. Doing** split, evaluating complexity at two key points:

1. **Conversation Start (Triage):** Auto-evaluates the user's initial request. If it's simple, recommends downgrading before starting.
2. **Implementation Handoff:** The planning phase always runs on `max-power` (Opus). After creating an implementation plan, the plan's *concrete complexity* is evaluated to recommend the cheapest safe model for the actual execution phase.

## Skills

| Skill | Description | When It Fires |
|-------|-------------|---------------|
| `model-triage` | Auto-evaluates coding task complexity across 6 dimensions. | Start of conversation. |
| `model-handoff` | Evaluates a completed implementation plan and appends a model recommendation with escalation triggers. | End of writing an implementation plan. |

## Commands

- `/triage <task description>`: Manually evaluate a task and get a model recommendation without actually starting the work.

## Installation

```bash
/plugin marketplace add hnb-rabear/claude-wise
/plugin install claude-wise@claude-wise
```

## Project-Specific Configuration

You can override the triage evaluation for specific domains in your project by adding an "always max-power" section to your project's `CLAUDE.md`:

```md
### Always `max-power` (skip evaluation)
- mv/ folder (renderer, effects, shaders)
- CDP / Suno integration
- SSE streaming endpoints
```

When a task touches these domains, the plugin will skip evaluation and stay on `max-power`.

## License

MIT
