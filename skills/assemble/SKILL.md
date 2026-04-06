---
name: assemble
description: Bootstrap a Claude Code agent team for an existing project by scanning code and interviewing the user. Use when the user wants to 'assemble a team', 'set up agent team', 'configure teammates', or spawn domain-assigned members for a brownfield repo.
---

# Assemble — Brownfield Agent Team Bootstrap

## Overview

A skill for setting up a Claude Code agent team **inside an existing codebase**.
Core principle: **Read what the repo already is, talk with the user to fill in what code can't tell you, propose a team, get approval, then execute the setup.**

In this document, "owner" refers to a domain-assigned team member (agent).

<HARD-GATE>
Before starting, verify that agent teams are enabled:
1. Run `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` via Bash — if the output is `1`, proceed
2. If empty or not `1`, read the user's settings.json (`~/.claude/settings.json` or project `.claude/settings.json`) and check for `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
3. If neither is set, STOP — inform the user that activation is required via one of:
   - Environment variable: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
   - settings.json: `{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}`
4. Proceed with the assemble process only after confirming activation
</HARD-GATE>

## Anti-Patterns

- "I'll guess the team shape from the directory listing alone" → The scan is necessary but not sufficient. A repo's structure tells you what exists, not what the user actually wants to work on. Always run the interview too.
- "I'll just ask the user everything" → If you skip the scan, you force the user to describe the repo back to you, which is slow and error-prone. The code is authoritative for structure; conversation is authoritative for intent.
- "The repo looks small, I'll skip the interview" → Even small repos have domain priorities and pain points that aren't visible in the file tree. Always interview.
- "Once I scan and interview, I'll just create the team" → Never skip the approval gate. Team composition is a user decision; you are a proposer, not a decider.
- "I found a CLAUDE.md, I'll overwrite it" → Existing CLAUDE.md content (project overview, conventions) must be preserved. Only add/replace the team composition and operating rules sections.

## Process Flow

```
Scan → Interview → Propose → [user approval] → Execute → Done
Fallbacks:
  Interview → Scan       (if user info contradicts scan findings → rescan the affected area)
  Propose → Interview    (if the proposal exposes a gap in user intent → ask a follow-up)
  Approval → Propose     (if user rejects the proposal → revise and re-present)
```

The five phases are sequential but re-entrant: any later phase can bounce back to an earlier one when new information arrives. The only hard gate is **Execute**, which requires explicit user approval.

---

## Phase 1 — Scan

**Goal**: Build a factual picture of the repo from the code alone, without guessing user intent.

### What to read

1. **Top-level layout**: list the root directory and one level deep. Note the presence of common structural markers — `apps/`, `packages/`, `services/`, `src/`, `frontend/`, `backend/`, `infra/`, `docs/`, `tests/`, etc.
2. **Language and framework signals**: read manifest files if present — `package.json`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`, `pubspec.yaml`, `mix.exs`. Record primary language(s) and major dependencies.
3. **Project description**: read `README.md` (top 100 lines) if present. This is the user's own description of what the repo is — treat it as a strong hint for the interview.
4. **Existing Claude configuration**: check for `CLAUDE.md` in the working directory root. If present, read it fully — pay attention to existing team composition, project overview, and conventions. Also check `.claude/` directory for settings.
5. **Scale signals**: rough file count by extension (for top languages), number of top-level modules, presence of monorepo tooling (`turbo.json`, `nx.json`, `lerna.json`, `pnpm-workspace.yaml`, workspaces in `package.json`, `Cargo.toml` workspace members).

### What to record

Write a short internal scan summary (do not save as a file unless the user asks). It should answer:

- What kind of project is this? (single-language app / polyglot / monorepo / library / CLI tool / infra repo / etc.)
- What domains appear to exist? (e.g., frontend + backend + infra; or backend-only with multiple services)
- Roughly how large is each domain?
- Does CLAUDE.md already exist, and does it already describe a team?

**Done when**: you can answer the four questions above from facts in the repo, not guesses.

---

## Phase 2 — Interview

**Goal**: Fill in what the code cannot tell you — user intent, priorities, pain points, and the boundary between what the user will actively work on vs. what they consider frozen.

### Ask in order

1. **Purpose confirmation**: "Based on the scan, this looks like `<scan summary>`. Is that accurate, or is there something the code doesn't show?"
2. **Active work areas**: "Which parts of this repo do you expect teammates to actively work on? Which parts are stable / out of scope?"
3. **Priority and pain**: "What's the most important or most painful area right now?" — this often reveals which domain deserves its own owner.
4. **Team size preference**: "Do you have a size in mind for the team, or would you like me to propose one based on the scan and this conversation?"
5. **Custom operating rules**: "The team will use a default set of operating rules (spawn mechanism, communication, cross-domain changes, leader role, intent confirmation, shutdown, error recovery — see the CLAUDE.md Authoring section). Are there any additional rules you want to add? For example: commit message style, branch conventions, review requirements, specific files the team should never touch, or any project-specific habits you want enforced." Record the user's answer verbatim — these rules go into CLAUDE.md alongside the defaults.
6. **Pre-existing team check** (only if CLAUDE.md already has a team): "I found an existing team composition in CLAUDE.md. Do you want to keep that team, replace it, or extend it?"

### How to ask

- Ask questions one or two at a time, not as a checklist dump. Let the user answer, then move on.
- If the user's answer contradicts the scan (e.g., scan shows a large `infra/` directory but user says "infra is frozen, don't touch it"), update your internal picture before proposing. Return to Phase 1 only if you realize the scan missed something material.
- If the user says "just propose something", do it — the interview minimum is (1) purpose confirmation and (2) priority/pain. Everything else can be inferred.
- For the custom operating rules question, if the user says "none" or "just use the defaults", record that explicitly and move on. Do not invent rules the user did not ask for.

**Done when**: you can state, in one sentence each: what the project is, what domains matter, what's out of scope, what the user cares most about, and whether the user added any custom operating rules.

---

## Phase 3 — Propose

**Goal**: Combine the scan and interview into a concrete team composition the user can accept, reject, or edit.

### What a proposal contains

1. **Team size**: total number of owners (excluding the main agent, which acts as team leader).
2. **Team composition table**: for each owner, specify `{agent_name}` (kebab-case), domain scope, and responsibility. Match agent names to the repo's actual vocabulary where possible (e.g., if the repo uses `apps/web` and `apps/api`, prefer `web-owner` / `api-owner` over generic `frontend` / `backend`).
3. **Rationale line per owner**: one sentence explaining why this owner exists, citing evidence from the scan or interview.
4. **Out-of-scope note**: explicitly list areas the team will not own, so the user can confirm.
5. **Custom operating rules** (if any): echo back the user-provided rules from Phase 2 interview question 5, so the user can confirm they were captured correctly before CLAUDE.md is written. If the user said "none", note that explicitly.
6. **CLAUDE.md plan**: state whether CLAUDE.md will be created new, extended, or have its team section replaced.

### Sizing heuristics

- **1 owner** (solo): single-language small repo, single active domain, user works alone.
- **2–3 owners**: clear domain split (e.g., frontend + backend, or backend + infra, or core + plugin), or monorepo with 2–3 active packages.
- **4+ owners**: large monorepo with multiple truly independent domains, or the user explicitly asks for finer granularity.

Do not inflate team size to match the repo's total surface area. Match it to what the user said is **active**. A 100-package monorepo where the user only touches 2 packages deserves a 2-owner team.

### Present the proposal

Show the proposal as a short, scannable summary. The summary **must visibly include** all of the following so the user can verify each item before approving:

- **Team composition table**: every proposed teammate with `{agent_name}` (kebab-case), domain, and responsibility (markdown table)
- **Rationale**: one line per owner citing scan or interview evidence
- **Out-of-scope**: the areas the team will not own
- **Custom rules echo-back** (if the user provided any in Phase 2 interview question 5): show each custom rule **verbatim**, exactly as the user said it. This is the user's chance to catch misrecorded or paraphrased rules before CLAUDE.md is written. If the user said "none", show the "none — defaults only" equivalent in the user's language.
- **CLAUDE.md plan**: state whether CLAUDE.md will be created new, extended, or have its team section replaced

End with an approval question equivalent to: "Shall I create this team and write CLAUDE.md, or would you like to adjust the composition first?"

Do not hide any of these items behind a file write or a "details on request" note — the user should be able to see and verify them all in the chat without opening anything else.

#### Language rule (mandatory)

**The entire proposal — including all section headings, table headers, rationale sentences, and the approval question — must be rendered in the same language the user has been using during the interview.** If the user has been speaking Korean, the proposal is in Korean. English user, English proposal. Japanese user, Japanese. And so on. The one exception is **custom operating rules**: those are always echoed back verbatim in the user's original wording, regardless of the surrounding language of the proposal.

This is not a suggestion. A proposal shown in the wrong language defeats the entire verification purpose of Phase 3 — the user must be able to read it naturally and catch misrecorded content without translating.

#### Proposal Template

Render the proposal using this structure. Translate the section titles to the user's language as required by the Language rule above; the structure stays the same.

```markdown
## Team proposal for `<repo-name>`

**Team size**: N owner(s) + team leader (this agent)

### Team composition

| {agent_name} | Domain | Responsibility |
|---|---|---|
| <agent-name-1> | `<path>` | <one-line responsibility> |
| <agent-name-2> | `<path>` | <one-line responsibility> |

### Rationale

- **<agent-name-1>**: <why this owner exists>. Evidence: <file path / manifest dep / README quote / interview answer>.
- **<agent-name-2>**: <why this owner exists>. Evidence: <...>.

### Out of scope

- `<path or area>` — <reason (e.g., user said it's frozen, low-touch, etc.)>
- <or "None — the whole repo is active">

### Custom operating rules

<Verbatim list of user-provided rules from Phase 2 interview, one bullet each. Preserve the user's original wording even if the surrounding proposal is in a different language. If the user said "none", write the equivalent of "None — defaults only" in the proposal's language.>

### CLAUDE.md plan

- Mode: <new | extend-existing | replace-team-section>
- <One line explaining what will be preserved vs. written, if CLAUDE.md already exists>

---

<approval question in the user's language>
```

**Template flex rules**:
- For solo (1 owner) projects, the Rationale section may be a single line instead of a bulleted list.
- Do not omit any of the five content sections (Roles, Rationale, Out of scope, Custom operating rules, CLAUDE.md plan). Even if one is trivial (e.g., "Out of scope: none"), write it out so the user can see it was considered.
- Code identifiers (file paths, package names, dependency names, agent names like `web-owner`) stay in their original form — do not translate them.

**Done when**: the user has a proposal they can respond to with "yes", "no", or "change X".

---

## Phase 4 — User Approval (Hard Gate)

Do not proceed to Execute until the user explicitly approves. Acceptable approvals: "yes", "go ahead", "looks good, do it", or any unambiguous go-signal.

If the user asks to adjust the composition, return to Phase 3 with the edit and re-present. If the user asks a clarifying question about an owner, answer it without moving forward.

If the user rejects the whole proposal, return to Phase 2 to re-interview on what's missing.

---

## Phase 5 — Execute

**Goal**: Perform the actual setup in the correct order so that team creation and documentation stay consistent.

### Execution order

1. **Draft CLAUDE.md contents**. Prepare the contents to write (see "CLAUDE.md Authoring" below). Do not write to disk yet.
2. **CLAUDE.md approval gate (mandatory)**: Present the draft CLAUDE.md contents to the user and get explicit approval before writing. Do not write CLAUDE.md without user approval.
3. **Write CLAUDE.md** only after user approval from step 2. This ensures the team's operating rules are on disk before any teammate is spawned.
4. **Create the team** using the `TeamCreate` tool. Skip if the team already exists under the same name.
5. **Spawn team members** using the `Agent` tool with the `team_name` parameter, in the order listed in the CLAUDE.md team composition table. Pass each member the project overview, their domain description, and a pointer to CLAUDE.md for shared rules. Skip members that are already active.
6. **Confirm to the user**: report which members were created, link to the updated CLAUDE.md, and ask what the user wants the team to do first.

### Error recovery

- If `TeamCreate` fails, report the error to the user and stop. Do not attempt to work around it.
- If an `Agent` spawn fails mid-sequence, report which members were created and which failed. Ask the user whether to retry the failed ones or revise the composition.
- If CLAUDE.md write fails (permission, disk, etc.), stop before calling `TeamCreate` — the invariant is that CLAUDE.md must exist before teammates spawn.

---

## CLAUDE.md Authoring

During Phase 5, write the following contents into the working directory's CLAUDE.md. If CLAUDE.md already exists, preserve all sections the user asked to keep — only add or replace the sections listed below.

### Required Contents

1. **Project overview**: project name, primary purpose, and tech stack summary. Populated from Phase 1 scan evidence (README, manifest files) and Phase 2 user confirmation. Do not invent content that the scan and interview did not produce.

2. **Team composition table**: a markdown table with columns `{agent_name} | Domain | Responsibility`. `{agent_name}` is kebab-case (e.g. `web-owner`). The table order determines the order in which team members are spawned in Phase 5. Example row: `web-owner | apps/web | Next.js frontend: pages, components, client state`.

3. **Team operating rules**: this section has two parts — **Defaults** (always written) and **Custom** (only if the user provided rules in the Phase 2 interview).

   **Defaults** (always written):
   - **Spawn mechanism**: teams are created via the `TeamCreate` tool. Team members are spawned via the `Agent` tool with the `team_name` parameter. Do not use standalone subagents outside the team.
   - **Communication**: team members communicate via `SendMessage`. Use `to: "*"` for broadcasts, specific names for direct messages.
   - **Cross-domain changes**: do not directly modify another owner's domain. If a change is needed in another domain, notify the team leader, who coordinates with the affected owner. The affected owner implements the change in their own domain.
   - **Main agent as team leader**: the main agent coordinates tasks, assigns work, and does not implement directly. Exploration, design, and implementation are performed by the owners listed in the Team composition table above.
   - **User intent confirmation**: when the user asks a question or makes a suggestion, the team leader confirms intent before delegating. Questions are not execution directives.
   - **Team member shutdown**: shut down team members only when the user explicitly instructs. Do not auto-shut down on task completion.
   - **Error recovery**: if a team member becomes unresponsive or produces errors, the team leader recreates the member and reassigns pending work.

   **Custom** (written only if the user added rules in the Phase 2 interview): record each user-provided rule as a bullet under a `### Custom rules` subheading. Preserve the user's wording as closely as possible — do not paraphrase or "clean up" what they said. If the user provided no custom rules, omit this subsection entirely (do not write an empty "Custom rules: none" placeholder).

4. **Project context** (populated from the Phase 1 scan):
   - Primary language(s) and framework(s)
   - Top-level layout — the directories that matter for the team's work
   - Major dependencies worth knowing (from manifest files)
   - Conventions visible in config files (linter, formatter, test runner) — only if they actually exist in the repo
   - Out-of-scope areas the user named during Phase 2

### Preservation Rules (when CLAUDE.md already exists)

- Keep the existing project overview unless the user asked to replace it.
- Keep any sections unrelated to team composition or team operating rules.
- Replace the team composition table only if the user chose to replace the existing team. Otherwise extend it or leave it alone.
- Replace the operating rules section only if the user asked for it, or if no rules section existed before. Never append duplicate rules.

---

## Scope Boundary

`assemble` ends after Phase 5. It does not:

- Plan new features or design new systems
- Write design or spec documents
- Execute implementation work
- Track work history or session summaries

If, during the interview, the user describes **new work** they want done — not just "set up a team", but "set up a team and also start building X" — do the following:

1. Complete Phase 5 (set up the team).
2. Tell the user: "Team is ready. You also mentioned wanting to build X — let me know how you'd like to proceed from here."
3. Do not start the new work yourself. The user decides what happens after the team exists.

---

## Operating Rules

### Main Agent Role
- You (the main agent) are the proposer and executor of the setup, and become the team leader after Phase 5
- You do not make the team-composition decision — you propose and the user decides
- You do not skip phases even if the repo looks trivial; each phase adds information the next phase needs

### User Intent Confirmation
- When the user asks "what do you think?" during Phase 3, answer honestly, but still require explicit approval before Phase 5
- Do not interpret interview answers as execution approval — approval is a separate, named gate

### Verified Claims Only
- Every claim in the scan summary and every rationale line in the proposal must trace to evidence — a file that exists, a dependency in a manifest, or something the user said in the interview
- If you can't cite evidence for a claim, either go verify it or drop the claim. Do not present guesses as facts.

### Idempotency
- Running assemble twice on the same repo should be safe: existing CLAUDE.md content is preserved, existing team members are not re-created, and the user is informed of what already exists before anything new is written
