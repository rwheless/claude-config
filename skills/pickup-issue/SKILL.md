---
name: pickup-issue
description: Pick up a Jira issue for implementation. Accepts a Jira slug + summary/description, orients in the codebase, runs grill-with-docs to reach alignment, creates a Git worktree named after the slug (location per WORKTREE-LOCATION.md), implements (using TDD for Angular/Spring Boot code changes), then self-reviews the diff via the bundled code-review skill before handing the branch over. Use when the user says "I'm picking up [JIRA-SLUG]" or invokes /pickup-issue.
---

# pickup-issue

You are picking up a Jira issue for implementation. Input is everything after
`/pickup-issue` (or the user's natural-language description). Parse the Jira slug as
the first whitespace-delimited uppercase token (e.g. `LNES-129`); treat the rest as
the issue summary and description.

If what follows the slug reads like a short title or paraphrase rather than the
full issue (no acceptance criteria, no detail beyond a sentence), ask the user to
paste the actual Jira issue description before exploring or grilling — a short
paraphrase can omit scope-defining details (e.g. which specific components/entities
are named) that would otherwise only surface after significant grilling has already
happened.

## Workflow

Run these phases strictly in order. Do not create a worktree or write any code until
Phase 3 alignment is explicitly confirmed by the user.

```
1. ORIENT  →  2. GRILL  →  3. CONFIRM ALIGNMENT  →  4. WORKTREE  →  5. IMPLEMENT
  →  6. SELF-REVIEW  →  7. DOCS
```

### Phase 1: ORIENT

**Resumed pickup check (first):** resolve the worktree directory per
`WORKTREE-LOCATION.md` (run `~/.claude/scripts/resolve-worktree-root.sh` once
now, reuse the result all session): `<bare-root>/{JIRA-SLUG}/` if it printed a
path, else `../<repo>-worktrees/{JIRA-SLUG}/`. If that worktree already exists
and contains `.claude/wrap-up/IMPLEMENTATION-HANDOFF.md`, this is a
resumed pickup. Follow the resume contract in `wrap-up/IMPLEMENTATION.md`: read the
handoff, verify it against `git log`/`git status`, skip re-grilling what it records
as aligned (address "Not yet aligned" items first), reuse the worktree in Phase 4,
land in the phase it names, and delete the handoff once caught up.

Otherwise, before asking the user anything, gather context:

1. Read `.claude/context/CONTEXT.md` and `docs/adr/` (plus any legacy
   `.claude/context/adr/`) if they exist. In a work repo, resolve the
   shared-doc root per `WORKTREE-CONTEXT.md` before this read.
2. Check for `issues.csv` in the relevant planning directory (`.claude/jira-planning/`,
   same resolved root as above). If found, read it for context on the issue
   being picked up and its dependencies before exploring the codebase.
3. Explore the code the issue most likely touches — data model, service layer, Helm
   charts, Angular components, whatever the description points at. Delegate this
   exploration per `grill-with-docs/EXPLORATION.md` (read it now, plus
   `grill-with-docs/STACK-WORK.md`) — one Explore agent, its report format, no
   persistence. For deployment or
   infrastructure issues, also search CI/CD pipeline files (Bamboo specs, GitHub
   Actions, etc.) and deploy scripts for references to the same secrets, flags, or
   values being changed.
4. Build an internal picture: what exists, what will change, what is ambiguous.

**Bug gate:** if the issue is a Bug and its text does not already pin a confirmed
root cause, run the `debug-problems` skill now — you cannot grill toward a fix
for an unknown cause. On return the issue text carries the DIAGNOSIS; Phase 2
grills against it (typically much shorter, since cause and evidence are
pre-answered).

Output a brief orientation summary (5 lines max), then move to Phase 2. Do not
dump file listings at the user.

### Phase 2: GRILL

Follow the `grill-with-docs` skill. Read its SKILL.md now if you have not already.

Goal: shared understanding of exactly what needs to change, why, what is out of
scope, and what could go wrong. Update `.claude/context/CONTEXT.md` as terms
resolve. Do not write ADRs or product docs — see Phase 7.

The grilling is done only when the user explicitly confirms (e.g. "we're aligned",
"let's do it", "that's everything"). Never declare alignment yourself.

### Phase 3: CONFIRM ALIGNMENT

Summarize the shared understanding:

1. The problem being solved and the proposed change.
2. Which files / layers are affected.
3. Whether TDD applies: **yes** if the change includes Angular or Spring Boot
   production code; **no** if the change is config, deployment, SQL, or docs only.
4. What is explicitly out of scope.
5. The worktree path (resolved in Phase 1 per `WORKTREE-LOCATION.md`), branch name (always
   the exact Jira slug, e.g. `LNES-129`), and base branch (default: `master`;
   use any other branch the user specifies).

Ask the user to confirm. Iterate until explicit confirmation. Only then proceed.

### Phase 4: WORKTREE

Create the worktree and branch at the directory resolved in Phase 1 (a sibling
directory, or inside the bare repo itself — see `WORKTREE-LOCATION.md`; either
way it keeps concurrent agents isolated so they can work different issues at
once):

```bash
git fetch origin {base-branch}
git worktree add -b {JIRA-SLUG} {worktree-dir} FETCH_HEAD
```

Fetching into `FETCH_HEAD` (rather than referencing `origin/{base-branch}` directly)
works even when the remote has no fetch refspec configured for remote-tracking
branches — a real condition in some bare-repo setups, where `origin/{base-branch}`
is not a resolvable ref. It also avoids writing to the shared base-branch worktree,
so it can't race with another concurrent session working there.

If `git fetch origin {base-branch}` itself fails (offline, no such branch upstream),
fall back to the local branch:

```bash
git worktree add -b {JIRA-SLUG} {worktree-dir} {base-branch}
```

Then work inside the worktree for all subsequent implementation. If the worktree
path already exists (e.g. a resumed pickup), reuse it instead of recreating.

Default base branch is `master`. If the user named a different base branch during
the session, substitute accordingly.

### Phase 5: IMPLEMENT

#### If TDD applies (Angular or Spring Boot production code is changing)

Follow the `tdd` skill. Read its SKILL.md now if you have not already.

Key rules:
- Vertical slices: one behavior at a time, tracer bullet first.
- Write one test → write minimal code to pass it → repeat.
- No refactoring until all behaviors are implemented and tests pass.
- Tests verify behavior through public interfaces, not implementation details.

#### If TDD does not apply (config, deployment, SQL, or docs only)

Implement directly with Edit/Write/Bash. Verify correctness with the appropriate
check (e.g. `./gradlew clean test`, Helm dry-run, static analysis).

### Phase 6: SELF-REVIEW

Review your own diff before handing the branch over — a reviewer's first pass,
run by you:

1. From the worktree, invoke Claude Code's bundled `code-review` skill (via the
   Skill tool) at medium effort against the branch diff
   (`git diff {base-branch}...HEAD`, using the local base branch — same reasoning
   as Phase 4's `FETCH_HEAD` approach). Check the available-skills list for an
   entry literally named `code-review` before invoking it. If it is not listed, do
   not substitute a differently-scoped skill (e.g. `simplify`, which reviews
   quality only, not correctness) — go straight to the manual correctness pass.
   `code-review` is a plugin slash command, not a Skill-tool-invocable skill, so
   this Skill-tool call is expected to fail every time; the manual correctness
   pass is the normal path here, not a rare fallback. It is a materially weaker
   substitute — an unstructured read-through, not the real multi-angle,
   verified methodology — so it does not satisfy this gate on its own.
2. Triage the findings:
   - **Confirmed correctness bugs** — fix now. When the fix touches Angular or
     Spring Boot production code, go through `tdd` (failing test first); re-run
     the tests after.
   - **Judgment calls** (design trade-offs, scope questions) — surface to the
     user; never silently expand scope beyond the Phase 3 alignment.
   - **False positives / out-of-scope findings** — skip, with a one-line note.
3. Summarize the outcome in a few lines: fixed, surfaced, skipped.
4. Tell the user plainly that only the manual correctness pass ran (not the real
   `/code-review` methodology — the agent cannot trigger slash commands itself),
   and recommend they run `/code-review high` (or `xhigh`/`max` for a
   higher-stakes change) themselves before opening the PR for peer review. Do not
   imply the manual pass was equivalent.

This gate is cheap insurance before the coworker review — it does not replace
the team's PR review, and on its own (without the user separately running
`/code-review`) it is weaker than a real review pass.

### Phase 7: DOCS

After implementation:

1. Ensure `.claude/context/CONTEXT.md` reflects any new domain terms introduced
   (same resolved root as Phase 1).
2. Do NOT write ADRs or product documentation (root `README.md`, `docs/`, module
   READMEs). Those are written by the lead's `update-docs` runs, which detect
   this issue's changes from the merged commits. If the work embodied a decision
   that is hard to reverse, surprising without context, and a real trade-off,
   say so in the session wrap-up so the user can flag it for the next
   `update-docs` run.
3. Confirm the worktree is in a reviewable state: tests pass, static analysis
   clean, no uncommitted changes. Optionally remind the user they can remove the
   worktree after the branch is merged:
   `git worktree remove {worktree-dir}` (the directory resolved in Phase 1).
4. If this session produced retro signal — corrections to your assumptions,
   context the user had to re-explain, or grilling questions the issue text
   should have pre-answered — offer `/skill-retro` before closing. Skip the
   offer if there was no signal.

## General conduct

- Never start Phase 4 without explicit alignment confirmation from Phase 3.
- The branch name is always the exact Jira slug — no modification, no lowercasing.
- If the issue scope is large enough to span multiple PRs, say so in Phase 3 and
  propose splitting before creating the worktree. If scope is discovered mid-grilling
  and split off rather than folded in, capture it in `.claude/wrap-up/IMPLEMENTATION-HANDOFF.md`
  (same format `wrap-up/IMPLEMENTATION.md` defines) so it can seed its own
  `/pickup-issue` session later, rather than losing the context.
- The user can jump phases backward at any time. Honor it, then resume.
