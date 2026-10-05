# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.0.0] - 2026-10-05

### Changed

- The repository is renamed from `dnd-mapp/renovate-config` to `dnd-mapp/config-renovate`, to match the other shared config repositories. This is a breaking change, because the rule that applies preset updates at any time now matches only the new name. Change `extends` in `renovate.json` to `github>dnd-mapp/config-renovate#v2.0.0` by hand, since Renovate never renames a preset.

### Fixed

- Lock file maintenance pull requests merge automatically again. Renovate treats an update without a release date as too young, and a refreshed lock file has none, so the `renovate/stability-days` check stayed pending forever. Lock file maintenance now skips the release age. This is a breaking change, because it changes the release age and lets more updates merge automatically. Each repository must set `minimumReleaseAge` in `pnpm-workspace.yaml`, which keeps releases younger than three days out of the lock file.

## [1.1.0] - 2026-09-28

### Added

- A rule that holds TypeScript below 7. TypeScript 7 drops the stable compiler API, which `typescript-eslint` and `prettier-plugin-organize-imports` need. Renovate closes the open TypeScript 7 pull requests once repositories use this release.

## [1.0.0] - 2026-09-25

### Added

- The preset. It schedules updates weekly on Monday morning, groups the non-breaking updates per environment, and merges them once CI passes. Breaking updates get their own pull request and wait for an approval.
- A custom manager that updates the Node.js and pnpm versions in `devEngines`.

[Unreleased]: https://github.com/dnd-mapp/config-renovate/compare/v2.0.0...HEAD
[2.0.0]: https://github.com/dnd-mapp/config-renovate/releases/tag/v2.0.0
[1.1.0]: https://github.com/dnd-mapp/config-renovate/releases/tag/v1.1.0
[1.0.0]: https://github.com/dnd-mapp/config-renovate/releases/tag/v1.0.0
