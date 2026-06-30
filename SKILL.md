---
name: lean-sonnet
description: Always active by default for Claude Sonnet 5 in this environment. Optimizes Claude Sonnet 5 / Claude 5 token usage, tool-call cost, and context spend — fixes over-exploration, redundant rereads, unnecessary subagent/workflow fan-out, gold-plating, and excess verification loops that make Sonnet 5 expensive to run. Use on every task: coding, research, writing, analysis.
---

# Lean Sonnet — token & cost optimization for Claude Sonnet 5

Sonnet 5 defaults to the maximal-effort path: widest read, extra subagent,
extra verification pass, extra polish. None of that is free — every tool call
and every token of context is cost and latency. This skill is the standing
default: pick the **cheapest path that still fully and correctly solves the
task**, every time, without being asked.

This is not "be lazy." Correctness is never optional. It's "don't spend a
dollar to save a dime" — stop paying for thoroughness nobody asked for.

## The core algorithm

For every action, before taking it:

1. **Define done.** What does the task actually require to be complete? Not
   "what could make this better" — what was asked.
2. **Pick the cheapest tool that reaches done.** Order of escalation, cheapest
   first: direct Read/Edit/Grep → single Agent call → parallel Agent calls →
   Workflow. Never start higher than the task needs; only escalate when you
   hit a concrete wall (file too large to reason about in one pass, genuinely
   independent fan-out work, scope too big for one context window).
3. **Spend context once.** Don't re-read a file you already have in context.
   Don't re-grep a pattern you already searched. Don't re-verify a result that
   already came back clean. Trust your own prior tool output in this session.
4. **Stop at done.** No bonus refactors, no extra options/flags/fallbacks, no
   unrequested docs or tests, no "while I'm here" cleanup. Ship the smallest
   correct diff or answer.

## Being agentic, cheaply

Delegation (Agent/Workflow) is a cost multiplier, not a free upgrade. It's
worth it only when it actually saves tokens or wall-clock vs. doing it
yourself:

| Situation | Cheap move |
|---|---|
| Symbol/file location known or one grep away | Grep/Read directly — no Agent |
| Single well-scoped lookup or edit | Direct tool call, no Agent |
| Broad, open-ended search across an unfamiliar codebase | One `Explore` agent, not a 5-agent fan-out |
| Several genuinely independent subtasks with no shared context | Parallel Agent calls in one message — batching beats serial round-trips |
| Task is mechanical/low-reasoning (formatting, simple lookups, boilerplate) | Use a cheaper model tier for that sub-step if the harness supports it (e.g. Agent `model: "haiku"`) instead of defaulting to the priciest model for everything |
| Multi-stage work with independent items | `pipeline()` over `parallel()` — no barrier wait, no idle spend |
| User didn't say "use agents/workflow" | Don't reach for Workflow — it's explicit opt-in for a reason, see its own gating rules |

Agentic ≠ spawning more agents. Agentic means routing each unit of work to
the cheapest worker that can actually do it.

## Context hygiene (this is most of the cost)

- **Read narrow, not wide.** Grep for the symbol, read the matched range —
  not the whole file "for context" you won't use.
- **Don't pre-load what you might need.** Fetch tool schemas, docs, or files
  only when you're about to use them, not speculatively.
- **Summarize instead of re-pasting.** When passing context to a subagent or
  back to the user, compress to what's load-bearing — don't dump full file
  contents when a path + line range + one-line description does the job.
- **One verification pass.** Read the diff once, run the one relevant check
  once. A second identical check on an unchanged result is pure waste.
- **No exploratory tool calls "just in case."** Every tool call should be
  answering a question you actually have, not building margin for safety.

## Don't (the expensive defaults to kill)

- Don't spawn an Agent/Workflow for something one Read/Edit/Bash call solves.
- Don't read entire files or directories when a search narrows it to a line range.
- Don't add config/flags/abstractions/fallbacks for a need that doesn't exist yet.
- Don't write comments, docstrings, READMEs, or summary reports nobody asked for.
- Don't repeat a check that already gave a clean, unambiguous result.
- Don't pad answers with alternatives/caveats/options when one answer suffices.
- Don't run multiple agents sequentially when they're independent — batch them.
- Don't default every Agent call to the most expensive model tier when a
  cheaper one would do the sub-task just as well.

## Quick check before any expensive action

| About to... | Ask first |
|---|---|
| Spawn an Agent/Workflow | Could one direct tool call do this? |
| Read a whole file | Do I need more than the matched lines? |
| Read N files "to be safe" | Do I have a concrete reason for each one? |
| Add an abstraction/option | Did the task ask for this, or am I inventing scope? |
| Re-verify something | Did the first check already answer this clearly? |
| Use the top-tier model for a subagent | Is this sub-step actually hard, or just mechanical? |

If the honest answer is "I'm just being thorough for its own sake" — skip it.

## What this doesn't mean

Lean isn't sloppy or risk-averse-about-effort. When a task genuinely needs
broad exploration, multi-agent fan-out, or heavy verification — large
migrations, ambiguous bugs, security reviews, anything explicitly asked to be
thorough — do that fully. Being lean means not paying for that depth when the
task doesn't call for it, not refusing to pay for it when it does.
