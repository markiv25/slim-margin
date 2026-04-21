---
name: token-budget-discipline
description: Enforce strict token-budget discipline AND smart model-tier routing (Opus / Sonnet / Haiku) on long, complex, or codebase-heavy tasks. Use this skill whenever the user mentions a token budget, rate limit, context limit, token pressure, "stay under X tokens", or any form of model-switching / cost-optimization concern — and also proactively on any sufficiently large task (multi-file refactors, new pipeline designs, cross-repo work, architecture reviews) where tokens could easily be wasted re-reading code Claude could have pre-digested or where the wrong model tier would be assigned to subtasks. Triggers on keywords like "rate limit", "token budget", "context window", "stay under", "don't blow the budget", "use a cheaper model", "save tokens", "auto-switch models", "Haiku for this", and on any request where scope clearly exceeds casual handling. This skill forces Claude to (1) invoke codebase-digesting skills (/caveman, /graphify, /claude-mem) BEFORE reading source files directly, and (2) decompose work so cheap mechanical subtasks go to Haiku, workhorse coding goes to Sonnet, and only the hard judgment calls get Opus — turning the budget constraint into a forcing function for smarter orchestration rather than an excuse to underperform.
---

# Token Budget Discipline

A skill for spending tokens on what matters — planning, decisions, architecture — instead of burning them re-reading code Claude could have pre-digested with cheaper, purpose-built skills.

## Why this skill exists

On large tasks (designing a new data pipeline, refactoring across files, reviewing architecture), Claude's default instinct is to read source files directly with its default tier to "understand the code." That burns the entire context window on comprehension — and pays the premium-tier price for mechanical reading work a cheap model could have done. By the time Claude has read everything, the budget is gone and the actual work — the design, the plan, the decisions the user actually wants — never happens, or happens with a depleted context.

There are two fixes, and this skill enforces both:

1. **Budget discipline: prep → confirm → plan.** Use cheap, summarizing skills to pre-digest the codebase before reading raw source. Confirm the budget with the user. Spend what's left on the thinking that only Claude can do.
2. **Tier discipline: fan out cheap, concentrate expensive.** Haiku handles mechanical digestion. Sonnet synthesizes. Opus only gets called in for the hard judgment calls — and only with distilled context, never raw files.

The budget constraint is a **forcing function for smarter orchestration**, not a reason to deliver a worse result.

## When to trigger

Trigger this skill whenever ANY of the following is true:

1. **Explicit budget language** — user says anything like "rate limit", "token budget", "context window", "stay under X tokens", "don't blow the budget", "token pressure", "we're tight on context".
2. **Implicit scope signal** — the task is clearly large: multi-file refactor, new pipeline design, architecture review, cross-repo work, debugging across a big codebase, "understand this project and then…".
3. **User-specified constraint** — user gives any numeric or qualitative budget ("keep it tight", "one-shot this", "minimal exploration").

When in doubt, trigger. Under-triggering this skill is worse than over-triggering — the cost of applying discipline to a task that didn't need it is low; the cost of blowing the budget is starting over.

## The workflow

Follow these steps IN ORDER. Do not skip ahead.

### Step 1: Get the budget

If the user already stated a budget (tokens, "one-shot", "keep it tight", etc.), note it and move on.

If not, ASK before doing anything else:

> "Before I start, what's the token budget for this? I'd like to plan around it so I don't burn the context re-reading code I could summarize first."

Accept any form of answer — a number ("50k"), a vibe ("keep it tight"), or "just do your best" (in which case assume aggressive conservation). Do not proceed without a budget answer.

### Step 2: Pre-digest with cheaper skills BEFORE reading any code

This is the core discipline. Before opening a single source file with `view`, invoke the skills that exist precisely to save tokens:

- **`/caveman`** — use for quick, terse summaries of files or modules. Produces minimal-token digests.
- **`/graphify`** — use for structural understanding: call graphs, module dependencies, data flow. Gives Claude the shape of the code without the bytes.
- **`/claude-mem`** — use to retrieve anything the user (or Claude) has already learned about this codebase in prior sessions. Do not re-learn what's already known.

Rules:
- Invoke these skills FIRST. Reading raw source is a last resort, not a first move.
- If multiple skills apply, run them in parallel where possible.
- If none of these skills are available in the current environment, say so explicitly to the user and ask whether to proceed anyway (with a much smaller implicit budget) or stop.

### Step 3: Plan within the budget

With the digests in hand, produce a plan. The plan is the deliverable for this phase — not code, not implementation. Specifically:

- State what you learned from the digests (1–3 sentences per digested module, not paragraphs).
- State the proposed approach.
- State the open questions — things the digests did NOT answer and that will require reading actual source, running code, or asking the user.
- Estimate the token cost of each open question so the user can choose which to spend the remaining budget on.

### Step 4: Spend the rest with the user's consent

After presenting the plan, STOP. Do not silently start reading source files to resolve open questions. Ask the user which open questions to pursue, given what's left of the budget.

Acceptable phrasings:

> "Here's the plan. Open questions A, B, C are unresolved — A costs ~5k tokens to answer (reading `pipeline.py`), B costs ~15k (full trace through the DAG), C costs ~2k (one clarifying question to you). How do you want to spend what's left?"

### Step 5: Track as you go

At natural checkpoints (end of a tool-use burst, before starting a new section), give a quick budget check:

> "Used roughly ~20k so far, ~30k remaining."

Estimates are fine — the point is to keep the user informed, not to be precise. If you blow through 80% of the budget, stop and confirm before continuing.

## Model tier routing (Opus / Sonnet / Haiku)

Budget pressure is not just a reason to use fewer tokens — it is a reason to use the **right tier** for each subtask. The goal is not "use the cheapest model everywhere"; that underperforms. The goal is to **decompose the task so cheap models handle the mechanical work and the expensive model gets the hard thinking with its full attention intact.**

This section applies in any environment where the orchestrator can choose a model per subtask — Claude Code, the Anthropic API with sub-calls, agentic setups, or any harness that dispatches work to different tiers. In a single-model chat (plain Claude.ai), treat this as guidance for how to structure the work conceptually: do the Haiku-shaped parts first and fast, save the deep reasoning for when the context is primed.

### The tiers

- **Haiku** — fast, cheap, high-throughput. Use for: file digests, structural extraction, "summarize this in N bullets", syntax/lint checks, regex-level transforms, boilerplate generation, classification ("is this file a test? a config? a model?"), mechanical search-and-replace, reading logs to find the error line, normalizing data formats. Anything where the task is well-defined and the answer is short.

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

1. **Haiku pass:** run cheap digests in parallel across every file in scope. Get titles, 5-line summaries, exported symbols, dependencies. This is what `/caveman`, `/graphify`, `/claude-mem` should be doing under the hood — if they're calling Sonnet/Opus for simple summarization, that's waste.
2. **Sonnet pass:** synthesize the Haiku outputs into a working model of the codebase, draft the plan, flag the hard questions.
3. **Opus pass:** hand Opus the synthesized context and the hard questions only. Do NOT hand Opus the raw files — that's exactly the waste this skill exists to prevent. Opus's job is to make the call, not to read the code.

This pattern — **fan out cheap, synthesize middle, concentrate expensive** — is how you keep quality high while spending ~10x less than a naive "just use Opus for everything" approach.

### Anti-patterns in model routing

- **Opus-for-everything.** The classic waste. Most subtasks in a large job are mechanical and Haiku-shaped.
- **Haiku-for-architecture.** The opposite failure. "It's just a design question, Haiku can do it" → ships a bad design, costs 100x more downstream than the Opus call would have.
- **Re-dispatching to the same tier for the same work.** If Haiku's digest was insufficient, don't re-run Haiku hoping for better — escalate to Sonnet.
- **Opus reading source files directly.** Opus should almost never be the tier that first-reads a file. Haiku reads, Sonnet synthesizes, Opus decides.
- **Switching tiers without saying so.** When dispatching subtasks to different tiers, tell the user briefly: "Running cheap digests on the 40 files in /src with Haiku, then I'll bring Opus in for the schema design question." Transparency keeps the user in control of the budget.

## Anti-patterns to avoid

- **Reading files "just to get oriented."** If you find yourself about to `view` a file before running any digesting skill, stop. Run the digesting skill instead.
- **Exhaustive exploration.** Do not read every file in a directory to "be thorough." Thoroughness is the user's budget to spend, not yours.
- **Silent context burn.** Do not use 30k tokens of tool calls and then report a plan as if nothing was spent. Surface the cost.
- **Proceeding without a budget.** If the user didn't give one and you didn't ask, that's a failure of this skill.
- **Re-learning what `/claude-mem` already knows.** Always check memory first.

## Minimal example flow

**User:** "Help me design a new data pipeline to handle the s3 → snowflake sync. Rate limit pressure on the API side is a concern."

**Claude (following this skill):**
1. Notes "rate limit pressure" → skill triggers.
2. Asks: "What's the token budget for this design session? And am I free to dispatch subtasks across Haiku/Sonnet/Opus, or should I stay on one tier?"
3. User: "50k budget, and yes — route smartly."
4. **Haiku pass (parallel):** runs `/claude-mem` for prior context, `/graphify` for the dependency shape, `/caveman` for file-level digests across the sync modules. ~5k tokens.
5. **Sonnet pass:** synthesizes the digests into a working model of the current pipeline, drafts the proposed design, flags the hard questions. ~15k tokens.
6. **Opus pass (only if warranted):** hands Opus the synthesized context plus the one genuinely hard tradeoff — "should the rate limiter be per-worker or centralized?" — and gets a decision. ~5k tokens.
7. Reports back: "Here's the plan. ~25k used, ~25k left. Open questions A and B — want me to spend the rest on those, or ship it as-is?"
8. Waits for user.

## Note on skill availability

The skills this one depends on (`/caveman`, `/graphify`, `/claude-mem`) may not be present in every environment. If they're not available, be upfront:

> "The skills I'd normally use to pre-digest the codebase (`/caveman`, `/graphify`, `/claude-mem`) aren't available here. I can either (a) proceed with aggressive manual conservation — read only the minimum, ask before each file — or (b) stop and we can revisit once those are installed. Which do you prefer?"

Never silently fall back to reading source files when the digesting skills are missing. That defeats the purpose.
