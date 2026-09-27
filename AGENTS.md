# Agent instructions

## Project

This repository is the shared Renovate preset `dnd-mapp/renovate-config`. The preset lives in `default.json`, and every D&D Mapp repository extends it by tag, as `github>dnd-mapp/renovate-config#vX.Y.Z`.

- Give every entry in `packageRules` and `customManagers` a `description` that states what the rule does.
- Consumers merge minor and patch releases of the preset without review. Release a change as major when a maintainer should see its effect first, as `CONTRIBUTING.md` lists under "Changelog and versioning".
- Keep `platformAutomerge` at `false`. GitHub's native auto-merge ignores the ruleset bypass that lets Renovate merge without an approval, so Renovate must merge through the API.
- Decide whether an update is breaking by `matchUpdateTypes` plus the `matchJsonata` condition that treats a 0.x minor as breaking. `matchIsBreaking` misreports GitHub Actions majors. Keep the condition identical in every rule that uses it.
- Update the README in the same commit when you change a group, the schedule, or what merges automatically.
- Run `format-check`, `lint-md`, `validate`, and `actionlint` before you commit.
