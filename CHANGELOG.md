# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-09-28

### Added

- A rule that holds TypeScript below 7. TypeScript 7 drops the stable compiler API, which `typescript-eslint` and `prettier-plugin-organize-imports` need. Renovate closes the open TypeScript 7 pull requests once repositories use this release.

## [1.0.0] - 2026-09-25

### Added

- The preset. It schedules updates weekly on Monday morning, groups the non-breaking updates per environment, and merges them once CI passes. Breaking updates get their own pull request and wait for an approval.
- A custom manager that updates the Node.js and pnpm versions in `devEngines`.

[Unreleased]: https://github.com/dnd-mapp/renovate-config/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/dnd-mapp/renovate-config/releases/tag/v1.1.0
[1.0.0]: https://github.com/dnd-mapp/renovate-config/releases/tag/v1.0.0
