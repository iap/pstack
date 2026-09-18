---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

- The user said "reflect" or "/reflect".
- A complex task (5+ tool calls) just landed cleanly and the recipe is worth keeping.
- The agent hit dead ends, found the working path, and the path generalizes.
- The user corrected the agent's approach mid-task.
- A non-trivial workflow emerged that isn't captured anywhere.

Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Locate the active transcript

The parent locates the current session via `session_search` (hermes stores sessions in its SQLite store; there are no JSONL transcript files). Query for the active conversation and take the most recent matching session id. If the exact session cannot be resolved, write a tight digest of the conversation and pass that instead.

### 2. Spawn three reviewers in parallel

One message, three `delegate_task` calls (role: `leaf`), explicit `model:` on each, agent mode (`readonly: false`). Reviewers need MCP access for context lookups (tickets, chat threads, observability traces referenced in the transcript); readonly on hermes restricts file writes only, so MCP access is unaffected. The prompt forbids file writes; the parent applies edits.

| Lens | `model` | Prompt template |
|---|---|---|
| Judgment | the reflect-judgment role model from `config/models.json` (fallback: parent chat model) | `references/judgment-reviewer.md` |
| Tooling | the reflect-tooling role model from `config/models.json` (fallback: parent chat model) | `references/tooling-reviewer.md` |
| Divergent | the reflect-judgment role model from `config/models.json` (fallback: parent chat model) | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the Hermes session id or digest where marked. Reviewers return findings in the `delegate_task` response body.

### 3. Synthesize

One `delegate_task` call (role: `leaf`), using the reflect-judgment role model from `config/models.json` (fallback: parent chat model), agent mode (`readonly: false`). The synthesizer's quality check includes spot-verifying citations, which can require MCP access; readonly on hermes restricts file writes only, so MCP access is unaffected. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. The synthesizer already applies this criterion; this is a final pass before edits land. See the **encode-lessons-in-structure** principle skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org; do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Those are tracker submissions, not skill edits. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to the hermes skill-authoring flow (the hermes-agent skill's guidance, or `skill_manage(action='create')`) and run its draft / test / iterate loop.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to the hermes skill-authoring flow's description pass.
- `new skill: <kebab-name>`: hand creation to the hermes skill-authoring flow. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done. Skip this step if it doesn't.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
