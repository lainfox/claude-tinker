---
name: magi
description: 'MAGI System: independent AI units analyze your question. Activated by /magi, or phrases like "ask magi", "run magi", "cross analyze", "cross check", "cross verify", "cross review", "cross examine", "ask other AI".'
---

# MAGI System

Inspired by Evangelion's MAGI — independent AI units that deliberate, argue, and converge on a verdict. No entry plug required.

Each available unit (Claude, Codex/GPT, Gemini) tackles the same question independently. The Lead collects their outputs and synthesizes a unified verdict. The Lead does NOT produce its own analysis.

## Initialization

### 1. Build the prompt

Extract the user's question. For code-related requests, gather context automatically:

- `git diff` output (if uncommitted changes exist)
- Relevant source files
- Project structure overview

Bundle everything into a single prompt. Append: "Respond in the same language as the question."

### 2. Detect available CLIs

```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_MISSING"
which gemini 2>/dev/null && echo "GEMINI_AVAILABLE" || echo "GEMINI_MISSING"
```

Proceed with whatever is available. If neither CLI is installed, run Claude-only analysis.

## Deliberation

Spawn all available units in a single message so they run concurrently.

### Team mode (preferred)

```
TeamCreate(team_name: "magi", description: "MAGI System")
```

If this fails, tell the user: "Agent Teams를 사용하려면 settings.json env에 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`을 추가하세요. 활성화할까요?" — if declined, fall back to background mode below.

Spawn each unit with the composed prompt:

```
Agent(name: "claude-unit",  subagent_type: "claude-unit",  team_name: "magi", model: "sonnet", mode: "dontAsk", ...)
Agent(name: "codex-unit",   subagent_type: "codex-unit",   team_name: "magi", model: "sonnet", mode: "dontAsk", ...)
Agent(name: "gemini-unit",  subagent_type: "gemini-unit",  team_name: "magi", model: "sonnet", mode: "dontAsk", ...)
```

Only spawn codex-unit / gemini-unit if the respective CLI was detected.

### Background mode (fallback)

Same Agent calls, but without `team_name` and with `run_in_background: true`.

## Synthesis

Once all units have reported back, the Lead produces a single structured output. The Lead must NOT inject its own answer — only organize and reconcile what the units returned.

```markdown
## MAGI Results

### Consensus

- Points where all units agree (high confidence)

### Unique Insights

- **Codex (GPT)**: What only this unit found
- **Gemini**: What only this unit found
- **Claude**: What only this unit found

### Conflicts

- Where units disagree — state each position and the Lead's judgment on which is more accurate

### Verdict

- Final synthesized recommendation
```

After presenting results, shut down all teammates:

```
SendMessage(to: "claude-unit",  message: {type: "shutdown_request"})
SendMessage(to: "codex-unit",   message: {type: "shutdown_request"})
SendMessage(to: "gemini-unit",  message: {type: "shutdown_request"})
```

## Resilience

- **CLI missing** — skip that unit, proceed with the rest.
- **API / model error** — skip that unit, note the failure in results.
- **Timeout** — synthesize with whatever results have arrived.
- **All external CLIs fail** — synthesize from Claude-unit alone.
- **TeamCreate fails** — offer to enable Agent Teams, otherwise use background mode.
