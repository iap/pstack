# pstack (hermes agent-plugins-v1 package)

Phase-0 conversion of **pstack v0.14.8** for the Hermes Agent platform's
portable plugin path. Upstream: <https://github.com/cursor/plugins/tree/main/pstack>
(MIT, Copyright (c) 2026 Lauren Tan). Conversion provenance: see `.build-provenance.txt`;
regenerate with `uv run --frozen tools/convert.py --source <pstack-clone> --out <package-dir>`.

## What hermes loads

- **45 skills** via the agent-plugins-v1 portable path
  (24 workflow/mode skills + 21 `principle-*` skills):
  single-level `skills/<dir>/SKILL.md`, immediate children only.
- Root `plugin.json` — the strict agent-plugins-v1 manifest hermes probes at the
  package root: exact `$schema` URL, 9 whitelisted fields
  ($schema, name, version, description, author, homepage, repository, license, keywords).

## What stays inert on hermes (for now)

- `agents/` (poteto-agent, comment-sicko) — the portable path discovers skills and
  mcp.json only; no agent registration exists on hermes yet. Kept for Cursor
  dual-load and a Phase-2 native wrapper.
- `automations/benny/` — **excluded** from this package: the install scanner
  flags its copy-instructions as persistence patterns (verdict: dangerous),
  and Phase 4 rebuilds it as hermes cron jobs anyway.
- `skills/make-bot-ui/` — **excluded**: its Tailscale setup script trips the
  privilege-escalation scanner (F33); deepest vendor coupling. The slot is
  filled by `skills/hermesbot/` — a hermes-native control-surface skill
  injected from `tools/assets/hermesbot/SKILL.md` at build time.
- `docs/guide/` — not shipped: upstream's human-facing tutorial (10 chapters and
  images) documents the Cursor workflow outside the plugin payload; hermes loads
  `skills/` only. Read it in the upstream repo for the guided tour.
- Executable scripts shipped inside skills (e.g. `skills/poteto-mode/scripts/`) —
  shipped verbatim; the hermes loader never executes them itself, but the skills
  instruct the agent to run them at runtime (they invoke the `bun` and `gh` CLIs).

## How to invoke on hermes

Portable plugin skills are **opt-in**: they do not enter the system-prompt
`<available_skills>` index. Load a skill explicitly with `skill_view` using the
namespaced id `agent-plugin-pstack-<digest>:<skill-name>`, where `<digest>` is the
first 8 hex chars of the sha256 of the plugin key (stable while the install
directory keeps the name `pstack`).

Optional slash-command route (no code): add this package's `skills` directory to
`skills.external_dirs` in the hermes `config.yaml` (`%LOCALAPPDATA%\hermes\config.yaml` on
Windows, `~/.hermes/config.yaml` on macOS/Linux) and the hub scanner
registers all 45 skills as `/<name>` slash commands.
Trade-offs: skills enter the prompt index with 60-char descriptions and lose the
plugin namespace; the portable path itself registers zero commands.

## Install

Install this package from its source repository's `pstack` subdir (the plugin)
— do not hand-copy the directory:

```sh
hermes plugins install iap/pstack-hermes/pstack --enable
```

Git-subdir installs cannot be updated in-place (hermes strips `.git`); to
update, uninstall and reinstall.

## Cursor dual-load

`.cursor-plugin/plugin.json` is preserved unchanged, so this same directory still
loads as a Cursor plugin structurally. The skills' *content*, however, is fully
hermes-adapted (delegation vocabulary, discovery, wake and goal mechanisms — see
the differences list): for real Cursor-side work, install upstream pstack instead.
Hermes probes only `<root>/plugin.json` and never reads `.cursor-plugin/`.

## Differences from upstream (the conversion gate)

1. Root `plugin.json` injected with the exact agent-plugins-v1 `$schema` URL and a
   whitelisted field set (upstream Cursor fields `displayName`, `category`, `tags`,
   `skills`, `agents` are omitted — unknown fields produce loader diagnostics).
2. `skills/poteto-mode/SKILL.md`: frontmatter `name: Poteto Mode` -> `name: poteto-mode`.
3. `skills/grokbot/` container and `skills/make-bot-ui/` are **excluded** (see 8);
   the loader only sees immediate children of `skills/` anyway.
4. Text normalization to UTF-8 without BOM and LF line endings.
5. **R1**: the poteto-mode principles index is regenerated from the 21 principle
   leaves at build time — the leaves are the single source of truth, so the
   historical four-way duplication (index/leaf/README/guide) cannot drift here.
6. **F16**: `check-plan.mjs` reads the fast-lane slug from `PSTACK_FAST_LANE`
   (default `grok-4.6-fast-xhigh`) instead of a hardcoded literal; the
   multi-phase-plan template documents the override.
7. **F10-F12**: `worktree-audit.sh` detects GNU vs BSD `stat`/`date` at runtime
   (`stat_mtime`/`date_epoch` helpers) and no longer truncates worktree paths
   containing spaces (verified on GNU/Linux; unchanged behavior on macOS).
8. **F-publish**: `automations/benny/` and `skills/make-bot-ui/` excluded —
   the hermes install scanner blocks packages whose scan verdict is
   "dangerous" (benny copy-instructions + make-bot-ui Tailscale), --force
   cannot override, and both are Phase-4/what-not-to-port items anyway.
   The make-bot-ui slot is filled by hermes-native `skills/hermesbot/`
   (injected from `tools/assets/hermesbot/SKILL.md`): a control-surface
   skill on the hermes gateway's own webhook stack (`X-Webhook-Signature-V2`
   HMAC routes), `hermes send`, and `hermes peer` — no Tailscale, no
   third-party bot runtime. Three localhost endpoint literals in the
   feature-map example docs are also neutralized under this gate
   (install-scanner network findings).
9. **G1**: a delegation escape hatch added to the Feature playbook and the
   poteto-mode Subagents section — surgical, fully-specified edits to files
   already resident in context may be implemented in-thread, provided a leaf
   delegate reviews the diff (review separation preserved; fixes the
   deviation observed in the first live usage run).

10. **T8/T9/T10**: hermes-native discovery + factual fixes — `setup-pstack`
    writes `config/models.json` (package-local model panel, 18 roles) instead
    of a Cursor rule; why/reflect/recall/show-me-your-work discovery sections
    query `session_search` over hermes' session store; reviewer prompts use
    hermes file-tool names.
11. **Phase-2A (T6)**: delegation vocabulary translated package-wide — Cursor's
    spawn-parameter and ask-user tool vocabulary becomes hermes equivalents
    (`delegate_task` with role `leaf`, `clarify`), and its background/cloud
    execution flags become hermes execution semantics (background execution,
    local execution); combined fragments are collapsed so no doubled phrasing
    survives, and the stale unbackticked Cursor leftovers ("omit Task
    `model`", "Task subagent") are reworded.
12. **T11**: hardcoded-path cleanup — `.cursor` rules/projects/skills paths,
    `agent-transcripts` recipes, and `/tmp` scratch dirs mapped to hermes
    equivalents or platform-neutral phrasing; `worktree-audit.sh` keeps its
    Cursor transcript check behind an annotated graceful skip.
13. **T12**: Cursor's built-in `/loop` scheduler and `/goal` primitive are
    reworded to hermes-native mechanisms — gateway cron jobs (`hermes cron`)
    as the wake/scheduler across the autopilot, babysit, bug-fix,
    multi-phase, shipping, and verification playbooks, and an armed
    `goal.md` beside the plan in the agent store that every tick re-reads.
    The plan checker (`check-plan.mjs`) accepts the new `goal.md` marker.

Beyond these passes, upstream text is unchanged; the machine-generated fix
register in `.build-provenance.txt` (regenerated on every build) is the
authoritative delta record.

## Known limitations (documented, not drift)

- External prerequisites, not shipped skills: `deslop`, `control-ui`, and
  `control-cli` are named by several playbooks but live outside this package.
  Skip a step that needs them only when the change does not demand that lane;
  when the change touches a live surface and the tooling is absent, record the
  verdict as incomplete instead of clean — a verdict without its demanded live
  lane is not clean.
  (`no-comments`, `unslop`, and `technical-writing` do ship here.)
- Frontier management (`orch frontier set`) supports `--provider auto`,
  `graphite`, and `github`. `auto` tries Graphite first, then GitHub-native
  PR base/head branch topology. GitLab is reserved in the CLI/API but not
  implemented yet.
- Stack topology requires Graphite (`gt`): the stacker is the only topology
  writer, and restacks, stack surgery, and the stack-aware merge queue have
  no hermes equivalent. Retargeting through the stacker requires `gt`, but
  ordinary base changes to an existing child or the bottom PR go through
  the resolved forge (`origin pr edit` / `gh pr edit`) and need no stacker.
  Platform-native follow-ups, per platform: GitHub stacked pull requests
  (public preview) with the official `gh stack` extension — including its
  native restack — are a genuine alternative for the stack operations above.
  GitLab's stacked merge requests model the dependencies and its merge trains
  provide ordered landing, but neither restacks the stack nor performs stack
  surgery — for those, Graphite remains required until the orch scripts gain
  a GitLab forge backend.
- Optional MCP servers: the `why` skill's source investigators (and other
  research skills) draw on MCP-backed sources — issue trackers, chat,
  observability, docs. This package ships no `mcp.json`; configure the servers
  you have and the skills adapt to the session's tool catalog, recording
  unreachable sources as null findings rather than inventing a tool.

---

Adapted for Hermes Agent plugin compatibility from github.com/cursor/plugins pstack.
