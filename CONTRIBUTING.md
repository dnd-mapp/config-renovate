# Contributing

Thank you for your interest in contributing to `dnd-mapp/config-renovate`.

This preset decides which dependency updates every D&D Mapp repository gets, and which of them merge without review. A mistake here spreads to all of them at once, so please keep changes small and deliberate.

## Before you start

Open an [issue](https://github.com/dnd-mapp/config-renovate/issues) to discuss any change beyond a typo fix before you send a pull request. This avoids work on changes that do not fit the goals of the preset.

## Development setup

The required Node and pnpm versions are set in `devEngines` in `package.json`. They are enforced through `engineStrict`, so installing with other versions fails.

Install the dependencies with:

```bash
pnpm install
```

Dependency versions live in the catalogs in `pnpm-workspace.yaml`, which uses `catalogMode: strict`. Add or bump versions there and reference them in `package.json`. Use `catalog:` for the default catalog and a named catalog such as `catalog:prettier` for a group of tools.

Newly published releases are held back for three days through `minimumReleaseAge`. You may need to wait before you can bump to a very recent version.

Install [actionlint](https://github.com/rhysd/actionlint) to lint the workflows locally, for example with `brew install actionlint`. CI runs the version that `.github/actions/ci/action.yaml` pins.

## Git hooks

[Lefthook](https://lefthook.dev/) installs the Git hooks when you run `pnpm install`. The hooks are defined in `lefthook.yaml`. `pnpm-workspace.yaml` turns off the side-effects cache of pnpm, because a cached build of lefthook skips the script that installs the hooks. If the hooks are still missing, install them with `pnpm exec lefthook install`.

| Hook         | Runs                                  | On                        |
|:-------------|:--------------------------------------|:--------------------------|
| `pre-commit` | Prettier and markdownlint-cli2 checks | The staged files          |
| `commit-msg` | commitlint                            | The message of the commit |

The pre-commit hooks only check files. Run `pnpm run format` to fix formatting issues and stage the result.

## Project layout

| Path                             | Purpose                                                                                     |
|:---------------------------------|:--------------------------------------------------------------------------------------------|
| `default.json`                   | The preset that consumers extend as `github>dnd-mapp/config-renovate#vX.Y.Z`                |
| `renovate.json`                  | The Renovate config of this repository, which extends a released tag of the preset          |
| `.github/actions/ci/action.yaml` | The checks that the pull request, push, and release workflows run                           |
| `.github/workflows/release.yaml` | Verifies a release tag with `dnd-mapp/action-verify-release` and creates the GitHub Release |
| `.github/actionlint.yaml`        | Declares the `ubuntu-26.04` runner label, which actionlint does not know yet                |

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

## Checking the repository

Check and format the repository with these commands. CI runs `format-check`, `lint-md`, `validate`, and actionlint. Run them yourself before you open a pull request.

```bash
pnpm run format-check
pnpm run format
pnpm run lint-md
pnpm run validate
actionlint
```

## Changelog and versioning

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Record every notable change for consumers under `[Unreleased]` in `CHANGELOG.md`, using the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

Consumers pin a tag, and Renovate updates that tag like any other dependency. Minor and patch releases of the preset merge without review, so they take effect in every repository right away. Release a change as major when a maintainer should see its effect first. These changes are breaking:

- Any change that makes more updates merge automatically.
- A change to the schedule or the release age.
- Renaming or removing a group, because its open pull requests close and new ones open under another branch.
- A change that raises what consumers must set up, such as a new required status check or repository setting.

Say why the change is breaking in the changelog entry.

## Releasing

1. Run the [prepare release workflow](.github/workflows/prepare-release.yaml) on `main` with the part of the version to bump, for example `gh workflow run prepare-release.yaml -f bump=minor`. It opens the `chore: release X.Y.Z` pull request with auto-merge on.
2. Review and approve the pull request. Once it merges, the `tag` job of the [push workflow](.github/workflows/push-main.yaml) creates the annotated tag `vX.Y.Z` on the merge commit.
3. The [release workflow](.github/workflows/release.yaml) runs the CI checks, verifies the tag and the changelog, and creates the GitHub Release.
4. Renovate opens a pull request in each consumer repository that moves the tag in `renovate.json`. A major release waits there for an approval.

## Code style

Follow the rules in `.editorconfig`.

- Use UTF-8 and LF line endings.
- Indent with 4 spaces, or 2 spaces in `package.json` and `pnpm-*.yaml`.
- End every file with a newline and trim trailing whitespace.

Follow these rules for prose, including Markdown files.

- Never hard wrap prose. Write each paragraph or list item on a single line.
- Use US spelling, for example "color" and "behavior".
- Keep every sentence at or under 40 words.
- Pretty print Markdown tables so the columns line up, with alignment markers on every separator line.

## Branches

Create a branch from `main` for each change. Name it `<type>/<short-description>` in lowercase with hyphens between words, for example `feat/group-docker-images` or `fix/node-schedule`.

Use the same types as for commits.

## Commits

Write commit messages that follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

```text
<type>(<optional scope>): <description>
```

Use one of these types.

| Type       | Use for                                              |
|:-----------|:-----------------------------------------------------|
| `feat`     | A new rule, group, or manager in the preset          |
| `fix`      | A correction to existing behavior                    |
| `docs`     | Changes to documentation only                        |
| `refactor` | Changes that do not alter the behavior of the preset |
| `ci`       | Changes to the workflows of this repository          |
| `build`    | Changes to dependencies or tooling                   |
| `chore`    | Other maintenance that does not fit above            |

Write the description in the imperative mood, such as "group the Docker images". Mark a breaking change with `!` after the type or scope, and add a `BREAKING CHANGE:` footer that explains what consumers must do.

## Pull requests

- Keep each pull request to one change.
- Link the issue it addresses.
- Update the changelog and README in the same pull request.
- Use a title that follows the commit convention.
- If you have write access, turn on auto-merge once the pull request is open, with `gh pr merge <number> --auto --merge` or the "Enable auto-merge" button. It then merges as soon as it is approved and the checks pass.
- If auto-merge is off, the author merges the pull request once it is approved and the checks pass. A maintainer merges pull requests opened by a contributor without write access.
- Renovate merges its own minor and patch pull requests once the checks pass. A maintainer approves a major update from Renovate and turns on auto-merge for it.
- Update the branch when it falls behind `main`, because auto-merge waits until the branch is up to date. The update dismisses the approval, so the pull request needs a new review.

## License

By contributing, you agree that your contributions are licensed under the [MIT license](LICENSE).
