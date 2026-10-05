# dnd-mapp/renovate-config

[![push main](https://github.com/dnd-mapp/renovate-config/actions/workflows/push-main.yaml/badge.svg?branch=main)](https://github.com/dnd-mapp/renovate-config/actions/workflows/push-main.yaml)
[![license](https://img.shields.io/github/license/dnd-mapp/renovate-config)](LICENSE)

Shared [Renovate](https://docs.renovatebot.com/) preset that groups updates weekly and merges the non-breaking ones automatically.

Every D&D Mapp repository extends this preset, so they all get the same update schedule, groups, and automerge policy. Renovate opens at most one pull request per environment each week, and merges minor and patch updates on its own once CI passes. Major updates wait for a maintainer.

## Usage

Add a `renovate.json` to the root of the repository that extends a released tag of the preset.

```json
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": ["github>dnd-mapp/renovate-config#v1.0.0"]
}
```

Renovate keeps the tag up to date. Minor and patch releases of the preset merge automatically, and a major release waits for an approval like any other major update.

## Requirements

- The Renovate GitHub App has access to the repository.
- CI runs on `pull_request`, and the ruleset of the default branch requires it as a status check. Renovate merges only once that check passes.
- The ruleset allows merge commits, because Renovate merges with a merge commit.
- The rule that requires an approval sits in a ruleset of its own, with the Renovate app as a bypass actor for pull requests only. See [Rulesets](#rulesets).
- `pnpm-workspace.yaml` sets `minimumReleaseAge` to three days (`4320` minutes), because lock file maintenance relies on pnpm to hold back new releases. See [Schedule](#schedule).

## Schedule

All times are in `Europe/Amsterdam`.

| Updates                                          | When                                            |
|:-------------------------------------------------|:------------------------------------------------|
| Dependencies, actions, images, Node.js, and pnpm | Mondays from 00:00 to 05:59                     |
| Lock file maintenance                            | The first day of each month from 00:00 to 05:59 |
| Non-breaking updates of `@dnd-mapp/*` packages   | At any time                                     |
| Updates of this preset                           | At any time                                     |

A new release must be at least three days old before Renovate proposes it. Packages, actions, and presets from D&D Mapp are exempt, so a new release of a shared config reaches the other repositories right away. D&D Mapp actions still follow the Monday schedule with the other GitHub Actions.

Lock file maintenance is exempt as well. A refreshed lock file carries no release dates, so Renovate would hold its pull request forever. The `minimumReleaseAge` setting of pnpm in `pnpm-workspace.yaml` still keeps releases younger than three days out of the lock file.

The [Dependency Dashboard](https://docs.renovatebot.com/key-concepts/dashboard/) issue lists every pending update. Tick an update there to have Renovate open its pull request outside the schedule.

## Groups

Renovate groups the non-breaking updates of each environment into one pull request. The branch of each group is `renovate/<slug>`.

| Group             | Slug              | Contains                                                                         |
|:------------------|:------------------|:---------------------------------------------------------------------------------|
| npm packages      | `npm`             | npm dependencies, except the ones in the groups below                            |
| GitHub Actions    | `github-actions`  | Actions and container images in workflows and composite actions                  |
| Docker images     | `docker`          | Images in Dockerfiles and Compose files                                          |
| Node.js           | `node`            | The Node.js version in `devEngines`, the `engines.node` range, and `@types/node` |
| pnpm              | `package-manager` | The pnpm version in `devEngines`                                                 |
| dnd-mapp packages | `dnd-mapp`        | `@dnd-mapp/*` npm packages                                                       |

A breaking update gets a pull request of its own, outside the groups. An update is breaking when it is a major update, or a minor update of a `0.x` version. The Node.js and pnpm groups are the exception, because their parts must move together. Their major updates get a separate group pull request, on the branch `renovate/major-<slug>`.

Lock file maintenance refreshes `pnpm-lock.yaml` on the branch `renovate/lock-file-maintenance`.

## Automerge

Renovate merges every non-breaking update once CI passes, without an approval. That covers the groups above, digest updates, and lock file maintenance. Renovate merges through the GitHub API with a merge commit, not with GitHub's auto-merge. GitHub's auto-merge does not honor a ruleset bypass.

A breaking update needs an approval from a code owner. Review it, approve it, and turn on auto-merge with `gh pr merge <number> --auto --merge`.

## Commits and pull requests

Renovate commits through the GitHub API, so GitHub signs every commit. Commit messages and pull request titles follow Conventional Commits without a scope, so they pass `@dnd-mapp/config-commitlint`.

| Updates         | Type    | Example                               |
|:----------------|:--------|:--------------------------------------|
| GitHub Actions  | `ci`    | `ci: update GitHub Actions`           |
| Everything else | `build` | `build: update dependency vite to v9` |

## Versions and ranges

- Actions are pinned to a commit SHA, with the version in a comment.
- Ranges keep their operator and move their lower bound, so `~1.2.3` becomes `~1.2.4`.
- Peer dependency ranges widen instead, so `^6` becomes `^6 || ^7`.
- The `engines` range changes only when the new version falls outside it.
- TypeScript stays below 7. TypeScript 7 drops the stable compiler API, which `typescript-eslint` and `prettier-plugin-organize-imports` need. Lift the hold once both support TypeScript 7.

## Node.js

The repositories stay on the active LTS line of Node.js. Renovate treats a Node.js major as stable only once its LTS phase starts, and the same goes for `@types/node`. A new major therefore shows up only when it becomes an LTS release.

That update lands in the `renovate/major-node` pull request. It moves the Node.js version in `devEngines`, `@types/node`, and the `engines.node` range, for example from `^24` to `^26`. In a published package, a new `engines.node` range is a breaking change, so add a changelog entry before you approve the pull request.

## Known gaps

- Renovate does not update `devEngines` yet ([renovatebot/renovate#38067](https://github.com/renovatebot/renovate/issues/38067)). The preset covers it with a custom manager until Renovate does.
- The custom manager does not touch `pnpm-lock.yaml`, which records the pnpm version under `packageManagerDependencies`. After a pnpm update, that section is stale until the next npm or lock file maintenance pull request refreshes it. `pnpm install --frozen-lockfile` still passes in the meantime.

## Rulesets

The default branch needs a code owner's approval for every pull request. A GitHub App cannot be a code owner, so Renovate could never merge on its own. The fix is to split the ruleset in two.

| Ruleset                  | Rules                                                                                                            | Bypass                                   |
|:-------------------------|:-----------------------------------------------------------------------------------------------------------------|:-----------------------------------------|
| `Default branch`         | Restrict deletions, block force pushes, require signed commits, the `CI` status check, and code scanning results | None                                     |
| `Default branch reviews` | Require a pull request with an approval from a code owner on the latest push, and allow merge commits only       | The Renovate app, for pull requests only |

The bypass lets Renovate merge its own pull requests without an approval. It never skips the status checks, because those sit in the other ruleset. Renovate merges only the updates that the preset marks for automerge.

## Changelog

Notable changes for consumers of this preset are listed in the [changelog](CHANGELOG.md).

## Contributing

Contributions are welcome. See the [contributing guide](CONTRIBUTING.md) for details.

## License

[MIT](LICENSE) © D&D Mapp
