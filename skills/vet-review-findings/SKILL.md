---
name: vet-review-findings
description: Re-screen code review findings (from Claude Code's bundled /code-review plugin, a manual review pass, or pickup-issue's Phase 6 self-review) for one specific failure mode - a finding that adds, keeps, or patches defensive/error-handling/validation/guard code without first checking whether the state it guards against can actually occur. Traces real reachability through the codebase's actual callers, controllers, and security filters before accepting any such finding, and flips unreachable-guard findings from "patch this" to "delete this." Use whenever the user has just run /code-review (or any review pass) and is about to apply fixes, or explicitly asks to check findings for unnecessary defensive code, dead guards, or "slop." Also trigger if the user says something like "is this guard actually reachable" or "does this null check ever matter."
---

# vet-review-findings

Code review tools are good at spotting that a piece of defensive code has a bug
in it - a null check that itself NPEs, a validation branch with an off-by-one,
a guard whose error message is wrong. They are much worse at asking the prior
question: does this code need to exist at all? A review that only patches
guards, never questions them, quietly accumulates defensive code that guards
against states nothing in the running system can ever produce. That code
still costs someone reading time, test-writing time, and cognitive load
forever after - it just never announces itself as a bug, because it never
fires.

This skill is that prior question, applied specifically to findings that
touch defensive code: null checks, type guards (`instanceof`, casts),
validation branches, try/catch blocks, `Optional` handling, "should never
happen" comments, bounds checks. It does not re-review everything else a code
review might have found - real bugs, simplification opportunities, reuse
findings, and so on are out of scope here; take those at face value.

## Scope

**In scope:** any finding - from `/code-review`, from a manual read-through,
from `pickup-issue`'s Phase 6 self-review - whose recommendation is to add
new defensive code, keep existing defensive code, or patch/fix defensive
code in place. This includes findings that don't use the word "guard"
explicitly but amount to one: "this cast could throw," "handle the null
case here," "this should check for an empty list first."

**Out of scope:** findings about actual business logic bugs, simplification/
reuse/efficiency findings that don't touch defensive code, and anything
already reverted or skipped by the user. Don't second-guess those here - if
something looks off, say so briefly, but the reachability check is the whole
point of this skill and everything else just dilutes it.

## Input

Either works:

1. **Findings already in hand** - the user pastes or describes `/code-review`
   output (or any other review's output). Work from what they give you.
2. **No findings yet** - the user wants a fresh pass. Run `git diff
   {base-branch}...HEAD` (or whatever range they specify) and read it
   yourself, looking specifically for defensive-code additions or
   modifications. This is a narrower read than a full code review - you are
   hunting for one pattern, not doing general correctness review.

## Workflow

For each in-scope finding:

### 1. Name the guarded condition

State plainly what state or input the code defends against, in one sentence.
If you can't state it precisely, that's itself a signal the guard may be
vague/speculative rather than answering a real observed failure.

### 2. Trace reachability

This is the core of the skill, and it has to be real tracing, not a plausible-
sounding guess. Find every path that reaches the guarded code:

- Grep for callers of the method/class directly.
- If the code sits behind a controller endpoint, check what's guaranteed by
  the security/auth layer before the request arrives (filters, interceptors,
  `@PreAuthorize`-style annotations, gateway config) - a lot of "what if the
  principal is null" guards are already impossible because something upstream
  either populates it correctly or rejects the request first.
- If the code runs in a scheduled job or event listener, check what triggers
  it and whether that trigger can supply the guarded-against state.
- Check existing tests for the class - a test that already exercises (or
  explicitly documents) the guarded state is evidence the state is considered
  reachable by the team, even if you can't find a live caller.

Cite the grep/read that supports your conclusion. "I looked and didn't
immediately see a caller" is not a reachability verdict - if you can't trace
it to a clear yes or no, that's the ambiguous case (step 3c), not a guess.

### 3. Decide

- **(a) Unreachable** - nothing in the codebase can put the code in the
  guarded state through normal operation. Override the original finding:
  recommend deleting the guard (and any now-unnecessary tests written for
  it), not patching it. State the evidence from step 2 as the justification,
  the same way you'd justify any other finding.

  Say this as an affirmative action, not just a rejection of the original
  patch. The original finding is often scoped narrower than the guard itself
  - "fix the NPE in this error message" rather than "should this guard exist"
  - and it's tempting to answer only the narrow question ("don't add that
  null check") and stop there. That's not enough: once you've established the
  guarded state can't occur, leaving the now-provably-dead guard in place is
  exactly the "slop" this skill exists to catch. Name the specific lines to
  remove (the guard's `if`/`throw`, and any check feeding into it, like a
  preceding `authentication == null` guard on the same unreachable path), not
  just "don't apply this fix."

  Don't let "but it could catch a future regression" talk you out of this.
  An unreachable guard can always be reframed as a tripwire for some
  hypothetical later change (a filter reordered, a config flag flipped) - that
  argument applies to literally any dead code, which is exactly why it isn't a
  real exception. This codebase's own working principle is to trust tests and
  the type system to catch future regressions rather than carry speculative
  runtime checks forward on the chance they might someday matter (see the
  project's "don't add error handling for scenarios that can't happen"
  guidance). If a future change genuinely reintroduces the guarded state, the
  right response is to add the check back then, with a real caller to point
  to - not to keep it on standby now. Recommend deletion; don't soften it into
  a question for the user to weigh unless reachability itself (not the guard's
  hypothetical future value) is genuinely unresolved per step 3c.
- **(b) Reachable** - a real caller, an untrusted boundary, or a documented
  edge case can produce the guarded state. Leave the original finding's
  recommendation as-is. Note the reachability evidence briefly - it's useful
  confirmation that the finding is worth acting on.
- **(c) Ambiguous** - reachability can't be resolved by reading code and
  tests alone (e.g. it depends on a caller outside this repo, or on
  production data shape you can't inspect). Don't guess either way. Surface
  it to the user as an open question with what you were and weren't able to
  determine, and let them decide.

## Output

A short list, one entry per finding vetted, in this shape:

```
### {finding, in one line}
- Guarded condition: {what state/input this defends against}
- Reachability: {(a) unreachable / (b) reachable / (c) ambiguous} - {evidence, with file:line citations}
- Recommendation: {delete guard + tests / keep original recommendation / ask the user}
```

Findings outside this skill's scope (not about defensive code) can be passed
through untouched with a one-line note that they weren't re-screened.

## Why this matters here

This exists because the project's own working principle - don't add
validation, fallbacks, or error handling for scenarios that can't happen -
applies at review time just as much as at write time, but the review tools
available (`/code-review`, a manual pass) don't check for it. They're good at
"is this code correct" and weaker at "should this code exist." This skill is
the missing second question, scoped narrowly enough to run in a few minutes
right after a normal review pass.

If a finding keeps surfacing the same unreachable-guard pattern across
multiple sessions in the same codebase, that's worth mentioning in
`/skill-retro` - it may mean the team's actual convention (e.g. "controllers
never trust the principal type without a runtime check, even though the
security filter guarantees it") should be written down somewhere durable,
rather than re-discovered every time.
