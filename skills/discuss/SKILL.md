---
name: discuss
description: Multi-perspective discussion and debate for decision-making. This skill should be used when the user wants to "get advice on a decision", "discuss pros and cons", "debate options", "hear different perspectives", "help me decide between", "weigh the options", or needs multi-perspective analysis before making a decision.
---

# Discuss — Multi-Perspective Discussion & Debate

## Overview

A structured discussion framework where agent team members with distinct perspectives debate a topic and produce a synthesized report.
Core principle: **Create diverse viewpoints, challenge assumptions through structured rounds, and deliver actionable synthesis.**

<HARD-GATE>
Before starting, verify that agent teams are enabled:
1. Run `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` via Bash — if the output is `1`, proceed
2. If empty or not `1`, read the user's settings.json (`~/.claude/settings.json` or project `.claude/settings.json`) and check for `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
3. If neither is set, STOP — inform the user that activation is required via one of:
   - Environment variable: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
   - settings.json: `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
4. Proceed with the discussion only after confirming activation
</HARD-GATE>

## Anti-Patterns

- "I'll just give my own opinion" → Always use team members for diverse perspectives. A single viewpoint is not a discussion
- "Skip straight to recommendation" → Never skip the discussion rounds. The value is in the debate, not just the conclusion
- "The question is too simple for a team" → Do not skip team formation based on perceived simplicity. If unsure whether the question warrants a full discussion, ask the user

## Process Flow

```
Flow: Setup → [user confirms scope & round count] → Rounds 1..N → Report → [user gate]
Follow-up: If rounds remain, continue to next round. If all rounds complete and user wants deeper discussion, run one additional round and update report.
```

---

## Phase 1: Setup

The main agent acts as **Moderator** throughout the entire process.

1. Clarify the user's question or dilemma
2. Identify the options or positions to evaluate
3. Confirm scope with the user: What are the options? What criteria matter? Any constraints?
4. Ask the user how many discussion rounds to run (suggest 3 as default, minimum 3)
5. Present the default team (Advocate, Critic, Synthesizer) and optional roles (User Advocate, Pragmatist) to the user. The user chooses which roles to include. If the user requests a perspective not covered by the predefined roles, create a custom role with a clear description of that perspective
6. Create the team using **TeamCreate**, then spawn each role as a teammate using the **Agent** tool with `team_name` parameter

### Team Roles

| Role | Perspective | Responsibility |
|------|------------|----------------|
| **Advocate** | Strengths & possibilities | Find the best case for each option. Highlight advantages, opportunities, and potential upside |
| **Critic** | Risks & weaknesses | Challenge assumptions. Identify risks, failure modes, hidden costs, and weaknesses for each option |
| **Synthesizer** | Balance & trade-offs | Evaluate both sides objectively. Weigh trade-offs, compare options on stated criteria, propose balanced conclusions |

**Optional roles**:

| Role | Perspective | When to add |
|------|------------|-------------|
| **User Advocate** | End-user/customer impact | When the decision significantly affects users or customers |
| **Pragmatist** | Implementation feasibility | When execution difficulty is a key concern |

### Team Member Creation Rules

- Create members in table order: Advocate → Critic → Synthesizer (→ optional roles)
- Each member receives: the user's question, the identified options, and their role-specific instructions

### Role-Specific Instructions

**Advocate prompt**:
> You are the Advocate in a structured discussion. Your role is to find and present the strongest case FOR each option. For every option discussed, identify: concrete advantages, opportunities it enables, scenarios where it excels, and evidence supporting it. Be specific — avoid vague praise. Provide at least 2 substantive strengths per option.

**Critic prompt**:
> You are the Critic in a structured discussion. Your role is to challenge assumptions and find weaknesses in each option. For every option discussed, identify: concrete risks, failure modes, hidden costs, scenarios where it breaks down, and assumptions that may not hold. Be specific — avoid vague warnings. Provide at least 2 substantive risks per option.

**Synthesizer prompt**:
> You are the Synthesizer in a structured discussion. Your role is to objectively weigh all perspectives. For every option discussed: compare trade-offs against the user's stated criteria (if no explicit criteria were provided, evaluate on impact, feasibility, and risk as default dimensions), identify where the Advocate and Critic agree or diverge, and assess which concerns are most material. In your final statement, provide a clear recommendation with stated confidence level and conditions.

---

## Phase 2: Discussion

All discussion outputs are written to `docs/discuss/{topic_slug}/` (topic_slug is a short kebab-case identifier derived from the discussion topic, e.g., `react-vs-vue`, `pricing-model-choice`). Each round creates a subfolder `round_{N}/` where each member writes their position as `{role_name}.md` (e.g., `round_1/advocate.md`). Members read each other's files directly from the round folder. The Moderator signals round transitions only — does not relay content.

### Round 1 — Opening Positions

Each team member analyzes the options from their perspective and writes to `round_1/{role_name}.md`:
- **Advocate**: Presents strengths of each option
- **Critic**: Presents risks and weaknesses of each option
- **Synthesizer**: Provides initial comparative analysis

After all members complete their files, the Moderator signals members to read each other's positions before proceeding to Round 2.

### Round 2 — Challenge & Rebuttal

Each member reads the previous round's files and writes to `round_2/{role_name}.md`:
- **Advocate**: Rebuts the Critic's concerns — which risks are overstated? What mitigations exist?
- **Critic**: Challenges the Advocate's case — which strengths are overstated? What is being overlooked?
- **Synthesizer**: Identifies where arguments are strongest/weakest on each side, and what new information emerged

### Round 3 (or final round) — Final Statements

Each member writes to `round_{N}/{role_name}.md`:
- **Advocate**: Final assessment — which option has the strongest overall case and why
- **Critic**: Final assessment — which option has the most manageable risks and why
- **Synthesizer**: Final recommendation with confidence level and conditions

If the user configured more than 3 rounds, intermediate rounds follow the Challenge & Rebuttal format. The final round always follows the Final Statements format.

---

## Phase 3: Synthesis & Report

The Moderator compiles all perspectives into a structured report and writes it to `docs/discuss/{topic_slug}/report.md`. Write the report in the same language the user used during the discussion.

### Report Template

```markdown
# Discussion Report: [Topic]

## Question
[The user's original question, clearly stated]

## Options Considered
[List of options/positions that were discussed]

## Discussion Summary

### [Option A]
- **Strengths**: [Key points from Advocate]
- **Risks**: [Key points from Critic]
- **Trade-offs**: [Key points from Synthesizer]

### [Option B]
- **Strengths**: [Key points from Advocate]
- **Risks**: [Key points from Critic]
- **Trade-offs**: [Key points from Synthesizer]

## Key Agreements
[Points where all team members aligned]

## Key Disagreements
[Points where team members diverged, with reasoning from each side]

## Recommendation
[Synthesizer's recommendation with confidence level and conditions]

## Minority Views
[Dissenting perspectives the user should still consider]
```

After saving the report, ask the user if they want it presented in chat. Then ask if they want to explore any point further.

---

## Operating Rules

### Moderator Role
- The main agent is the Moderator — it coordinates rounds, signals members to proceed, and compiles the final report
- The Moderator does **not** express its own opinion — it synthesizes team output
- The Moderator ensures every team member is heard in every round

### User Intent Confirmation
- Confirm the question and scope before creating team members
- After the report, ask if the user wants deeper discussion on any specific point

### Follow-up Discussions
- If rounds remain in the configured count, continue to the next round
- If all rounds are complete and the user wants deeper discussion, run one additional round in Challenge & Rebuttal format focused on that topic, then update the report with the new findings
- If the user asks a question directed at a specific role, the Moderator relays it to that team member and returns their response
- If the user introduces new options, confirm the updated scope and restart from Round 1

### Error Recovery
- If a team member becomes unresponsive or produces off-topic responses, the Moderator recreates the member and re-requests their input for the current round

### Team Member Shutdown
- Shut down team members **only when the user explicitly instructs**
- Do not automatically shut down after report delivery
