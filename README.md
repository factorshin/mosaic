# Mosaic — DCVE Methodology Plugin

A mosaic of skills crafted for Claude Code multi-agent team mode.

DCVE (Diverge · Converge · Verify · Execute) is a systematic methodology for project planning and execution using Claude Code agent teams.

## Background

This plugin was born from real-world experience building a backend system with Claude Code agent teams. During the project, the DCVE cycle was used to minimize spec omissions and conflicts between team members' designs before execution. As a result, the number of post-execution corrections dropped significantly — the team spent less time fixing issues and more time building.

## What it does

Guides you through a structured cycle:
1. **Diverge** — Generate ideas without judgment
2. **Converge** — Select what matters most
3. **Verify** — Design, validate, assign teams, cross-check
4. **Execute** — Build with full team coordination

When changes occur after execution, the cycle re-enters from D.

The Verify phase is where DCVE differentiates itself: each team member reviews their own domain, then cross-reviews adjacent domains to catch interface mismatches, conflicting interpretations, and missing edge cases — all before a single line of code is written.

## Requirements

- Claude Code with agent teams enabled via one of:
  - Environment variable: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
  - settings.json: `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`

## Installation

### From Marketplace
```
/plugin marketplace add factorshin/mosaic-marketplace
/plugin install mosaic@mosaic-marketplace
```

### Local Development
```bash
claude --plugin-dir ./mosaic
```

### Verify
Start a new session and type:
```
/dcve
```

Or describe your project naturally — the skill triggers automatically when you want to build a new project, service, or platform from scratch.

## License

MIT
