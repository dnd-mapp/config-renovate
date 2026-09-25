# Agent instructions

## Project

This repository is the shared Renovate preset `dnd-mapp/renovate-config`. The preset lives in `default.json`, and every D&D Mapp repository extends it by tag, as `github>dnd-mapp/renovate-config#vX.Y.Z`. Read `CONTRIBUTING.md` for the layout, the checks, the release steps, and the commit and branch conventions.

- Give every entry in `packageRules` and `customManagers` a `description` that states what the rule does.
- Consumers merge minor and patch releases of the preset without review. Release a change as major when a maintainer should see its effect first, as `CONTRIBUTING.md` lists under "Changelog and versioning".
- Keep `platformAutomerge` at `false`. GitHub's native auto-merge ignores the ruleset bypass that lets Renovate merge without an approval, so Renovate must merge through the API.
- Decide whether an update is breaking by `matchUpdateTypes` plus the `matchJsonata` condition that treats a 0.x minor as breaking. `matchIsBreaking` misreports GitHub Actions majors. Keep the condition identical in every rule that uses it.
- Update the README in the same commit when you change a group, the schedule, or what merges automatically.
- Run `format-check`, `lint-md`, `validate`, and `actionlint` before you commit.

## Writing style

- Never hard wrap prose. Write each paragraph or list item on a single line and let the editor wrap it.
- Use US spelling only, for example "color", "behavior", and "initialize".
- Keep every sentence at or under 40 words.
- Pretty print Markdown tables so the columns line up in the source.
- Give every separator line alignment markers (`:---`, `:---:`, or `---:`).
- Carry the separator line from edge to edge of each column, with no spaces between the pipes and the dashes.

Example:

| Option   | Default  | Description               |
|:---------|:---------|:--------------------------|
| `strict` | `true`   | Enables all strict checks |
| `target` | `es2025` | Emitted language version  |

After creating or updating a file that contains prose, including Markdown files, do reading passes over it until every rule above is satisfied. Fix any violation you find, then read the file again.

## Pull requests

Turn on auto-merge for every pull request you open, so it merges as soon as it is approved and the checks pass.

1. Open the pull request with `gh pr create`.
2. Run `gh pr merge <number> --auto --merge` on it. A merge commit is the only merge method the repository allows.

- A draft cannot have auto-merge. Mark it ready with `gh pr ready <number>` first, then run the command above.
- When the user asks to keep a pull request open, leave auto-merge off. If it is already on, turn it off with `gh pr merge <number> --disable-auto`.
- Release pull requests (`chore: release X.Y.Z`) get auto-merge too. The release starts only when the maintainer pushes the `vX.Y.Z` tag on the merge commit by hand, as `CONTRIBUTING.md` describes.
