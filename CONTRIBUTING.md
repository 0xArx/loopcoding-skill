# Contributing to LoopCoding

Thanks for helping improve the `loop-coding` skill. The entire skill is one readable file:
**[`SKILL.md`](./SKILL.md)** — that's where almost every change happens.

## Ground rules

- **Keep it tight.** The skill should stay readable in a few screens. Precision beats length.
- **No unverified claims or stats.** Anything user‑facing should be sourced.
- **Preserve the shape.** Keep the phase structure (discover → interview → draft → hand off → monitor)
  and the `LOOP.md` template intact unless the change is specifically about improving them.

## Test your change

1. Copy your edited `SKILL.md` to `~/.claude/skills/loop-coding/SKILL.md` (or point a fork's install at it).
2. Restart your agent and run **`/loop-coding`** against a throwaway repo.
3. Confirm it:
   - discovers the environment before asking anything,
   - asks only the gap questions,
   - writes a `LOOP.md` wired to assets that actually exist,
   - ends with the copy‑paste `/loop` command **and** the offer to launch it.

## Submitting

- Open an issue first for anything larger than a small wording/clarity fix.
- Keep PRs focused — describe **what** changed and **why**.
- After a change is merged, the maintainer syncs `SKILL.md` to the website and installer
  (`loopcoding.dev/SKILL.md`), so you don't need to touch those.

By contributing, you agree your contributions are licensed under the repository's
[MIT License](./LICENSE).
