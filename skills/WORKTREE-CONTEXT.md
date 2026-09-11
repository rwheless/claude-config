# Worktree context: shared docs in gitignored .claude/

Work repos (Oracle SQL / Spring Boot / Angular) gitignore the entire `.claude/`
directory — not every teammate uses Claude Code, so it never gets committed.
Personal repos commit and push `.claude/context/`, so git's normal merge
process already reconciles it across branches; nothing here applies to them.

Because a work repo's `.claude/` is never committed, each worktree of a
bare-repo-and-worktrees setup gets its own independent, permanently divergent
copy of anything written under it — there is no merge step to bring them back
together. A term added to `CONTEXT.md` in one issue's worktree stays invisible
to every other worktree forever.

## The fix: one physical file, not one per worktree

For the paths below, treat the **main/master worktree's copy as the only
copy** — read and write it directly, regardless of which worktree the current
session is running in.

**In scope** (redirect when gitignored):
- `.claude/context/CONTEXT.md`
- `.claude/context/CONTEXT-MAP.md`
- `.claude/context/ORIENTATION.md`
- `.claude/jira-planning/` (the whole directory)
- `.claude/research/` (the whole directory, written by `research-this`)

**Out of scope** (never redirect, even though also gitignored):
- `.claude/wrap-up/IMPLEMENTATION-HANDOFF.md` — intentionally scoped to the
  worktree it's written in; it records that worktree's own in-flight state.
- `.claude/context/adr/` (legacy) — superseded by tracked `docs/adr/`, which
  git already reconciles normally. Not worth building redirect support for.

## How to resolve the root

Once per session, before the first touch of any in-scope path:

1. Check whether it's actually gitignored here: `git check-ignore -q .claude`.
   Personal repos exit non-zero (tracked) — read/write the current worktree's
   copy exactly as before; nothing else in this file applies.
2. If ignored, run `~/.claude/scripts/resolve-main-worktree.sh`. It prints the
   absolute path of the repo's `master` (preferred) or `main` worktree, or
   nothing if neither exists (e.g. a normal single-checkout repo, or a
   worktree layout that doesn't follow this convention).
3. If it printed a path: use `<that path>/.claude/context/...` and
   `<that path>/.claude/jira-planning/...` as the read/write root for the rest
   of the session — not the current directory.
4. If it printed nothing: fall back to the current directory, unchanged. Safe
   no-op rather than guessing at a layout that isn't there.

Resolve once and reuse the result — don't re-run the script per file access.

## Why this design

- **Gated on `git check-ignore`, not branch-name convention** — the actual
  cause is "nothing reconciles this file," which gitignore status answers
  directly. Hardcoding "master = work = redirect, main = personal = don't"
  would silently misbehave the day either convention has an exception.
- **`git worktree list --porcelain`, not a fixed relative path** — worktrees in
  this setup live in inconsistent places (inside the bare repo directory for
  some projects, in a sibling `-worktrees/` directory for others). Asking git
  directly for the worktree list works regardless of layout.
- **Full redirect, not read-through-then-write-locally** — since these paths
  are never committed, there's no branch-scoped copy to protect; every
  worktree touching an in-scope path is touching the *same* physical file by
  design, so there's nothing to reconcile later.
