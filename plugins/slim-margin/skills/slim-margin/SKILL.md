---
name: slim-margin
description: Enforce strict token-budget discipline AND smart model-tier routing (Opus / Sonnet / Haiku) on long, complex, or codebase-heavy tasks. Use this skill whenever the user mentions a token budget, rate limit, context limit, token pressure, "stay under X tokens", or any form of model-switching / cost-optimization concern — and also proactively on any sufficiently large task (multi-file refactors, new pipeline designs, cross-repo work, architecture reviews) where tokens could easily be wasted re-reading code that should have been pre-digested, or where the wrong model tier would be assigned to subtasks. Triggers on keywords like "rate limit", "token budget", "context window", "stay under", "don't blow the budget", "use a cheaper model", "save tokens", "auto-switch models", "Haiku for this", and on any request where scope clearly exceeds casual handling. This skill has HARD DEPENDENCIES on three installed skills — graphify (knowledge graph of the codebase), claude-mem (persistent session memory), and caveman (output compression). Without all three installed, this skill refuses to proceed and instructs the user to install them. When all three are present, the skill orchestrates them: check claude-mem for prior context, use graphify to navigate by structure instead of grepping, keep caveman active to compress output, and route subtasks across Haiku / Sonnet / Opus so cheap mechanical work doesn't burn premium tier budget.
---

# slim-margin

A skill for spending tokens on what matters — planning, decisions, architecture — instead of burning them re-reading code that could be navigated by structure, re-learning context that was already captured last session, or emitting verbose prose no one needed.

## Why this skill exists

On large tasks (designing a new data pipeline, refactoring across files, reviewing architecture), Claude's default instinct is to grep and read source files directly with its default tier to "understand the code." That burns the entire context window on comprehension — and pays the premium-tier price for mechanical reading work a cheap model could have done. By the time Claude has read everything, the budget is gone and the actual work — the design, the plan, the decisions the user actually wants — never happens, or happens with a depleted context.

There are three fixes, and this skill enforces all three:

1. **Budget discipline: prep → confirm → plan.** Use the installed helper skills to pre-digest the codebase and retrieve prior context *before* reading raw source. Confirm the budget with the user. Spend what's left on the thinking that only Claude can do.
2. **Tier discipline: fan out cheap, concentrate expensive.** Haiku handles mechanical digestion. Sonnet synthesizes. Opus only gets called in for the hard judgment calls — and only with distilled context, never raw files.
3. **Output discipline: compress the wrapper.** Keep caveman active so Claude's own explanations don't eat the output side of the budget.

The budget constraint is a **forcing function for smarter orchestration**, not a reason to deliver a worse result.

## Hard dependencies (non-negotiable)

This skill has three hard dependencies. It does NOT work without them. If any of the three is missing, Claude must refuse the task and instruct the user to install what's missing.

| Dependency | What it does | Install |
| --- | --- | --- |
| **graphify** (safishamsi/graphify) | Builds a persistent knowledge graph of the codebase using tree-sitter AST + Leiden clustering. Produces `graphify-out/GRAPH_REPORT.md` with god nodes, communities, and surprising connections. Claude reads the graph report instead of grepping raw files. Reported 49–71× token reduction on mixed corpora. | `pip install graphifyy && graphify install && graphify claude install` |
| **claude-mem** (thedotmack/claude-mem) | Automatic persistent session memory. Captures tool usage across sessions, compresses it via Claude's Agent SDK, and injects relevant context at session start. Query prior history via `mem-search`. | `/plugin marketplace add thedotmack/claude-mem && /plugin install claude-mem` |
| **caveman** (JuliusBrussee/caveman) | Output compression — strips filler, hedging, and pleasantries from Claude's responses. ~75% reduction in output tokens with technical accuracy intact. Activated with `/caveman` (levels: lite, full, ultra). | `claude plugin marketplace add JuliusBrussee/caveman && claude plugin install caveman@caveman` |

### Checking dependencies

At the start of any task that triggers this skill, Claude verifies all three are available before doing anything else.

Signals that a dependency is installed:
- **graphify** — a `graphify-out/` directory exists in the project root, and/or the `CLAUDE.md` file contains a graphify directive, and/or `/graphify` is an available slash command.
- **claude-mem** — context from prior sessions appears near the top of the session, and/or `mem-search` is available as a skill, and/or there's a claude-mem block in the initial context.
- **caveman** — `/caveman` is an available slash command, and/or the session is already in caveman mode (short-form, no articles, no filler).

If ALL THREE are present, proceed to the workflow.

If ANY is missing, STOP. Respond with:

> This skill requires three other skills to be installed first: **graphify** (for codebase pre-digestion), **claude-mem** (for prior session context), and **caveman** (for output compression). Missing: [list]. Please install them — see the repo README for exact install commands — and then come back. I'm not going to try to work around it, because working around these defeats the entire purpose of budget discipline.

Do not attempt to substitute manual reading or grepping for graphify. Do not attempt to recall context manually when claude-mem would have surfaced it. Do not generate verbose output in place of caveman-compressed output. The skill fails closed on purpose.

## The workflow

Follow these steps IN ORDER. Do not skip ahead.

### Step 0: Dependency check

Verify all three hard deps are installed. If not, refuse as described above.

### Step 1: Get the budget

If the user already stated a budget (tokens, "one-shot", "keep it tight", etc.), note it and move on.

If not, ASK before doing anything else:

> "Before I start, what's the token budget for this? I'd like to plan around it so I don't burn the context re-reading code."

Accept any form of answer — a number ("50k"), a vibe ("keep it tight"), or "just do your best" (assume aggressive conservation). Do not proceed without a budget answer.

### Step 2: Pull prior context from claude-mem

claude-mem injects prior session context automatically at session start. Check what's already there before asking or exploring. If the user's current task is continuing or adjacent to past work, `mem-search` can surface specific observations by keyword.

Do NOT re-learn what claude-mem already knows. If a decision, bug, or architecture note from last session is already in context, reference it — don't re-derive it.

### Step 3: Read the graphify report BEFORE any file-level reading

Before using `view` or `Grep` on any source file, read `graphify-out/GRAPH_REPORT.md`. That report surfaces:

- **God nodes** — highest-degree concepts (files, classes, functions) that everything routes through. These are almost always the right starting points for understanding the system.
- **Communities** — clusters of related code found via Leiden clustering on the graph topology. Treat each community as a module-level unit.
- **Surprising connections** — cross-file or cross-modal edges the graph extracted with plain-English "why" annotations. These are where non-obvious coupling lives.

If the project doesn't have a `graphify-out/` directory yet, stop and ask the user to run `/graphify .` first. Don't proceed by grepping — that defeats the skill.

If the graph report doesn't answer a specific question, use the targeted graphify CLI commands (`graphify query`, `graphify path`, `graphify explain`) to get edge-level detail before reading raw files. Reading raw source is a last resort, not a first move.

### Step 4: Plan within the budget

With prior context (claude-mem) and codebase structure (graphify) in hand, produce a plan. The plan is the deliverable for this phase — not code, not implementation. Specifically:

- State what's already known from claude-mem and graphify (1–3 sentences each, not paragraphs).
- State the proposed approach.
- State the open questions — things the digests did NOT answer and that will require reading specific code, running it, or asking the user.
- Estimate the token cost of each open question so the user can choose which to spend the remaining budget on.

### Step 5: Spend the rest with the user's consent

After presenting the plan, STOP. Do not silently start reading source files to resolve open questions. Ask the user which open questions to pursue, given what's left of the budget.

Acceptable phrasings:

> "Here's the plan. Open questions A, B, C are unresolved — A costs ~5k tokens to answer (read `pipeline.py`), B costs ~15k (trace the DAG via graphify query), C costs ~2k (one clarifying question to you). How do you want to spend what's left?"

### Step 6: Track as you go

At natural checkpoints (end of a tool-use burst, before a new section), give a quick budget check:

> "Used roughly ~20k so far, ~30k remaining."

Estimates are fine. If you blow through 80% of the budget, stop and confirm before continuing.

### Step 7: Keep caveman on

Output compression applies throughout. If the user hasn't activated `/caveman` and the task is large, suggest it at the start:

> "I'd like to run in `/caveman full` mode for this — it'll cut my explanation wrapper by ~75% and leave more budget for actual work. OK?"

Default to `/caveman full` for most tasks; escalate to `ultra` for very tight budgets; drop to `lite` or normal if the user needs more explanation (e.g., teaching contexts, user-facing copy).

## Model tier routing (Opus / Sonnet / Haiku)

Budget pressure is not just a reason to use fewer tokens — it is a reason to use the **right tier** for each subtask. The goal is not "use the cheapest model everywhere"; that underperforms. The goal is to **decompose the task so cheap models handle the mechanical work and the expensive model gets the hard thinking with its full attention intact.**

This section applies in any environment where the orchestrator can choose a model per subtask — Claude Code, the Anthropic API with sub-calls, agentic setups, or any harness that dispatches work to different tiers. In a single-model chat, treat this as guidance for how to structure the work conceptually: do the Haiku-shaped parts first and fast, save the deep reasoning for when the context is primed.

### The tiers

- **Haiku** — fast, cheap, high-throughput. Use for: file-level classification, structural extraction, "summarize this in N bullets", syntax/lint checks, regex-level transforms, boilerplate generation, mechanical search-and-replace, reading logs to find the error line, normalizing data formats, querying graphify for simple lookups. Anything where the task is well-defined and the answer is short.

- **Sonnet** — the workhorse. Default tier for most work. Use for: typical coding tasks, writing tests, debugging with moderate complexity, writing documentation, most refactors, data pipeline implementation, SQL authoring, drafting plans that don't require cross-cutting architecture calls.

- **Opus** — the heavy tier. Use for: architectural decisions with real tradeoffs, debugging across multiple systems where the cause is non-obvious, designing new pipelines or schemas from scratch, reviewing critical code for subtle bugs, resolving conflicts between competing designs, anything where being wrong has compounding downstream cost.

### Routing heuristics

Ask these questions before dispatching a subtask:

1. **Is the answer short and the question well-defined?** → Haiku.
2. **Is it well-scoped coding or reasoning inside a known frame?** → Sonnet.
3. **Does getting this wrong cascade?** → Opus.
4. **Am I about to use Opus to do something Haiku could do?** → Re-route to Haiku.
5. **Am I about to use Haiku on something that needs real judgment?** → Escalate to Sonnet or Opus.

### The decomposition pattern

The smart move on a big task is almost always:

1. **Haiku pass (in parallel):** query graphify for structural facts, run `mem-search` for prior observations, classify files by type. Cheap, wide, fast.
2. **Sonnet pass:** synthesize the graphify + claude-mem outputs into a working model of the codebase, draft the plan, flag the hard questions.
3. **Opus pass:** hand Opus the synthesized context and the hard questions only. Do NOT hand Opus raw files — that's exactly the waste this skill exists to prevent. Opus's job is to make the call, not to read the code.

This pattern — **fan out cheap, synthesize middle, concentrate expensive** — is how you keep quality high while spending ~10× less than a naive "just use Opus for everything" approach.

### Anti-patterns in model routing

- **Opus-for-everything.** The classic waste. Most subtasks in a large job are mechanical and Haiku-shaped.
- **Haiku-for-architecture.** The opposite failure. Ships a bad design that costs 100× more downstream than the Opus call would have.
- **Re-dispatching to the same tier for the same work.** If Haiku's classification was insufficient, don't re-run Haiku — escalate to Sonnet.
- **Opus reading source files directly.** Opus should almost never first-read a file. Haiku classifies, Sonnet synthesizes, Opus decides.
- **Switching tiers without saying so.** Tell the user briefly: "Running classifications on the 40 files in /src with Haiku, then bringing Opus in for the schema design question."

## Anti-patterns to avoid

- **Grepping when graphify exists.** If `graphify-out/GRAPH_REPORT.md` is there and you haven't read it, you're doing it wrong.
- **Re-deriving what claude-mem already captured.** Check prior context first. Always.
- **Running without caveman on a tight budget.** The output wrapper will eat 30–40% of the budget on its own.
- **Reading files "just to get oriented."** Orientation is what graphify's report is for.
- **Exhaustive exploration.** Thoroughness is the user's budget to spend, not yours.
- **Silent context burn.** Surface the cost at checkpoints.
- **Proceeding without a budget.** If the user didn't give one and you didn't ask, that's a failure.
- **Silently falling back when a dep is missing.** The skill is designed to fail closed. Refuse and explain.

## Minimal example flow

**User:** "Help me design a new data pipeline to handle the s3 → snowflake sync. Rate limit pressure on the API side is a concern."

**Claude (following this skill):**
1. Notes "rate limit pressure" → skill triggers.
2. Dep check: confirms graphify report exists, claude-mem context is loaded, `/caveman` is available. All green.
3. Asks: "Token budget for this? Want me in `/caveman full` mode?"
4. User: "50k, yes caveman full."
5. **Haiku pass:** queries graphify for the god nodes in the sync subsystem, runs `mem-search "s3 snowflake"` for prior decisions. ~3k tokens.
6. **Sonnet pass:** synthesizes graph report + prior observations into a working model, drafts the proposed design, flags the hard questions. ~15k tokens.
7. **Opus pass (only if warranted):** hands Opus the synthesized context plus the one genuinely hard tradeoff — "per-worker vs centralized rate limiter?" — and gets a decision. ~5k tokens.
8. Reports back: "Plan ready. ~23k used, ~27k left. Open: A, B. Pursue which?" (output short because caveman is on.)
9. Waits for user.

## When to deliberately bypass this skill

This skill is opinionated and that's the point. But there are real cases where its discipline gets in the way:

- **The task is trivial.** "Fix this typo" doesn't need graphify, claude-mem, tier routing, or a budget conversation. The skill should not trigger for one-off simple tasks — that's why the triggers emphasize *long, complex, or codebase-heavy*.
- **The codebase is tiny.** graphify itself notes that graphs add value above ~50 files. For a 6-file repo, just read them.
- **The user explicitly opts out.** If the user says "skip the ceremony, just write the code," respect that. This skill is a default, not a prison.

Use judgment. The goal is spending tokens well, not performing a ritual.
