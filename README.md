# lean-sonnet

**A Claude Code skill that optimizes Claude Sonnet 5 for lower token usage and lower cost.**

Sonnet 5 is powerful but token-hungry by default: it over-explores codebases, rereads files it already has, spawns subagents/workflows for tasks a single tool call could solve, and gold-plates simple requests with unrequested abstractions and extra verification passes. `lean-sonnet` is a standing default-on skill that fixes this — it makes Claude Sonnet 5 pick the cheapest correct path on every task instead of the most expensive one.

## What this fixes

- **Token waste** — rereading files/context already in hand, reading whole files instead of matched ranges
- **Tool-call sprawl** — spawning an Agent or Workflow for something one Read/Edit/Grep call solves
- **Redundant verification** — re-running the same check after it already passed
- **Scope creep / gold-plating** — unrequested abstractions, config flags, fallbacks, docs, comments
- **Inefficient delegation** — serial agent calls that should be batched, or top-tier models used for mechanical sub-tasks that a cheaper tier could handle

## Keywords

claude sonnet 5 optimization, reduce claude token usage, claude code cost optimization, anthropic sonnet 5 skill, claude agent skill, lower claude api cost, claude code token efficiency, claude sonnet 5 agentic workflow, cheaper claude code, claude 5 cost reduction, claude code skills directory

## Install

Drop `SKILL.md` into your Claude Code skills directory:

```bash
git clone https://github.com/<your-username>/lean-sonnet.git ~/.claude/skills/lean-sonnet
```

Claude Code auto-discovers it from `~/.claude/skills/`. For project-local use, clone into `.claude/skills/lean-sonnet` inside your repo instead.

To make it apply on every turn rather than relying on description-matching, add a pointer to it in your global `~/.claude/CLAUDE.md`:

```markdown
Apply the `lean-sonnet` skill's rules on every task: cheapest tool that
reaches "done", no unnecessary subagent/workflow fan-out, no rereading
context already in hand, one verification pass, no unrequested scope.
```

## License

MIT
