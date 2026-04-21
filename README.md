# slim-margin

> Run Claude on a slim token margin. Spend the budget on decisions, not comprehension.

A Claude Code plugin that enforces **token-budget discipline** and **smart model-tier routing** (Opus / Sonnet / Haiku) on long, complex, or codebase-heavy tasks.

This repo is both the plugin and its marketplace — one `/plugin marketplace add` away from being installable.

---

## ⚠️ Required prerequisites

`slim-margin` is an **orchestrator**. It does not do codebase digestion, session memory, or output compression itself — it coordinates three other skills that do. **You must install all three before installing `slim-margin`**, or it will refuse to run.

| Skill | Repo | What it does |
| --- | --- | --- |
| **graphify** | [safishamsi/graphify](https://github.com/safishamsi/graphify) | Builds a persistent knowledge graph of your codebase. Claude reads the graph instead of grepping raw files. |
| **claude-mem** | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | Captures and compresses session history so future sessions start with relevant context already loaded. |
| **caveman** | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | Strips filler and hedging from Claude's output. ~75% fewer output tokens, same technical content. |

### Install the prerequisites

```bash
# 1. graphify (requires Python 3.10+)
pip install graphifyy
graphify install
# then, from inside your project:
graphify claude install

# 2. claude-mem (run inside a Claude Code session)
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem

# 3. caveman (run inside a Claude Code session)
claude plugin marketplace add JuliusBrussee/caveman
claude plugin install caveman@caveman
```

Restart Claude Code after installing each one. Verify they're active before installing `slim-margin`.

**Why hard deps?** The skill's whole premise is that you don't burn tokens doing work these three tools already do. If they're not there, the skill degrading to manual reads would defeat its own purpose. It fails closed on purpose.

---

## What `slim-margin` does

On big tasks — designing pipelines, refactoring across files, reviewing architecture — Claude's default instinct is to grep and read source files directly and burn the context window on comprehension. By the time it has "understood" everything, the budget is gone and the actual work never happens. `slim-margin` forces a different pattern:

1. **Check prerequisites.** If graphify / claude-mem / caveman aren't installed, refuse.
2. **Ask for the budget.** No budget, no work.
3. **Pull prior context from claude-mem** before asking or exploring.
4. **Read the graphify report** before opening any raw file.
5. **Route by tier.** Haiku does mechanical classification, Sonnet synthesizes, Opus only gets called in for the hard judgment calls — with distilled context, never raw files.
6. **Keep caveman on** so output doesn't eat the budget wrapper.
7. **Track token usage** at checkpoints and stop to reconfirm before blowing past 80%.

The budget constraint becomes a forcing function for smarter orchestration, not an excuse to underperform.

Full skill text: [`plugins/slim-margin/skills/slim-margin/SKILL.md`](./plugins/slim-margin/skills/slim-margin/SKILL.md)

---

## Install this plugin

**Only after** installing graphify, claude-mem, and caveman as shown above.

### In Claude Code

```bash
/plugin marketplace add markiv25/slim-margin
/plugin install slim-margin@slim-margin
```

### In Claude.ai (as a raw skill)

Download `slim-margin.skill` from the [Releases](../../releases) page and upload via Settings → Capabilities → Skills.

Note: the Claude.ai upload path does not automatically install the three prerequisite skills. The skill will still refuse to run until they're present — on Claude.ai this is a harder lift since the prerequisites are Claude Code plugins. `slim-margin` is most useful in a Claude Code environment.

---

## Repo layout

```
slim-margin/                                   (marketplace repo root)
├── .claude-plugin/
│   └── marketplace.json                       ← marketplace catalog
├── plugins/
│   └── slim-margin/                           ← the plugin
│       ├── .claude-plugin/
│       │   └── plugin.json                    ← plugin manifest
│       └── skills/
│           └── slim-margin/
│               └── SKILL.md                   ← the skill itself
├── .github/workflows/
│   └── package.yml                            ← auto-builds .skill on tag push
├── README.md
├── LICENSE
├── CHANGELOG.md
└── .gitignore
```

---

## Cutting a release

```bash
git tag v0.1.0
git push origin v0.1.0
```

The GitHub Action validates the manifest files, zips the skill into `slim-margin.skill`, and attaches it to the GitHub release.

---

## A note on the name

`slim-margin` because that's what you're running on when the budget matters. Slim margins demand discipline: no wasted reads, no wasted tier, no wasted words. If you have abundant context, you don't need this skill. If you're on a slim margin, you do.

---

## Credit

`slim-margin` is a thin orchestrator on top of the hard work of three upstream authors:

- [Julius Brussee](https://github.com/JuliusBrussee) for **caveman**
- [Safi Shamsi](https://github.com/safishamsi) for **graphify**
- [Alex Newman (thedotmack)](https://github.com/thedotmack) for **claude-mem**

If this skill saves you tokens, go star their repos too.

---

## Contributing

Issues and PRs welcome. The skill is deliberately opinionated — open an issue before a large PR so we can discuss direction.

## License

MIT. See [LICENSE](LICENSE).
