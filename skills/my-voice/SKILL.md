---
name: my-voice
description: Rewrite a wordy, technical-manual-sounding statement (or a whole file of them, e.g. a review-comments-*.md produced by review-pr or a pr-review-*.md produced by address-pr-comments) into Ron's own casual, human voice. Manual-only — invoke explicitly via `/my-voice <text>` or `/my-voice <path>`. Never run this automatically as part of another skill's workflow, even when that skill just produced text this would improve; only run it when the user directly invokes /my-voice.
---

# my-voice

Claude's default prose reads like a technical manual: it hedges, over-explains,
restates the obvious, and reaches for formal connective tissue ("Since X, can
you Y rather than Z" instead of just "can you Y?"). This skill rewrites that
into how Ron actually talks — direct, a little informal, first-person, and
willing to state a fact instead of building up to it.

**This skill only runs when explicitly invoked as `/my-voice ...`.** Other
skills (`review-pr`, `address-pr-comments`, etc.) must never call into this
one automatically, even on their own output — Ron decides when something needs
naturalizing, not Claude.

## Input handling

The argument to `/my-voice` is either inline text or a file path.

- **If it resolves to an existing file** (e.g. `review-comments-LDB-1205.md`,
  `pr-review-handoff-LDB-1203.md`), read it, rewrite the prose in place, and
  write the result back to the same path. These are scratch/handoff files Ron
  pastes elsewhere or reads once — there's no need to produce a second file
  alongside the original.
- **Otherwise**, treat the argument as the literal text to translate and reply
  with the translated version directly in chat.

When working on a file, rewrite every prose block that reads like Claude
wrote it — review comments, PR-level summaries, responses to reviewer
questions — but leave structural scaffolding alone: headers, the
Blocking/Non-blocking/Discuss-directly labels, file/line anchors, table
layout, and fenced code blocks. Report back a short summary of what you
changed, not a full diff dump.

## What "casual" means here

This is not just "make it shorter." A terse sentence that's still stiff hasn't
been fixed. The goal is a rewrite Ron would actually have typed himself. Work
from these tendencies, in rough order of how often they matter:

1. **Lead with the fact, not the setup.** Claude's draft often spends a clause
   or two building context before landing the point ("That's not what either
   the old `LocalDate` output or the new `LocalDateTime` output actually looks
   like — per the javadoc..."). Ron's version usually states the concrete
   problem first (what's wrong, where) and lets the reasoning trail behind it
   or drop out entirely if the reader can infer it from the code reference.

2. **Cut throat-clearing and hedges.** Phrases like "Since this PR is exactly
   what changes the format these examples document, can you update them here
   rather than leaving them wrong in a new way?" are doing a lot of work to
   say "can you update these to match?" Look for any clause whose only job is
   to justify the ask rather than state it, and cut it unless the reasoning is
   genuinely load-bearing for the reader to act correctly.

3. **First person, contractions, plain connectors.** "We went back and forth
   on this during ticket review" becomes "I considered this during ticket
   review." Swap "we" for "I" when the decision was Ron's individually (most
   review comments and PR responses are — check context rather than assuming).
   Use contractions. Replace formal connectors ("given that," "in favor of,"
   "which trades X for Y") with how someone would say it out loud.

4. **Compress restated reasoning, don't delete the reasoning itself.** Claude's
   drafts often explain a decision twice — once in the abstract, once via a
   concrete example that re-proves the same point. Keep one clear pass through
   the actual reasoning (Ron does still want to explain *why*, e.g. a design
   decision made during ticket review) but cut the parts that only restate a
   conclusion already reached.

5. **Never touch the substance.** Preserve every concrete detail the reader
   needs to act: file paths, line numbers, exact strings/values, code
   identifiers in backticks, and the Blocking/Non-blocking/Discuss-directly
   label. Don't soften a Blocking finding into something that reads as
   optional, and don't invent reasoning, names, or context (like "after
   reviewing it with Chris") that wasn't in the original — only compress and
   reorder what's actually there.

6. **Don't manufacture typos or forced slang.** Casual voice means natural
   phrasing, not deliberately broken spelling or try-hard informality. If the
   rewrite reads like it's performing casualness, it's overcorrected.

## Worked examples

**Example 1 — trims a redundant justification:**

> Wordy: "Non-blocking: `import static org.junit.jupiter.api.Assertions.*;`
> on line 32 already covers `assertFalse` — the explicit import on line 33 is
> dead weight left over from the refactor. Same fix as the wildcard-import
> comment above would clean this up too."
>
> Casual: "Non-blocking: This line is redundant because `assertFalse` is
> already pulled in with the wildcard above it."

The "same fix as the comment above" cross-reference and the refactor backstory
weren't load-bearing for the reader — cut.

**Example 2 — leads with the concrete problem, keeps the exact values:**

> Wordy: "Blocking: these example responses for `/api/blueprints`,
> `/api/blueprints/{id}`, and `/api/masters` show
> `courseLoadStartDate`/`effectiveDate` etc. as `"2026-06-20 17:58:53.0"`.
> That's not what either the old `LocalDate` output or the new `LocalDateTime`
> output actually looks like — per the `MasterResource` javadoc you updated in
> this PR, Jackson now serializes these as ISO-8601
> (`"2026-06-20T17:58:53"`). Since this PR is exactly what changes the format
> these examples document, can you update them here rather than leaving them
> wrong in a new way?"
>
> Casual: "Blocking: In docs/api-reference-course-load-service.md:47-48,
> 75-76, 89-90, 119-121 the dates in the JSON examples (like
> `"courseLoadStartDate": "2026-06-20 17:58:53.0"`) are out of sync with the
> new LocalDateTime format. The dates in the JSON examples should look like
> this: `"2026-06-20T17:58:53"`."

Note it swaps in the actual file:line anchors instead of describing the
endpoints in prose — when the file is known, point at it directly.

**Example 3 — a longer response to a reviewer question:**

> Wordy: "Good question — we went back and forth on this during ticket
> review. The original plan (from the ticket title) was actually a per-entity
> `XFilterSpec` class holding `roleFilter`, but that got superseded before
> implementation in favor of keeping it repository-local, matching the
> existing `termFilter` pattern already in this file.
>
> The other reason a fully generic version is awkward here: the join path to
> reach campusCode/departmentCode/collegeCode genuinely differs per repo —
> `CourseOfferingRepository` filters the root directly,
> `MasterRepository`/`LiveCourseRepository` each do one join to
> `courseOffering`, and this one (`BlueprintRepository`) does a double join
> through `Master`. A shared utility would need each caller to pass in its own
> join-path function, which trades the current "read top to bottom" clarity of
> authorization-critical logic for an extra layer of indirection, for what's
> ultimately ~15 lines of duplicated boilerplate per repo.
>
> Given that, we're keeping it as-is for now. Appreciate you flagging it
> though — if the duplication becomes a real maintenance pain once more
> roles/rules get added, extracting a shared helper is a reasonable follow-up
> at that point."
>
> Casual: "Good question — I considered this during ticket review and decided
> on this per-repository solution on purpose, matching the existing
> `termFilter` pattern. The original plan was actually a per-entity
> `XFilterSpec` class holding `roleFilter`, but that got overridden before
> implementation in favor of keeping it repository-local, matching the
> existing `termFilter` pattern already in this file.
>
> The other reason a fully generic version is awkward here: the join path to
> reach campusCode/departmentCode/collegeCode differs per repo —
> `CourseOfferingRepository` filters the root directly,
> `MasterRepository`/`LiveCourseRepository` each do one join to
> `courseOffering`, and this one (`BlueprintRepository`) does a double join
> through `Master`. A shared utility would need each caller to pass in its own
> join-path function per caller for not much savings."

Note this one keeps almost the full reasoning — it's genuinely the substance
of the answer — but still switches to "I," drops the closing paragraph's
softening ("Appreciate you flagging it though..."), and tightens the sentence
about the tradeoff.

Use these to calibrate the *amount* of compression, not as a literal
find-replace list — every input is different, and a one-line comment doesn't
need the same treatment as a multi-paragraph justification.
