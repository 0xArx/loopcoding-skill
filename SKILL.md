---
name: loop-coding
description: >-
  Design a tight, self-verifying autonomous LOOP for a coding task — grounded in THIS repo and THIS
  agent's real capabilities. Unlike a copy-paste loop generator, this skill first inventories the actual
  environment (installed skills, subagents, MCP servers/plugins, hooks, scheduled tasks, the codebase,
  git worktrees), then interviews you only for the gaps, wires the loop to assets that really exist, and
  can launch it. TRIGGER when the user says "loop coding", "design a loop", "start a loop", "loopcode
  this", "set up a loop for X", or asks to build an autonomous agent loop. SKIP for a one-shot edit the
  user just wants done now.
---

# Loop Coding

You are entering **Loop Design mode**. Your job is *not* to start building the feature — it's to design
a self-verifying **loop** that an agent runs to completion, and to wire that loop to the tools, skills,
subagents, and memory that **actually exist in this environment**. The human authors the loop; the agent
closes it.

The whole point of doing this as a skill (instead of an online generator) is **access**: you can read the
real codebase, enumerate the installed skills and subagents, see which MCP servers/plugins are connected,
and reference them by name in the loop. A loop that says "use the `code-reviewer` subagent and the
`postgres` MCP server" beats a generic one every time.

A complete loop has **four parts** — Goal · Verification · Test-as-you-go · Guardrails — plus **six
primitives** that make it run: Automations · Worktrees · Skills · Plugins/Connectors · Sub-agents · State.

## Design vs. execute (how this fits with `/loop`)
This skill **designs** the loop — it writes a `LOOP.md` file. **Running** it is a separate step:
- The user runs **`/loop`** pointed at the file — e.g. `/loop Follow LOOP.md — keep iterating until every
  DONE-WHEN box passes, then stop.` (omit an interval to self-pace; or put one **right after `/loop`** —
  e.g. `/loop 10m Follow LOOP.md …` — to re-check on a timer), **or**
- this skill **launches it now** as a background subagent (coder + an independent verifier).

So: **`loop-coding` is the architect; `/loop` (or a subagent) is the engine.** Always end the session by
telling the user the exact command to run their loop — don't leave them guessing.

> **Built-in loop primitives worth knowing:** Claude Code's **`/goal`** and Claude Managed Agent's
> **`Outcomes`** implement this same pattern as a first-class feature — a goal + rubric the model grades
> itself against, with an independent grader sub-agent spawned automatically. Use them when you're inside
> those platforms; use a `LOOP.md` for full portability or cross-tool composition.

---

## The procedure

Do the phases in order. Do **not** skip discovery, and do **not** start implementing the feature — your
deliverable is the loop, not the code.

### Phase 0 — Discover the environment (the differentiator)
Before asking anything, inventory what's actually here. Run these and keep the results; they make every
later question sharper and let you wire the loop to real assets.

**Installed skills** (composable knowledge):
- `ls ~/.claude/skills/ .claude/skills/ 2>/dev/null`
- For the relevant ones, read the frontmatter `name` + `description` so you know what each does.

**Subagents / agent teams** (for the coder↔verifier split):
- `ls .claude/agents/ ~/.claude/agents/ 2>/dev/null` — read each agent's name/description.
- Also note the built-in agent types available to you (e.g. Explore, Plan, code-reviewer).

**Plugins / MCP connectors** (the loop's tools):
- Check `.mcp.json`, `.claude/settings.json`, `.claude/settings.local.json`, `~/.claude/settings.json`
  for `mcpServers` / plugin config.
- Note which MCP tools are actually connected in this session (you can see them in your tool list).

**Automation surface** (how the loop gets triggered):
- Hooks: look for `hooks` in the settings.json files above.
- Scheduled tasks / cron: check for a `schedule`/`loop` skill, cron entries, or `.github/workflows/`.

**The codebase** (what "verify" and "test" mean here):
- Detect package manager + scripts: `package.json` (test/build/lint/dev), `pyproject.toml`/`Makefile`/
  `Cargo.toml`/`go.mod`, etc.
- CI config: `.github/workflows/`, other CI files — reuse the same checks the team trusts.
- Git: current branch, `git status`, and `git worktree list` (for parallel isolation).

Summarize what you found in 4-6 lines before moving on ("Here's what this environment gives us…").

### Phase 1 — Interview the gaps only
Ask in **small batches of 2-3 questions** (use `AskUserQuestion` when available; offer a default in
[brackets]). Don't ask what discovery already answered. Cover:
- **Goal & scope** — the precise finished state, edge cases, what's OUT of scope, files in play.
- **Verification** — which discovered command proves "done" (the test/build/script). Tests exist or must
  be written? Any eval/judge step?
- **Test-as-you-go** — the cheap per-iteration check (typecheck/lint/one test/curl).
- **Guardrails** — what must never break; off-limits paths/secrets/prod; commit cadence; stop/escalate.
- **Primitives** — for each of the six, confirm whether to use it and *which discovered asset*:
  - Automations → one-shot run-until-done, `/loop`, a scheduled task, or a hook?
  - Worktrees → single tree, or a `git worktree` per parallel task?
  - Skills → which installed SKILL.md(s) to load (offer the relevant ones you found by name)?
  - Plugins/Connectors → which connected MCP servers/tools does the loop need?
  - Sub-agents → which agent definition is the coder, which is the independent verifier?
  - State → `PROGRESS.md` in-repo, `AGENTS.md`, or an issue tracker (e.g. Linear via MCP if connected)?

### Phase 2 — Draft LOOP.md (wired to real assets)
Write `LOOP.md` from the template below, naming the **actual** skills/agents/MCP servers/commands you
discovered — not placeholders. Scaffold `PROGRESS.md` (the loop's cross-context memory). Show LOOP.md to
the user and iterate until they approve. A sharp, environment-specific spec is the whole value.

### Phase 3 — Launch
Once `LOOP.md` is approved, either launch it for them or hand them the run command. Options:

- **Let me launch it now (default if they say go):** spawn the coder subagent (the `Task`/`Agent` tool,
  using the discovered agent definition if there is one) with the full `LOOP.md` as its prompt, plus a
  **separate verifier subagent** to check each iteration. Run in the background; report at checkpoints.
  Use a `git worktree` if they chose parallel isolation.
- **Run it yourself:** hand them the copy-paste command (see Phase 4 — this is the most important
  thing you produce; make it clean).
- **Scheduled / hook:** if they wanted automation, register it via the `schedule` skill or a settings hook.
- **Run elsewhere (Codex/cloud):** print `LOOP.md` as a ready-to-paste task block.

Never auto-launch anything touching prod, remote systems, or money without explicit confirmation.

### Phase 4 — End with ONE beautiful copy-paste command (always)
This is the final thing every loop-coding session must produce. After `LOOP.md` is written, end your reply
with a single, clean, ready-to-run command that makes `/loop` follow the file — using the **real path**
where you saved it. Make it impossible to get wrong: state the path, then the command, set off on its own
so it's one-click copyable. Use exactly this shape (fill in the real path and a 1-line goal recap):

> ✅ Your loop is ready → \`<path>/LOOP.md\`
>
> Copy-paste this to run it:
>
> ```
> /loop Follow <path>/LOOP.md — <one-line goal>. Work the loop: run the TEST-AS-YOU-GO checks every
> iteration, respect every GUARDRAIL, and keep going until all DONE-WHEN boxes pass, then stop and summarize.
> ```
>
> _Tip: omit an interval to self-pace, or add one **right after `/loop`** (e.g. `/loop 15m Follow …`) to
> re-check on a timer._

Keep it to that — one command, the right path, nothing noisy around it. That command is the handoff.

### Phase 5 — Monitor & close
Keep the user oriented: surface checkpoint pauses, summarize what changed, and confirm the verification
actually passed before declaring done. If the agent got stuck or "succeeded" by weakening a test, the
spec was incomplete — fix `LOOP.md` (often a missing guardrail), then relaunch.

---

## LOOP.md template

Fill every `<…>` with real, discovered values. Keep it tight and unambiguous.

```md
# LOOP: <short title>

## GOAL
<one precise paragraph: behavior, edge cases, and what is explicitly OUT OF SCOPE>

## DONE WHEN (verification — how the agent knows to STOP)
- [ ] <real command, e.g. `npm test`> passes, including new tests for <feature>
- [ ] <real build/typecheck> succeeds with no errors
- [ ] <acceptance check: command/script> produces <expected output>

## TEST AS YOU GO (inner loop — every iteration)
- After each change: `<fast check discovered in this repo>`
- Observe behavior: `<dev server / curl / script>` -> expect <result>

## GUARDRAILS (do not break)
- The existing suite must stay GREEN — never skip, delete, or weaken a test to pass.
- Only modify files under <scope>; ask before touching anything else.
- Off-limits: <secrets / migrations / prod / network>.
- Commit after each working increment with a clear message.
- Append progress + next step to PROGRESS.md every iteration.
- <multi-session runs only> After any failure: document the exact error → diagnose the cause → verify the
  fix with a real check → distill a general rule into PROGRESS.md → consult it next session. No unverified guesses.

## STOP / ESCALATE
- Stop when every DONE-WHEN box is checked, then summarize.
- Pause for a human if: <ambiguity / a check you can't fix in N tries / out-of-scope need>.
- Hard limits: max <N> iterations or <T> minutes.

## SETUP (run once)
- <install deps / env vars / seed data, or "none">

## LOOP PRIMITIVES (wired to THIS environment — include only what applies)
- Automations: <one-shot run-until-done | /loop | scheduled task | hook>
- Worktrees: <single tree | `git worktree` per parallel task>
- Skills: <names of installed SKILL.md(s) to load, e.g. `frontend-design`, or "none">
- Plugins / connectors: <connected MCP servers/tools the loop uses, or "none">
- Sub-agents: <coder = <agent>, verifier = <agent>; where their defs live, or "single agent">
- State / memory: <PROGRESS.md | AGENTS.md | Linear via MCP>

## THE LOOP
Work toward GOAL. After each change, run the TEST-AS-YOU-GO checks. If a check fails, diagnose and fix
before continuing. Respect every GUARDRAIL. When every DONE-WHEN box is checked, STOP and summarize.

## HOW TO RUN THIS LOOP
Copy-paste this into the agent:

    /loop Follow ./LOOP.md — work toward the GOAL, run the TEST-AS-YOU-GO checks every iteration,
    respect every GUARDRAIL, and keep going until all DONE-WHEN boxes pass, then stop and summarize.

(No interval = self-paced; or add one right after `/loop`, e.g. `/loop 15m Follow ./LOOP.md …`, to re-check on a timer.)
- Or ask the loop-coding skill to launch it now (background coder + independent verifier subagent).
- To run elsewhere (Codex / a cloud agent): paste this entire file as the task.
```

> Tip: keep the `## HOW TO RUN THIS LOOP` block in the file — it makes `LOOP.md` self-documenting, so
> anyone (or any future session) who opens it knows exactly how to start it.

---

## Loop-primitive → real-asset mapping (resolve against discovery)

| Primitive | Job | Resolve to (this environment) |
|---|---|---|
| **Automations** | trigger / cadence | `/loop`, the `schedule` skill, a settings.json hook, `.github/workflows`, or one-shot run-until-done |
| **Worktrees** | isolate parallel work | `git worktree add` per task; or `isolation: worktree` on a spawned subagent |
| **Skills** | codify knowledge | the installed `SKILL.md`s under `~/.claude/skills` / `.claude/skills` you found — load by name |
| **Plugins / connectors** | tools | the MCP servers/plugins connected this session (name them) |
| **Sub-agents** | coder + verifier | agent defs in `.claude/agents/`, built-in agent types, or agent teams — **keep the verifier independent**: grading in a fresh context window consistently outperforms self-critique because the model can't reliably catch its own errors in the same window |
| **State / memory** | track progress | `PROGRESS.md` / `AGENTS.md` committed to the repo, or an issue tracker via a connected MCP |

---

## Memory as an outer loop

For runs that span multiple sessions or many iterations, memory is the **outer loop** — it lets the agent
build verified knowledge across context windows instead of re-deriving the same facts each time.
Effective memory use follows a progression; require it in your GUARDRAILS when the run will span sessions:

| Step | What the agent does |
|---|---|
| **1 · Fail** | Attempt the task; document what went wrong and the exact error |
| **2 · Investigate** | Before moving on, find out *why* — reproduce the failure, narrow the cause |
| **3 · Verify** | Turn the diagnosis into a **checked fact** (confirm the schema, test the assumption with a real query) |
| **4 · Distill** | Compress the verified finding into a **general rule** written into `PROGRESS.md` or memory |
| **5 · Consult** | On the next session or iteration, **read** the rule — don't re-derive it |

Models that short-circuit to step 1 (a list of failure notes and open guesses like "maybe `prc` instead of
`prc_usd`?") stay stuck re-deriving the same facts. The full progression means verified rules accumulate
and performance compounds across sessions. Concretely: Fable 5-class models running this progression reach
up to **73% verification coverage** on memory tasks vs. under 20% for models that skip to "fail + note."

Include this in your GUARDRAILS template for any loop that will span multiple context windows:

```
MEMORY RULE: After any failure, complete the full progression before moving on:
(1) document the exact error → (2) diagnose the cause → (3) verify the diagnosis with a real check
→ (4) distill into a general rule in PROGRESS.md → (5) consult that rule in the next session.
Never write an open guess (e.g. "maybe X?") without verifying it first.
```

---

## Principles to enforce
- **Discover before you ask, and wire to what's real.** Reference actual skills/agents/MCP/commands by
  name. This grounding is the reason to run a skill instead of a generic generator.
- **No coding before the spec.** If you're editing the feature during the interview, stop.
- **Verification before goal.** If the user can't say how success is checked, the loop isn't ready.
- **Keep the verifier independent** from the coder so it can actually catch bad work.
- **Guardrails close the shortcut to "done"** — an unconstrained agent will delete the failing test.
- **PROGRESS.md is the loop's memory** across context windows — insist on it for multi-step runs.
- **Human "on the loop," not "in the loop":** review at checkpoints, not every keystroke.
- Keep the spec readable in one screen. Precision beats length.
- **Loops scale with better models.** A well-designed loop doesn't become obsolete as models improve —
  it gets *more* powerful. Better models make bigger structural bets, push through regressions, reach
  higher verification coverage, and complete the full memory progression unsupervised. Design the loop
  right once; the model capability does the heavy lifting. Brian Cherny (@bcherny) at Anthropic put it
  plainly: *"my job is to write loops."*
