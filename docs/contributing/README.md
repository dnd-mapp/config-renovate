# Contributing to dnd-mapp/config-renovate

This page adds the details of `dnd-mapp/config-renovate` to the [shared contributing guide](https://github.com/dnd-mapp/.github/blob/main/CONTRIBUTING.md). Read that guide first.

This preset decides which dependency updates every D&D Mapp repository gets, and which of them merge without review. A mistake here spreads to all of them at once, so keep changes small and deliberate.

## Project layout

| Path                             | Purpose                                                                                     |
|:---------------------------------|:--------------------------------------------------------------------------------------------|
| `default.json`                   | The preset that consumers extend as `github>dnd-mapp/config-renovate#vX.Y.Z`                |
| `renovate.json`                  | The Renovate config of this repository, which extends a released tag of the preset          |
| `.github/actions/ci/action.yaml` | The checks that the pull request, push, and release workflows run                           |
| `.github/workflows/release.yaml` | Verifies a release tag with `dnd-mapp/action-verify-release` and creates the GitHub Release |
| `.github/actionlint.yaml`        | Declares the `ubuntu-26.04` runner label, which actionlint does not know yet                |

## Checks

On top of the [shared checks](https://github.com/dnd-mapp/.github/blob/main/CONTRIBUTING.md#checks), CI runs `validate`. Run it yourself before you open a pull request.

```bash
pnpm run validate
```

## Changing the preset

Give every entry in `packageRules` and `customManagers` a `description` that states what the rule does.

Update the README in the same pull request when you change a group, the schedule, the release age, or what merges automatically.

`pnpm run validate` checks `default.json` and `renovate.json` with `renovate-config-validator --strict`. It catches unknown options and deprecated ones, but it does not show which updates a change produces. For that, do a dry run against a consumer repository.

1. Clone a consumer repository into a scratch folder, so the dry run leaves your working copy alone.
2. Copy `default.json` into the clone as `renovate.json`, and commit it there.
3. Run Renovate from the clone with the command below. Call the binary by its path, not through `npx`, because npm fails on the `devEngines` of the consumer repository. It writes every lookup, update type, and branch name to `report.json`.

```bash
RENOVATE_GITHUB_COM_TOKEN=$(gh auth token) RENOVATE_FORCE='{"schedule":null}' <path-to-this-repository>/node_modules/.bin/renovate --platform=local --dry-run=lookup --report-type=file --report-path=report.json
```

A dry run does not show pull request titles or commit messages. Check those on the first real pull requests after a release.

## Changelog and versioning

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Record every notable change for consumers under `[Unreleased]` in `CHANGELOG.md`, using the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

Consumers pin a tag, and Renovate updates that tag like any other dependency. Minor and patch releases of the preset merge without review, so they take effect in every repository right away. Release a change as major when a maintainer should see its effect first. These changes are breaking:

- Any change that makes more updates merge automatically.
- A change to the schedule or the release age.
- Renaming or removing a group, because its open pull requests close and new ones open under another branch.
- A change that raises what consumers must set up, such as a new required status check or repository setting.

Say why the change is breaking in the changelog entry.

## Releasing

1. Run the [prepare release workflow](../../.github/workflows/prepare-release.yaml) on `main` with the part of the version to bump, for example `gh workflow run prepare-release.yaml -f bump=minor`. It opens the `chore: release X.Y.Z` pull request with auto-merge on.
2. Review and approve the pull request. Once it merges, the `tag` job of the [push workflow](../../.github/workflows/push-main.yaml) creates the annotated tag `vX.Y.Z` on the merge commit.
3. The [release workflow](../../.github/workflows/release.yaml) runs the CI checks, verifies the tag and the changelog, and creates the GitHub Release, which opens a discussion in the Announcements category.
4. Renovate opens a pull request in each consumer repository that moves the tag in `renovate.json`. A major release waits there for an approval.
