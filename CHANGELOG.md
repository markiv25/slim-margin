# Changelog

All notable changes to `slim-margin` will be documented in this file.

## [0.1.0] - 2026-04-20

### Added
- Initial release as a Claude Code marketplace plugin.
- `slim-margin` skill: budget discipline workflow + model-tier routing + output compression orchestration.
- Hard dependencies on three upstream skills:
  - **graphify** (safishamsi/graphify) for codebase knowledge graph.
  - **claude-mem** (thedotmack/claude-mem) for persistent session memory.
  - **caveman** (JuliusBrussee/caveman) for output compression.
- The skill fails closed when any dependency is missing, refusing to degrade to manual reads.
- `marketplace.json` so the repo is installable via `/plugin marketplace add`.
- `plugin.json` manifest for the single plugin.
- GitHub Actions workflow that validates the manifests and packages a `.skill` file on tag push.
