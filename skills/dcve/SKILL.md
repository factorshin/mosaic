---
name: dcve
description: Structured design-to-implementation process for complex projects. This skill should be used when the user wants to "build a new project from scratch", "create a new service", "create a new app", "make a new service", "build an application", "build a new platform", "design a new system", "plan a new application", "start building from scratch", or needs to design and plan a multi-component project, application, or platform before implementation.
---

# DCVE — Devise · Curate · Verify · Execute

## Overview

A methodology for systematically cycling from idea generation through execution.
Core principle: **Run the D→C→V→E cycle. If changes arise after E, re-enter from D or V depending on scope.**

In this document, "owner" refers to a domain-assigned team member (agent).

<HARD-GATE>
Before starting, verify that agent teams are enabled:
1. Run `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` via Bash — if the output is `1`, proceed
2. If empty or not `1`, read the user's settings.json (`~/.claude/settings.json` or project `.claude/settings.json`) and check for `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
3. If neither is set, STOP — inform the user that activation is required via one of:
   - Environment variable: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
   - settings.json: `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
4. Proceed with the DCVE process only after confirming activation
</HARD-GATE>

## Anti-Patterns

- "I'll handle everything solo, it's faster" → Do not skip team formation for projects that warrant it
- "Skip design, just start coding" → Never skip V1 (Design & Feasibility Verification). V2/V3 may be adjusted by project scale, but V1 is mandatory
- "The user suggested it, let's execute immediately" → Distinguish suggestions from directives. Always confirm intent before acting

## Process Flow

```
Default flow: D → [user gate] → C → [user gate] → V1 → [user gate + scale] → V2 → V3 → [user approval] → E → Done
Solo flow:     D → [user gate] → C → [user gate] → V1 → [user gate + scale] → E → Done
Fallbacks:     V1→C (infeasible spec), Approval→V2 or V1 (revisions), E→D or E→V (scope change)
```

**User gate (V1 complete)**: Present a summary of spec.md (architecture, tech stack, key risks) and ask about project scale. Example: "Here is the architecture and tech stack. See `docs/spec/v{N}/spec.md` for details. Which project scale would you like?" Do not decide unilaterally. Present the scale options with a brief explanation of what each entails, and let the user choose:

- **Solo**: Skip V2 and V3. Proceed directly from V1 to E. Spec folder contains `devise.md`, `curated.md`, `spec.md` only — no `review-*.md`
- **Team** (2+ owners): Execute V2 and V3 fully. V3 cross-review scope is limited to directly adjacent domains. Re-review cycle (step 7) is still required

Even if the project appears small, explain the trade-offs (e.g., "This project could be handled solo, but forming a team would allow domain-separated reviews. Which approach would you prefer?") and wait for the user's decision before skipping any phase.

See `checklists.md` for detailed prompts and checklists for each phase.

---

## D — Devise

**Goal**: Generate as many ideas as possible without constraints.

1. Clarify the problem to solve or the service to build
2. List all possible features, approaches, and solutions **without judgment**
3. Devise ideas evenly across user, technical, and business perspectives
4. Record every idea in `docs/spec/v{N}/devise.md` (current spec version folder)

**Do NOT**: Evaluate ideas or assess feasibility. Do not limit quantity.

**Done when**: No new ideas emerge, and all three perspectives (user, technical, business) have been covered.

**User gate**: Present a summary of devise.md and ask: "These are the ideas generated. Shall I proceed to curate the core features? (See `docs/spec/v{N}/devise.md` for the full list.)"

---

## C — Curate

**Goal**: Select the essential, core ideas from the devised pool.

1. Identify core features that directly contribute to solving the problem
2. Distinguish core vs. nice-to-have features and prioritize (impact, difficulty, dependencies, urgency)
3. Merge overlapping or combinable features
4. Define the MVP scope
5. Record the curated result in `docs/spec/v{N}/curated.md` (include deselected ideas for future reference)

**Done when**: The feature list and priorities are finalized with clear criteria.

**User gate**: Present the MVP scope and core feature list from curated.md and ask: "This is the MVP scope. Shall I proceed to design? (See `docs/spec/v{N}/curated.md` for details.)"

---

## V — Verify

### V1. Design & Feasibility Verification

Transform the curated spec into an implementable design.

1. Design the overall system architecture
2. Select the tech stack (with rationale)
3. Identify technical bottlenecks and risk factors
4. Review from performance, scalability, and security perspectives
5. Estimate required resources (agent count, major dependencies, and implementation complexity per component)
6. Compile the project specification in `docs/spec/v{N}/spec.md` — the content scope depends on what the user is building
7. If an infeasible spec is found, return to the C phase

### V2. Team Assignment & Detailed Design

Partition the overall design into domains and assign owners.

#### Team Creation Procedure

1. Read CLAUDE.md in the working directory (if it exists)
   - If team-related content already exists (team composition, operating rules, or document structure): use the existing configuration as-is. Do not modify CLAUDE.md — skip to step 4
   - If no CLAUDE.md: define domain-specific agents with kebab-case names and draft the full CLAUDE.md contents (see "CLAUDE.md Authoring" section below)
   - If CLAUDE.md exists but has no team-related content: draft team composition and DCVE-specific items to add
2. **CLAUDE.md approval gate (mandatory)**: Present the draft CLAUDE.md contents to the user and get explicit approval before writing. Do not write CLAUDE.md without user approval
3. Write CLAUDE.md only after user approval from step 2
4. Create the team using the **TeamCreate** tool (skip if the team already exists)
5. Spawn team members using the **Agent** tool with `team_name` parameter, in the order listed in the CLAUDE.md team composition table (skip if team members are already active)

#### Each Owner's Responsibilities

1. Perform detailed design for their domain
2. Review feasibility of their domain
3. Derive task list, priorities, and execution order
4. Define the Definition of Done for each task
5. Write the review document as `review-{agent_name}.md` in the current spec version folder (e.g., `docs/spec/v1/review-backend-api.md`)
   - Include: domain design summary, feasibility assessment, identified risks, task list with priorities, and interface expectations toward adjacent domains

### V3. Cross-Team Consistency Verification

Verify that each owner's design does not conflict and align on interfaces.

1. Each owner reads collaborating team members' review documents from the current spec version folder (e.g., the `backend-api` owner reads `review-frontend-web.md`, `review-infra.md`)
2. Cross-check interface alignment (APIs, data formats, protocols, etc.) against their own review
3. Verify no field name/type/structure mismatches exist at layer boundaries
4. Check for differing interpretations of the same spec
5. Owners communicate freely via SendMessage to resolve conflicts. All conflict resolutions and design changes must be recorded in the respective review documents with change history (before → after)
6. Unresolved conflicts after one round (each involved owner has documented their position and one counter-response) → escalate to the user with both positions and a recommendation
7. After resolution, have relevant owners re-review the final revision (verify previous feedback was applied correctly and check for new issues)

**V phase done when**: All domain designs are finalized, no unresolved conflicts remain, and documentation is complete.

---

## E — Execute

**Pre-execution final checks**:
1. All V-phase verifications are complete
2. No unresolved conflicts or pending decisions remain
3. Each owner's final task list and order are confirmed
4. **User has granted execution approval**

Proceed only after approval.

### Execution Flow

1. Each owner implements tasks in the order defined during V2
2. The team leader monitors progress and coordinates between owners — does not implement directly
3. When an owner completes a task, they verify it against the Definition of Done and report completion to the team leader
4. The team leader tracks overall progress and reports milestone completions to the user
5. If an owner encounters an issue that affects another domain, they notify the team leader, who coordinates with the affected owner before proceeding

### E → D Re-entry Conditions

Re-enter the DCVE cycle when any of the following occur during execution:
- New feature request
- Existing spec requires modification
- Unexpected technical constraint discovered

On re-entry:
1. Identify the scope of impact (which domains, which owners)
2. Re-enter from D or V depending on the scope
3. Create a new spec version folder (`docs/spec/v{N+1}/`) — do not overwrite the previous version
4. Affected owners re-execute from the re-entry point through E with the new version
5. Reflect changes in the new version's documents with change history referencing the previous version

---

## Team Operating Rules

### Team Leader Role
- The main agent (the one running the DCVE skill) is the team leader
- Do not execute user commands directly — delegate to the responsible owner
- Exploration, design, and implementation are all performed by domain owners
- The team leader handles **task creation, assignment, coordination, and review only**

### User Intent Confirmation
- When the user asks for opinions or makes suggestions, **execute only after explicitly confirming their intent**
- Do not interpret questions or suggestions as immediate directives

### Team Member Shutdown
- Shut down team members **only when the user explicitly instructs it**
- Do not automatically shut down members even when work is complete

### Error Recovery
- If a team member becomes unresponsive or produces errors, the team leader recreates the member and reassigns pending tasks

### Document Management
- For Solo projects, the single agent manages documentation directly if the user opts in
- After team composition is confirmed in V2, **ask the user** whether to create and maintain change management documents (history.md, session_summary.md)
- If the user opts for document management, ask which team member (domain owner or team leader) should be the document manager. Inform the user of the trade-offs:
  - **Domain owner as document manager**: that owner's implementation performance may degrade due to increased context consumption
  - **Team leader as document manager**: the team leader's coordination ability may degrade as compaction discards conversation-based coordination context that cannot be recovered from files
- The designated document manager updates history.md and session_summary.md at the following points:
  - Each phase transition (when a user gate is passed)
  - Each task completion report received during E phase
  - Each E→D re-entry decision
- `history.md`: chronological log of decisions, changes, and their rationale
- `session_summary.md`: current state overview, completed work, and an open issues table at the bottom

---

## CLAUDE.md Authoring

During V2, when assigning owners, write the following in the working directory's CLAUDE.md. What to write depends on the current state:

- **CLAUDE.md already has team-related content** (team composition, operating rules, or document structure): **Do not modify CLAUDE.md.** Use the existing configuration as-is
- **No CLAUDE.md**: Draft all items (1–6) and **present to the user for approval before writing**
- **CLAUDE.md exists but has no team-related content**: Draft items 2, 3, 4, and 5 and **present to the user for approval before writing**. Preserve existing project overview and context

**CLAUDE.md approval gate (mandatory)**: CLAUDE.md를 생성하거나 수정할 때는 반드시 사용자에게 내용을 먼저 보여주고 승인을 받은 후에만 디스크에 쓴다. 승인 없이 CLAUDE.md를 쓰지 않는다.

### Required Contents

1. **Project overview**: Project name, purpose, core spec summary
2. **DCVE cycle overview**: Explain the full cycle so team members understand the workflow they operate within:
   - **Devise**: Unconstrained idea generation across user, technical, and business perspectives
   - **Curate**: Select core features, define MVP scope, prioritize by impact and difficulty
   - **Verify**: V1 designs architecture and assesses feasibility. V2 partitions domains, assigns owners, and each owner writes a detailed design review. V3 cross-reviews adjacent domains for interface consistency
   - **Execute**: Owners implement tasks in defined order; team leader coordinates and monitors
   - **User gates**: The user approves transitions between D→C, C→V1, V1→E (or V1→V2 for team scale), and V3→E. Execution requires explicit user approval
   - **Re-entry**: If scope changes during E, re-enter from D or V depending on impact; create a new spec version folder
3. **Team composition table**: `{agent_name}` (kebab-case, e.g. `backend-api`), domain, scope of responsibility (the table order determines team member creation order). This `{agent_name}` is used in file naming conventions such as `review-{agent_name}.md`
4. **Shared operating rules**: Applicable items from "Team Operating Rules" above
   - Must include: teams are created via **TeamCreate** tool, and team members are spawned via **Agent** tool with `team_name` parameter. Do not use standalone subagents outside the team.
   - Team members may communicate freely via SendMessage. All conflict resolutions and design changes must be recorded in review documents.
   - **Development flow**: Before implementing changes that affect multiple components or domain interfaces, write a spec in `docs/spec/v{N}/`. Use the current version folder unless the change significantly alters the existing spec. Adjacent team members review the spec and check interface alignment. After review is complete, get user confirmation before implementation.
   - **Review rules**: After completing their own review document, each member reads adjacent members' review documents and checks interface alignment (APIs, data formats, protocols). Conflicts are resolved via SendMessage and recorded in review documents (before → after). Unresolved conflicts after one exchange are escalated to the user. Review is complete when no interface mismatches remain and all changes are recorded.
5. **Document structure**: Document paths and conventions used in the project (`docs/spec/v{N}/`, `docs/history/`, etc.)
   - Must include: at session start, read the latest `docs/history/YYYY-MM-DD/session_summary.md` for prior context. `docs/history/history.md` is append-only — do not read at session start
6. **Project context** (when CLAUDE.md is newly created): project structure, tech stack, key conventions established during implementation — so that subsequent DCVE cycles can understand the existing codebase

---

## Document Structure

All project documents live under `docs/` in the project root:

```
docs/
├── spec/
│   ├── v1/
│   │   ├── devise.md                  # D phase — all ideas without judgment
│   │   ├── curated.md                 # C phase — core selection, priorities, MVP scope
│   │   ├── spec.md                    # V1 phase — project specification
│   │   ├── review-{agent_name}.md     # V2 phase — each owner's detailed design review
│   │   └── ...
│   └── v2/                            # Created on spec revision (E→D re-entry, V3 feedback, etc.)
│       └── ...
└── history/
    ├── history.md                     # Append-only log of major decisions and changes (when, what, why)
    └── YYYY-MM-DD/
        └── session_summary.md         # Session work summary, completed/pending tasks, open issues
```

**File roles**:
- `history.md`: append-only write. Do not read at session start — use session_summary.md instead
- `session_summary.md`: read at session start (latest date folder). Contains current state needed to resume work

**Version bump rules**:
- `v1` is created during the first D→C→V cycle
- V1→C fallback does not create a new version — the current version's documents are updated in place, since V1 is not yet finalized
- A new version folder (`v2`, `v3`, ...) is created when the spec is revised — either from E→D re-entry or major changes after V3 feedback
- Previous versions are kept as-is for traceability; never overwrite them

---

## Artifact Management Principles

1. **History preservation**: Record original and revised versions distinctly on every change
2. **Traceability**: Document the background and rationale for each decision
3. **Version control**: Mark the last modified date and version at the top of each document

---

## Reference

See `checklists.md` for detailed prompts (self-assessment questions) and checklists for each phase.
