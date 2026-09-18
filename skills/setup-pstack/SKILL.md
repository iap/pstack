---
name: setup-pstack
description: Configure which models pstack uses per role. Detects your available models and writes config/models.json that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `config/models.json` in this plugin's directory (next to plugin.json); it sets pstack's model per role. poteto-mode reads it and falls back to `inherit-parent` (the parent chat model) when a role is absent, so this is an override layer, not a requirement.

## Steps

### 1. Detect available models

Enumerate the model slugs available in this session (the configured providers' catalog); that is the dependable source. If you cannot detect any, ask the user via `clarify` to paste the slugs they have access to. Never write a real slug you have not confirmed is available. `inherit-parent` is always valid even though it is not a detected slug (hermes has no Cursor-style `auto` selector; the parent chat model IS the inherit-parent semantic).

### 2. Load current state

The default role-to-model mapping is the shape shown in step 5 below. If `config/models.json` already exists, read it and treat its values as the current choices. Otherwise start from those defaults.

### 3. Map and confirm

Show every role with its current model, marking any real slug not in the detected set as needing a choice. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` (this role runs on the parent chat model) as the options. Prefer clarify over free text. For panel roles (how critics, arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real slug written must be in the detected set; `inherit-parent` always passes. If a chosen real slug is not available, stop and ask again. A rule pointing at a model the user cannot use breaks every delegation that reads it.

### 5. Write the config

Write `config/models.json` in this plugin's directory with one entry per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```json
{
  "roles": {
    "feature, refactoring": "inherit-parent",
    "bug-fix": "inherit-parent",
    "perf-issue": "inherit-parent",
    "hillclimb": "inherit-parent",
    "judgment and prose": "inherit-parent",
    "hardest tasks": "inherit-parent",
    "how explorer": "inherit-parent",
    "how explainer": "inherit-parent",
    "how critics": ["inherit-parent"],
    "why investigators": "inherit-parent",
    "why synthesizer": "inherit-parent",
    "reflect tooling": "inherit-parent",
    "reflect judgment, divergent, synthesizer": "inherit-parent",
    "arena runners": ["inherit-parent"],
    "arena cross-judge pool": ["inherit-parent"],
    "swarm workers": "inherit-parent",
    "architect runners": ["inherit-parent"],
    "interrogate reviewers": ["inherit-parent"]
  }
}
```
Panel roles (how critics, arena runners, arena cross-judge pool, architect runners, interrogate reviewers) take an ARRAY; one subagent runs per entry, so the list length sets the count. `swarm workers` is the default for every worker unless a race assigns another model per arm.

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, load the create-verification-skill skill via skill_view (it ships in this plugin) and follow it. On no, move on without pushing.
