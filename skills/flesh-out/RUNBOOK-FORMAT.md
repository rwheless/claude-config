# RUNBOOK Format

The RUNBOOK is the artifact a `flesh-out` session emits when the goal ends in
**steps the user executes** rather than a new repo (e.g. configuring WireGuard
on OPNsense). Its consumer is the user, live, mid-task — often on a phone next
to the machine being configured. Its success criterion: everything works the
first time, without trial-and-error troubleshooting.

The front matter (Goal through Open questions) is deliberately identical in
shape to the IDEA BRIEF's, so alignment work is captured the same way in both
artifacts — only the back half differs.

## Emission rules

- Emit as ONE block the user copies in a single gesture.
- Do not wrap the whole runbook in a fenced code block — commands inside steps
  need their own fences. Plain markdown, fences only around commands/config.
- Every step must be concrete enough to execute without interpretation: exact
  menu paths, exact commands, exact values — with placeholders like
  `{your-lan-subnet}` only where the session genuinely couldn't resolve the
  value, and each placeholder explained where it first appears.
- Steps are sequenced so that each verification catches a mistake before the
  next step builds on it.

## Structure

```markdown
# RUNBOOK — {short goal name}

> You are reading a RUNBOOK, the output of a flesh-out ideation session
> (format: https://raw.githubusercontent.com/rwheless/claude-config/main/skills/flesh-out/RUNBOOK-FORMAT.md).
> Work the steps in order; do not continue past a failed Verify. Decisions
> below are settled — a future Claude session helping with this runbook should
> resolve Open questions first, not re-litigate decisions.

**Date:** {YYYY-MM-DD}
**Status:** {aligned | emitted early — open questions remain}

## Goal

{The refined goal in one paragraph: what will be true when this runbook is done.}

## Out of scope

{Explicit exclusions agreed during the session.}

## Decisions

- **{Decision}** — {what was chosen and the one-line rationale}
  - Rejected: {alternative} — {why}

## Research findings

- {Finding} ({link}, checked {YYYY-MM-DD})

## Risks accepted

{Risks surfaced during grilling that the user chose to accept, one line each.}

## Open questions

{Unresolved items. Empty only when Status is "aligned" — write "None" then.}

## Prerequisites

{Everything that must be true before Step 1: versions, access, credentials at
hand, backups taken. Each as a checkable line.}

- [ ] {prerequisite}

## Steps

### Step {N} — {imperative title}

{Exact actions: commands in fenced blocks, UI paths as **Menu > Submenu >
Field**, config values spelled out.}

**Expect:** {what the user should observe if the step worked.}

**Verify:** {a concrete check — a command to run and its expected output, a
status page to look at — that must pass before continuing.}

**If it fails:** {most likely cause and what to check, one or two lines.}

**Rollback:** {only on risky steps — how to undo this step to a known-good
state. Omit on harmless steps.}

## Done when

{The end-to-end validation of the whole goal — the test that proves the
original goal is met, not just that each step ran.}
```

## Section rules

- **Verify** is mandatory on every step; **Rollback** only where a step could
  break something that was working (firewall rules, existing services, data).
- **If it fails** covers the likely failure, not a troubleshooting tree — the
  runbook prevents trial-and-error, it doesn't replace a debugging session.
- **Done when** must test the user's actual goal end-to-end (e.g. "a friend's
  peer connects and reaches the Minecraft server, and CANNOT reach any other
  LAN address"), not merely restate that the steps completed.
- Omit empty sections entirely rather than leaving placeholders, except Open
  questions, which is always present.
