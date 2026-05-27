# MAGI System

Inspired by Evangelion's MAGI — independent AI units that deliberate, argue, and converge on a verdict. No entry plug required.

| Unit | Source |
|------|--------|
| Claude | Built-in Claude analysis |
| Codex (GPT) | [Codex CLI](https://github.com/openai/codex) |
| Gemini | [Gemini CLI](https://github.com/google/gemini-cli) |

## Prerequisites

At least one external CLI is recommended:

```bash
npm install -g @openai/codex        # Codex CLI
npm install -g @google/gemini-cli   # Gemini CLI
```

Without either, MAGI falls back to Claude-only analysis.

## Parallel execution (recommended)

Add to `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Usage

```
/magi Is this authentication implementation secure?
/magi Review the architecture of this module
```

Or natural language: "ask magi", "run magi", "cross analyze", "cross check", "cross verify", "cross review", "cross examine", "ask other AI"
