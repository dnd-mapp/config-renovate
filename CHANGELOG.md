# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- The preset. It schedules updates weekly on Monday morning, groups the non-breaking updates per environment, and merges them once CI passes. Breaking updates get their own pull request and wait for an approval.
- A custom manager that updates the Node.js and pnpm versions in `devEngines`.

[Unreleased]: https://github.com/dnd-mapp/renovate-config/commits/main
