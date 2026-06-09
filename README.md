<div align="center">

<img src="https://loopcoding.dev/logo.svg" width="92" alt="LoopCoding logo" />

# LoopCoding

**Design loops. Don't prompt agents.**

A [Claude Code](https://claude.com/claude-code) **skill** that designs a tight, self‑verifying autonomous **loop** for a coding task — grounded in *your* real repo and *your* agent's real capabilities.

[![License: MIT](https://img.shields.io/badge/License-MIT-2563eb.svg)](./LICENSE)
[![Claude Code skill](https://img.shields.io/badge/Claude%20Code-skill-7c3aed.svg)](https://claude.com/claude-code)
[![Website](https://img.shields.io/badge/site-loopcoding.dev-0ea5e9.svg)](https://loopcoding.dev)

</div>

> *"You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents."*
> — Peter Steinberger ([@steipete](https://x.com/steipete))

---

## What it is

`loop-coding` flips how you hand work to a coding agent. Instead of babysitting it prompt‑by‑prompt, you
run **`/loop-coding`** and it interviews you, inspects your environment, and writes a **`LOOP.md`** — a
small, self‑verifying spec the agent runs to completion on its own. Then it hands you one clean command to
run it (or offers to launch it for you).

**The mental model:** `loop-coding` is the **architect** → [`/loop`](https://loopcoding.dev) (or a
sub‑agent) is the **engine**. You author the loop once; the agent closes it.

## Why a skill (instead of a copy‑paste generator)

Because a skill has **access**. Before asking you anything, it inventories what actually exists here:

- installed **skills** (`~/.claude/skills`, `.claude/skills`)
- **sub‑agents** / agent teams (`.claude/agents`) for the coder ↔ verifier split
- connected **MCP servers / plugins** (the loop's tools)
- **hooks**, scheduled tasks, and `.github/workflows` (how the loop gets triggered)
- the **codebase** — package manager, test/build/lint scripts, CI, git worktrees

…then it wires the loop to assets that *really exist* and references them by name. A loop that says
"use the `code-reviewer` sub‑agent and the `postgres` MCP server" beats a generic one every time.

## What a loop is

Four parts — and six primitives that make it run:

| The four parts | |
|---|---|
| **Goal** | the precise finished state, edge cases, and what's out of scope |
| **Verification** | the machine‑checkable proof of "done" — the command that lets the agent **stop** |
| **Test‑as‑you‑go** | the cheap per‑iteration check (typecheck / lint / one test / curl) |
| **Guardrails** | boundaries so the agent can't "succeed" by deleting the failing test |

**Six primitives:** Automations · Worktrees · Skills · Plugins/Connectors · Sub‑agents · State — each
resolved against what the skill discovered in your environment.

## Install

**One‑line installer (recommended):**
```bash
curl -fsSL https://loopcoding.dev/install.sh | bash
```

**git:**
```bash
git clone https://github.com/0xArx/loopcoding-skill.git ~/.claude/skills/loop-coding
```

**curl (single file):**
```bash
mkdir -p ~/.claude/skills/loop-coding && \
  curl -fsSL https://loopcoding.dev/SKILL.md -o ~/.claude/skills/loop-coding/SKILL.md
```

Then restart your agent — it's available as **`/loop-coding`**.

## Use

1. Run **`/loop-coding`** → it discovers your environment and asks a few targeted questions → it writes
   `LOOP.md` (+ scaffolds `PROGRESS.md`, the loop's cross‑context memory).
2. It ends with **two options**:
   - **Run it yourself** — copy‑paste the `/loop` command it gives you:
     ```
     /loop Follow ./LOOP.md — run the TEST-AS-YOU-GO checks every iteration, respect every
     GUARDRAIL, and keep going until all DONE-WHEN boxes pass, then stop and summarize.
     ```
     *(Omit an interval to self‑pace, or add one right after `/loop` — e.g. `/loop 15m Follow …` — to
     re‑check on a timer.)*
   - **Let it launch for you** — just say *"launch it"* and it spawns a coder sub‑agent plus an
     independent verifier, runs in the background, and reports at checkpoints.
3. The agent builds, tests, and self‑corrects until every `DONE-WHEN` box passes.

## What it produces — example `LOOP.md`

```md
# LOOP: Add rate limiting to the /api/upload route

## GOAL
Add a 10 req/min per-IP rate limit to POST /api/upload. Return 429 with a Retry-After header when
exceeded. OUT OF SCOPE: auth changes, other routes, the storage layer.

## DONE WHEN (verification — how the agent knows to STOP)
- [ ] `npm test` passes, including new tests in test/upload.rate.test.ts
- [ ] `npm run typecheck` succeeds with no errors
- [ ] `curl -s -o /dev/null -w "%{http_code}" …` returns 429 on the 11th request within a minute

## TEST AS YOU GO (inner loop — every iteration)
- After each change: `npm run typecheck && npm test -- upload`

## GUARDRAILS (do not break)
- The existing suite must stay GREEN — never skip, delete, or weaken a test to pass.
- Only modify files under src/routes/upload/ and test/; ask before touching anything else.
- Commit after each working increment. Append progress + next step to PROGRESS.md every iteration.

## STOP / ESCALATE
- Stop when every DONE-WHEN box is checked, then summarize.
- Hard limit: max 25 iterations.

## LOOP PRIMITIVES (wired to THIS environment)
- Sub-agents: coder = default; verifier = `code-reviewer`
- State / memory: PROGRESS.md
```

## How it works (the phases)

```
Phase 0  Discover the environment     → what skills / agents / MCP / scripts actually exist
Phase 1  Interview the gaps only      → 2–3 sharp questions, defaults offered
Phase 2  Draft LOOP.md                → wired to real assets, iterate until you approve
Phase 3  Hand off                     → copy-paste command  +  offer to launch it now
Phase 4  Monitor & close              → confirm verification actually passed before "done"
```

## Requirements

- **[Claude Code](https://claude.com/claude-code)** (or another agent that loads skills from
  `~/.claude/skills`).
- A **`/loop`** runner to execute the loop — or let the skill launch it via a background sub‑agent.
- Everything else (tests, CI, MCP servers, sub‑agents) is **optional** — the skill discovers and uses
  whatever you have, and degrades gracefully when something isn't there.

> The skill is authored for Claude Code's conventions (`~/.claude/skills`, `/loop`, the `Agent`/`Task`
> sub‑agent tool). It's plain Markdown, so it's easy to adapt to any agent that supports skills + a loop
> runner.

## Contributing

Issues and PRs welcome. The entire skill is a single, readable file — **[`SKILL.md`](./SKILL.md)**.

- Keep it tight: the skill should stay readable in a few screens. Precision beats length.
- Don't add unverified claims or stats — everything user‑facing should be sourced.
- Test a change by installing it locally (the `curl`/`git` methods above point at this folder) and
  running `/loop-coding` against a throwaway repo.

## Credits & sources

The loop framing draws on work by **Peter Steinberger** (loops that prompt your agents),
**Addy Osmani** ("Loop Engineering" — the six primitives), Anthropic's engineering writing on
self‑correction loops, verifier sub‑agents, and memory as an outer loop, and the broader
spec‑driven‑development community.

## License

[MIT](./LICENSE) © Abdulrahman Shahzad ([@0xArx](https://github.com/0xArx))
