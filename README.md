# token-budget-tools

A Claude Code plugin marketplace containing skills for **token-efficient work** on data engineering and architecture tasks.

Currently ships one plugin: [`token-budget-discipline`](./plugins/token-budget-discipline).

---

## What's in here

### `token-budget-discipline`

Enforces **token-budget discipline** and **smart model-tier routing** (Opus / Sonnet / Haiku) on long, complex, or codebase-heavy tasks.

On big tasks — designing pipelines, refactoring across files, reviewing architecture — Claude's default instinct is to read source files directly and burn the context window on comprehension. By the time it has "understood" everything, the budget is gone and the actual work never happens. This skill forces:

1. **Ask for the budget** before doing anything.
2. **Pre-digest the codebase** with cheaper, purpose-built skills (`/caveman`, `/graphify`, `/claude-mem`) *before* opening raw source.
3. **Route by tier**: Haiku does mechanical digestion, Sonnet synthesizes, Opus only gets called in for the hard judgment calls — with distilled context, never raw files.
4. **Track token usage** at checkpoints and stop to reconfirm before blowing past 80%.

The budget constraint becomes a forcing function for smarter orchestration, not an excuse to underperform.

Full skill content: [`plugins/token-budget-discipline/skills/token-budget-discipline/SKILL.md`](./plugins/token-budget-discipline/skills/token-budget-discipline/SKILL.md)

---

## Install

### In Claude Code (recommended)

Add this repository as a marketplace, then install the plugin:

```bash
/plugin marketplace add <your-username>/token-budget-discipline
/plugin install token-budget-discipline@token-budget-tools
```

Or from the CLI:

```bash
claude plugin marketplace add <your-username>/token-budget-discipline
claude plugin install token-budget-discipline@token-budget-tools
```

### In Claude.ai (as a raw skill)

1. Download `token-budget-discipline.skill` from the [Releases](../../releases) page.
2. Settings → Capabilities → Skills → Upload.

The GitHub Actions workflow in this repo auto-builds the `.skill` file whenever you tag a release (see below).

### Via the Claude API

Upload the `SKILL.md` via the Skills API. See [docs.claude.com](https://docs.claude.com/en/api/overview).

---

## Repo layout

```
token-budget-discipline/                       (marketplace repo root)
├── .claude-plugin/
│   └── marketplace.json                       ← marketplace catalog
├── plugins/
│   └── token-budget-discipline/               ← the plugin
│       ├── .claude-plugin/
│       │   └── plugin.json                    ← plugin manifest
│       └── skills/
│           └── token-budget-discipline/
│               └── SKILL.md                   ← the skill itself
├── .github/workflows/
│   └── package.yml                            ← auto-builds .skill on tag push
├── README.md
├── LICENSE
├── CHANGELOG.md
└── .gitignore
```

This structure follows the [Claude Code marketplace spec](https://code.claude.com/docs/en/plugin-marketplaces), so one repo serves three audiences: Claude Code users (via `/plugin install`), Claude.ai users (via the `.skill` file on Releases), and API users (who can upload the `SKILL.md` directly).

---

## Dependencies

The skill references three other skills by name:

- `/caveman` — terse file/module summaries
- `/graphify` — structural understanding, dependency graphs
- `/claude-mem` — memory of prior sessions on the same codebase

If they're not installed, the skill explicitly tells the user and asks how to proceed — it does not silently fall back to reading raw source.

---

## Cutting a release

```bash
git tag v0.1.0
git push origin v0.1.0
```

The GitHub Action zips `plugins/token-budget-discipline/skills/token-budget-discipline/` into `token-budget-discipline.skill` and attaches it to the GitHub release.

---

## Listing in the official Anthropic marketplace

This repo is structured so it *could* later be submitted to [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official). That's a separate review process — start here, build usage, then submit.

---

## Contributing

Issues and PRs welcome. The skill is opinionated — open an issue before a large PR so we can discuss direction.

## License

MIT. See [LICENSE](LICENSE).
