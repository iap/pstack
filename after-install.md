# pstack installed

1. `hermes gateway restart`
2. Run the `setup-pstack` skill once — it writes `config/models.json`
   (the model panel the workflow skills read).
3. Load `poteto-mode` and hand it a real task — it classifies the work and
   routes to the right playbook.

Skills are opt-in on the portable path: load one with
`skill_view agent-plugin-pstack-<digest>:<skill>`, or add this package's
`skills/` dir to `skills.external_dirs` in the hermes config for
`/<name>` slash commands. Update anytime with `hermes plugins update pstack`.
Docs: <https://github.com/iap/pstack#readme>
