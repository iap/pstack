# pstack

*The pstack agent method — go fast by going deep first — packaged for the
Hermes Agent portable plugin path. Unofficial port of
<https://github.com/cursor/plugins/tree/main/pstack> (MIT, © Lauren Tan);
package keeps the upstream identity pstack v0.14.8.*

## Install

```sh
hermes plugins install iap/pstack --enable
```

Legacy route: `hermes plugins install iap/pstack-hermes/pstack --enable`
(the dev repo's subdir; works, but cannot update in-place).

## First run

1. `hermes plugins doctor pstack --ci` — manifest parses, all skills discover.
2. Load the `setup-pstack` skill once — it writes `config/models.json`
   (the 18-role model panel the workflow skills read).
3. Load `poteto-mode` and give it a real task — it classifies the work and
   routes to the right playbook. You do not pick playbooks yourself.

Skills are opt-in on the portable path: load one with
`skill_view` id `agent-plugin-pstack-<digest>:<skill-name>` (digest = first
8 hex of the sha256 of the plugin key, stable while the install dir is named
`pstack`). For `/<name>` slash commands instead, add this package's `skills`
dir to `skills.external_dirs` in the hermes `config.yaml` — trade-off: skills
enter the prompt index with 60-char descriptions and lose the namespace.

## Update

```sh
hermes plugins update pstack
```

Root installs keep `.git`: this pulls the latest published build and
autostashes uncommitted local skill edits. Subdir installs strip `.git` —
reinstall to update (local edits are wiped; copy them out first).

## Customizing skills

- **Tweak a line or two** — edit the installed `SKILL.md`, restart the
  gateway, keep the edit uncommitted (updates autostash it).
- **Own several changed skills** — fork `iap/pstack`, commit, install from
  your fork (`hermes plugins install <you>/pstack --enable`).
- **Add your own new skills alongside** — a skill tap:
  `hermes skills tap add <you>/skills-repo`, then
  `hermes skills install <you>/skills-repo/<skill>`.
- Keep the hermes vocabulary when editing (`delegate_task`, `clarify`,
  `session_search`, `hermes cron`); Cursor primitives will mislead the agent.

Problems: port behaviour → <https://github.com/iap/pstack-hermes/issues>;
a hermes defect that reproduces with the plugin disabled →
`NousResearch/hermes-agent`; an upstream pstack bug → `cursor/plugins`.

## What hermes loads

- **45 skills** (24 workflow/mode +
  21 `principle-*`), single-level `skills/<dir>/SKILL.md`,
  discovered via the root `plugin.json` (agent-plugins-v1 manifest,
  9 whitelisted fields).
- `skills/hermesbot/` — hermes-native control-surface skill on the gateway
  webhook (`X-Webhook-Signature-V2` HMAC) plus `hermes send` / `hermes peer`;
  fills the excluded `make-bot-ui` slot.
- Skill scripts (e.g. `skills/poteto-mode/scripts/`) ship verbatim; the loader
  never executes them, but skills instruct the agent to run them (`bun`, `gh`).

Not shipped: `automations/benny/` and `skills/make-bot-ui/` (install-scanner
verdicts — persistence / privilege escalation; `--force` cannot override) and
upstream's `docs/guide/` (read it upstream for the guided tour). `agents/` and
`.cursor-plugin/` are inert on hermes — kept so the tree still dual-loads as a
Cursor plugin structurally; for real Cursor-side work install upstream pstack.

## Differences from upstream

| Pass | What changed |
|---|---|
| manifest | root `plugin.json` injected: exact agent-plugins-v1 `$schema`, 9-field whitelist |
| frontmatter | `poteto-mode` name fixed to kebab-case (loader requirement) |
| R1 | poteto-mode principles index regenerated from the 21 leaves |
| F16 | fast-lane slug overridable via `PSTACK_FAST_LANE` |
| F10–F12 | `worktree-audit.sh` portable (GNU/BSD), space-safe |
| F-publish | `benny` + `make-bot-ui` excluded (scanner verdicts); `hermesbot` fills the slot; 3 localhost literals neutralized |
| T8/T9/T10 | `setup-pstack` writes `config/models.json`; discovery via `session_search`; hermes tool names |
| Phase-2A | delegation vocabulary → `delegate_task` / `clarify` package-wide |
| T11 | hardcoded Cursor paths → hermes equivalents |
| T12 | `/loop` → `hermes cron` wake; `/goal` → `goal.md` in the agent store |
| T13 | residual vendor coupling reworded (`cursor-team-kit`, cloud fleet, `gt`) |
| T14 | stacker-role / topology re-pointed (issue #33) |
| T15 | `orch frontier set` provider selection: `auto` / `graphite` / `github` |
| G1 | delegation escape hatch: in-thread surgical edits with mandatory delegate review |

Text is otherwise unchanged; the machine-generated fix register in
`.build-provenance.txt` is the authoritative per-build record, and the full
ledger with reviewer checks lives in the dev repo
(`iap/pstack-hermes`, `docs/ADAPTATIONS.md`).

## Known limitations (documented, not drift)

- `deslop`, `control-ui`, `control-cli` are named by playbooks but ship
  outside this package; a verdict without its demanded live lane is
  incomplete, not clean. (`no-comments`, `unslop`, `technical-writing` ship.)
- `orch frontier set` providers: `auto` (Graphite first, then GitHub-native
  PR base/head topology), `graphite`, `github`. GitLab reserved, not
  implemented.
- Stack topology needs Graphite (`gt`): restacks, stack surgery, and the
  stack-aware merge queue have no hermes equivalent. GitHub-native stacked
  PRs (`gh stack`) cover some stack operations; ordinary base changes go
  through the forge and need no stacker.
- No `mcp.json` shipped: research skills use whichever MCP servers the
  session configures; unreachable sources are recorded as null findings,
  never invented.

---

Adapted for Hermes Agent plugin compatibility from github.com/cursor/plugins pstack.
