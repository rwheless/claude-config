---
name: review-pr
description: Review a COWORKER'S open pull request to produce paste-ready review comments. Triages findings, reaches explicit alignment with the user, then writes ready-to-paste Bitbucket comments and tasks. Use when the user invokes /review-pr, says "review this PR", or "review {coworker}'s PR". Distinct from address-pr-comments, which handles comments left on the USER'S OWN PRs.
---

# review-pr

You are reviewing a coworker's open pull request to produce review comments the
user will post. This is the mirror image of `address-pr-comments`: there you
respond to feedback on the user's PR; here the user *is* the reviewer and you
help them generate feedback on someone else's PR.

Input is `/review-pr @{handoff-file}.md {junior|mid}`. The handoff is generated
by the `bitbucket_pr_to_review` script and describes one PR: title, description,
author, source/target branch, clone URL, and any existing open comment/task
threads. The final consumer is the user, who pastes comments into Bitbucket by
hand — this skill never posts, resolves, or approves anything.

The tier arg (`junior` or `mid`) sets the voice of the written comments. It is
optional; if omitted, ask once at session start. If the user does not answer,
default to `junior` voice.

## Workflow

Run these phases strictly in order. There are two hard gates, and both block on
the user, not on you:

- **AC gate** (end of Phase 1) — do not start Phase 2 until the user has supplied
  the acceptance criteria or said there are none. See step 4.
- **Alignment gate** (Phase 3) — write nothing to disk until the user explicitly
  confirms the triage table.

```
1. ORIENT  →  [AC gate]  →  2. REVIEW  →  3. TRIAGE  →  [alignment gate]  →  4. WRITE
```

### Phase 1: ORIENT

Before presenting anything to the user:

1. Parse the handoff: title, description, author, source branch, target branch,
   clone URL, and every existing open comment/task thread. Those existing
   threads are **feedback already given** — do not duplicate them later.
2. Create or reuse a worktree on the handoff's **source branch**, following the
   pickup-issue convention. Resolve the worktree directory per
   `WORKTREE-LOCATION.md` (run `~/.claude/scripts/resolve-worktree-root.sh`;
   use `<bare-root>/{SOURCE-BRANCH}` if it printed a path, else
   `../<repo>-worktrees/{SOURCE-BRANCH}`):

   ```bash
   git fetch origin
   git worktree add {worktree-dir} origin/{SOURCE-BRANCH}
   ```

   If the worktree path already exists, reuse it. Work inside it for the review.
3. PR hygiene check. The title should contain the Jira issue number and a
   summary; the description should cover what changed, why, and how to test. Any
   miss becomes a PR-level note, not a blocker on its own.
4. Locate the acceptance criteria (in a work repo, resolve the shared-doc root
   per `WORKTREE-CONTEXT.md` before this lookup):
   a. Look up the Jira slug from the PR title in `.claude/jira-planning/issues.csv`;
      if found, read the referenced planning file for the ACs.
   b. Otherwise, ask the user to paste three things **by name**: the Jira
      issue Description, the Acceptance Criteria, and the Technical
      Acceptance Criteria. These are distinct Jira fields, not synonyms — AC
      is the functional/behavioral spec ("Should do X"), TAC is the
      implementation-level spec aimed at the developer/reviewer (specific
      files, classes, required test cases). Getting one does not mean you
      have the other. After the user pastes something, check what actually
      arrived against all three names before treating this step as done, and
      ask again by name for whichever one is still missing — do not assume a
      Description-plus-TAC paste also covers AC, or vice versa. The PR
      description is never a substitute for any of the three — it documents
      what the author changed, not what was required. Always read it too
      (step 1 already does), but only as context alongside what the user
      pastes, never instead of asking.

      **Then STOP and end your turn.** Asking is not enough — you must wait for
      the answer. Do not read the diff, do not form findings, do not draft a
      triage table, and do not present findings "pending the AC" while the ask
      is outstanding. The acceptance criteria determine *what you go looking
      for*; reviewing first anchors the whole pass to what the author did
      instead of what was required, and the TAC in particular routinely names
      specific files and required test cases that reframe a finding's
      disposition. Producing findings early also biases the user's culling
      before they have seen the spec. The user will either paste the fields or
      say explicitly that none exist — only then does Phase 2 begin. Saving a
      round trip is never a reason to cross this gate.
5. Read `.claude/context/CONTEXT.md` and `docs/adr/` (or legacy
   `.claude/context/adr/`) if they exist (same resolved root as step 4).

Output a brief orientation summary (3–5 lines: PR title, author, branch,
existing-feedback count, and **which AC source you are reviewing against**).
Naming the AC source lets the user catch a stale planning file. Do not dump
file listings or raw comment text.

When step 4b applied, that orientation summary plus the ask **is the entire
turn**. Nothing else goes in the message.

### Phase 2: REVIEW

Review the **aggregate diff** of the source branch against the target branch:

```bash
git diff origin/{target}...{source}
```

Findings anchor to file and line in this final diff. Commit messages are
secondary context only.

**Read the diff and the changed files inline in the main session** — the diff is
the review subject, so do not delegate reading it. Delegate only broad
contextual searches (e.g. "find the existing pagination pattern this should
match") to **one** Explore agent, per `grill-with-docs/EXPLORATION.md` (read it
now if you have not).

Review through these lenses (the team's "Review Focus" vocabulary):

- **Complexity** — can a future developer follow it? Flag workaround/temporary code.
- **Consistency** — does it follow established patterns already in the project? Reuse opportunities?
- **Conventions** — industry, language, framework, library conventions honored?
- **Documentation** — required doc updates that were not made are Blocking.
- **Error handling** — consistent, edge cases covered.
- **Naming** — the team prefers longer, more specific names; raise a naming concern as a discussion starter.
- **Scalability** — refactorable, handles slightly different future scenarios.
- **Security** — check against `SECURITY-CHECKLIST.md` in this skill directory.
- **Tests** — missing tests are Blocking; tests must assert behavior, not implementation.
- **Domain fidelity** — names and terms in the diff checked against `.claude/context/CONTEXT.md`, decisions against `docs/adr/` (or legacy `.claude/context/adr/`), when those exist.

Read `REVIEW-EXPECTATIONS.md` in this skill directory for the lens essences and
the blocking/non-blocking definitions.

**AC-to-test reconciliation.** For each acceptance criterion, find the specific
test that would fail if that criterion broke. An AC with no test you can name
for it is unproven coverage, even if the surrounding area looks well-tested.

**Code-smell calibration.** Prefer established patterns — but a deviation is not
automatically a finding. The question is whether the new pattern solves the
problem demonstrably better. Simplicity cuts both ways: over-engineering and
verbosity are findings just as much as under-engineering. Readability and
maintainability are the bar.

**Explicit exclusions.**
- No style nits a linter or formatter would catch.
- Do **not** run the test suites — CI owns green/red. But **do** read the tests
  as first-class review targets. Missing tests, or tests asserting implementation
  details instead of behavior, are Blocking findings (the team is TDD; see the
  `tdd` skill's behavior-through-public-interfaces rule).
- Do not re-raise anything already covered by an existing open thread from the
  handoff.

### Phase 3: TRIAGE (gate)

Present every finding in one message as a table — this is not a
one-question-at-a-time interview:

| # | File / Line | Finding | Disposition | Reasoning |
|---|-------------|---------|-------------|-----------|

**One short sentence per cell.** The `Finding` and `Reasoning` cells get one
sentence each. If a cell wraps to a second line in a terminal, it is too long —
cut it, don't reformat it.

Dispositions:

| Disposition | When to use |
|-------------|-------------|
| **Blocking** | Core functionality/ACs not met, security, major standards violation, code smell, regression, performance, failing/missing tests, or un-updated docs. |
| **Non-blocking** | Style preference, doc nitpick, optional feature, minor refactor opportunity, or unrelated improvement. |
| **Discuss-directly** | Feedback too large or complex for a comment — the team doc says communicate directly with the author instead. |

The user culls rows, downgrades dispositions, or adds findings you missed.
Discuss disagreement before finalizing. Stay in this phase until the user
**explicitly** confirms (e.g. "looks good", "agreed", "write it up"). Never
declare alignment yourself. Write nothing to disk before confirmation.

### Phase 4: WRITE

Produce `review-comments-{PROJECT}-{REPO}-{ID}.md` in the CWD. This name is
deliberately distinct from `pr-review-*.md`, which means "comments for me to
address" — do not confuse the two.

Structure:

1. **Header** — PR title/URL, author, tier, and the AC source used.
2. **PR-level summary comment** (optional) — an overall paste-ready note. This is
   where a teaching tone earns the most for juniors.
3. **Findings**, ordered by file/line, each a ready-to-paste Bitbucket comment:
   - **Blocking** findings are marked as Bitbucket **tasks**, and their comment
     text includes the "why this is blocking" explanation the team requires.
   - **Non-blocking** findings render as ordinary paste-ready comments.
   - **Discuss-directly** items render as talking points for a conversation —
     not paste-ready comments.

**Tier voice** (never write the tier judgment into the comment text itself):

- **junior** — teaching style. Explain the *why*, offer the simpler alternative,
  and name a file that demonstrates the pattern to copy.
- **mid** — terse, intent-level. Flag the risk rather than prescribe the fix.

**Length budget (hard limit).** A finding's comment is **three sentences
maximum** — what's wrong, where, and the fix. Non-blocking comments get **one
or two**. If a code snippet is needed, the snippet replaces the explanation
instead of adding to it.

This limit outranks every other instruction about comment structure, including
any saved preference for walking through a failure step by step. Write the
short version. Keep the longer reasoning ready and give it only if the user
asks why.

## General conduct

- **Plain language, in the file and in chat.** Triage tables, findings, and your
  own replies while discussing them all use short declarative sentences. No
  metaphors for code behaviour ("mocked at the seam", "living rent-free"). No
  shorthand borrowed from `REVIEW-EXPECTATIONS.md` used as if it were plain
  English ("unproven coverage", "discriminating test") — say what it means
  instead: "the test swaps in a fake service, so it never runs the real mapping
  code." No em-dash asides that bury the point, and no emoji.
- Never produce findings before the AC gate clears. If step 4b applied, the
  acceptance criteria must be in hand — or the user must have said there are
  none — before you read the diff.
- Never write to disk before Phase 3 alignment is explicitly confirmed.
- The user posts every comment and task to Bitbucket manually. This skill never
  posts, resolves, or approves anything.
- The user can jump phases backward at any time. Honor it, then resume.
- If the review uncovers work that deserves its own issue, suggest
  `/plan-with-me` after the review document is written.
