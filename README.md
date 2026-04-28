# claude-wise

Smart model selection for Claude Code. Auto-evaluates coding task complexity and recommends the optimal model (Opus vs Sonnet) to optimize both cost and time.

## Why

- **Opus** (~$15/Mtok) is overkill for simple tasks like CSS tweaks or config changes.
- **Sonnet** (~$3/Mtok) fails complex tasks — wrong output wastes 20+ minutes of human time.
- Manually evaluating task complexity defeats the purpose of having an AI assistant.

claude-wise has the AI evaluate complexity itself and recommend the right model.

## How It Works

### Session Workflow

```
THINKING PHASE (always Opus)
  Brainstorming -> Planning -> Model Handoff Recommendation

DOING PHASE (model assigned by handoff)
  Implementation on recommended model
  -> warns if complexity escalates mid-task
```

### Two Skills

| Skill | When | What |
|-------|------|------|
| `claude-wise:model-triage` | Conversation start | Scores task on 6 dimensions (algorithm, scope, domain, ambiguity, edge cases, risk). Total >= 15 -> Opus, < 15 -> Sonnet. |
| `claude-wise:model-handoff` | End of implementation plan | Evaluates the concrete plan and appends a Model Handoff Recommendation with escalation triggers. |

### One Command

`/triage <task description>` -- Manual evaluation with full scoring table output.

## Installation

```
/plugin marketplace add hnb-rabear/claude-wise
/plugin install claude-wise@claude-wise
```

## Configuration

### Custom Model Names

If you use custom model aliases (e.g., `max-power` for Opus), add to your CLAUDE.md:

```md
## Model Triage (config)
- Strong model: `max-power`
- Fast model: `medium-power`
```

### Project-Specific Overrides

Add "always strong model" domains to your project's CLAUDE.md to skip triage for complex areas:

```md
### Always strong model (skip evaluation)
- GPU/shader code
- Authentication / OAuth flows
- Database migrations
- Debugging subtle or intermittent bugs
```

When a task touches these domains, the plugin skips evaluation and stays on the strong model.

## License

MIT
