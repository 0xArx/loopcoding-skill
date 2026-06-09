# LoopCoding — the loop-design skill 🔁

> *"You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents."* — Peter Steinberger (@steipete)

`loop-coding` is a Claude Code **skill** that designs a tight, self-verifying autonomous **loop** for a coding task — grounded in *your* real repo and agent: its installed skills, subagents, MCP servers, hooks, and git worktrees. It writes a `LOOP.md` (+ `PROGRESS.md`), then hands you one command to run it.

**The model:** `loop-coding` is the architect → `/loop` is the engine.

## Install

**git (recommended):**
```bash
git clone https://github.com/0xArx/loopcoding-skill.git ~/.claude/skills/loop-coding
```

**curl (single file):**
```bash
mkdir -p ~/.claude/skills/loop-coding && \
  curl -fsSL https://loopcoding.dev/SKILL.md -o ~/.claude/skills/loop-coding/SKILL.md
```

Then restart your agent — it's available as `/loop-coding`.

## Use

1. Run `/loop-coding` → answer a few questions → it writes `LOOP.md`.
2. Run the loop with the command it gives you:
   ```
   /loop Follow ./LOOP.md — run the checks every iteration, respect every guardrail,
   and keep going until all DONE-WHEN boxes pass, then stop.
   ```
3. The agent builds, tests, and self-corrects until done.

Learn more at **[loopcoding.dev](https://loopcoding.dev)**.
