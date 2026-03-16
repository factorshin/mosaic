# Mosaic2

A mosaic of skills crafted for Claude Code multi-agent team mode.

## Background

This plugin was born from real-world experience building complex systems with Claude Code agent teams. The first skill, DCVE, was designed to minimize spec omissions and conflicts between team members' designs before execution. As a result, the number of post-execution corrections dropped significantly — the team spent less time fixing issues and more time building.

## Skills

### DCVE (Devise · Curate · Verify · Execute)

A structured design-to-implementation methodology for complex projects.

1. **Devise** — Generate ideas without judgment
2. **Curate** — Select what matters most
3. **Verify** — Design, validate, assign teams, cross-check
4. **Execute** — Build with full team coordination

When changes occur after execution, the cycle re-enters from D.

The Verify phase is where DCVE differentiates itself: each team member reviews their own domain, then cross-reviews adjacent domains to catch interface mismatches, conflicting interpretations, and missing edge cases — all before a single line of code is written.

**Usage**: `/mosaic:run-dcve` or describe a project to build from scratch.

### Discuss (Multi-Perspective Discussion & Debate)

A structured discussion framework for decision-making.

1. **Setup** — Clarify the question and create a discussion team (Advocate, Critic, Synthesizer)
2. **Discussion** — 3 rounds: Opening Positions, Challenge & Rebuttal, Final Statements
3. **Synthesis** — Moderator compiles a structured report with recommendations

When you need advice, the skill creates agent team members with distinct perspectives who debate the topic and deliver a comprehensive report — covering strengths, risks, trade-offs, agreements, disagreements, and a final recommendation.

**Usage**: `/mosaic:run-discuss` or ask for advice on a decision.

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

## License

MIT
