---
name: grill-with-docs
description: Interview the user one question at a time about a planned change until shared understanding is reached, grounding every question in the actual codebase and the repo's domain glossary (CONTEXT.md), and growing that documentation inline as decisions crystallize. Use during the GRILL phase of plan-with-me, or whenever the user wants their plan stress-tested, says "grill me", or wants to align on a design before building. Adapted from Matt Pocock's grill-with-docs skill (github.com/mattpocock/skills).
---

# grill-with-docs

Interview the user relentlessly about every aspect of the plan until you reach a
shared understanding. Walk down each branch of the design tree, resolving dependencies
between decisions one by one.

## Scope gate

Stop grilling and recommend `/deconstruct` when ANY of these holds:

- The scope spans more than one bounded context (per `.claude/context/CONTEXT-MAP.md`
  when it exists).
- The scope bundles independent sub-goals that share no resolved design decision.
- The running effort estimate exceeds **72 hours** of implementation work. Produce a
  rough total at ORIENT time (for work repos, junior-developer hours per
  `jira-formats` estimation rules) and revise it as grilling reveals scope — re-check
  the gate whenever the number grows.

For personal repos (no `jira-formats` estimation rules apply), produce a rough ad hoc
hour estimate by decomposing the proposed scope into its natural sub-parts and
eyeballing implementation time for each — precision doesn't matter, only whether the
running total is comfortably under or clearly over the 72-hour line.

Don't try to grill an oversized scope to completion in one sitting.

## Core rules

1. **One question at a time.** Ask, wait for the answer, then continue. Never batch
   questions.
2. **Always include your recommended answer**, with a one-or-two-sentence rationale.
   The user decides; you advise.
3. **If the codebase can answer the question, explore the codebase instead of
   asking.** Only bring questions to the user that require human judgment or domain
   knowledge the repo doesn't contain. Follow `EXPLORATION.md` in this skill
   directory: if you can name the exact file, read it inline; if you'd have to
   search, delegate to one Explore agent and require its report format. When you
   state how something currently works, cite the file you saw it in. If the user's
   description contradicts the code, surface the contradiction immediately:
   "You said X, but `MasterService.java` does Y — which is right?"
4. **Track open branches.** When an answer spawns new questions, note them and resolve
   them in dependency order — don't lose threads.
5. **Stop when the user says the grilling is done**, not before and not on your own
   judgment.
6. **Re-ground periodically.** After every 10th question, and after returning from
   any exploration detour, re-read the Core rules section of this file before
   asking the next question. Long sessions decay discipline; this restores it.
7. **Flag scope expansion in your own recommendations.** If your recommended answer
   introduces something the user didn't ask for (a new status code, a new
   dependency, new client-side infrastructure, a new pattern not already in the
   codebase), say so explicitly and name the literal minimal-scope alternative
   alongside it. "More correct" and "more work" are different axes — let the user
   see the cheap option, don't make them notice its absence and ask for it back.

## Stack guidance

Stack-specific exploration guidance lives in supporting files (currently
`STACK-WORK.md` for the Oracle/Spring Boot/Angular work stack). Orchestrating
skills say which to read. If invoked directly on a work-stack repo, read
`STACK-WORK.md` now.

## Question priorities

Probe, in rough order, whichever of these the plan leaves ambiguous:

1. **Domain language** — do the terms in the plan match CONTEXT.md? Are any terms
   fuzzy, overloaded, or colliding with existing definitions?
2. **Data model** — cardinality, nullability, uniqueness, deletion behavior
   (CASCADE / SET NULL / RESTRICT), lifecycle/status semantics (manual vs derived),
   migration impact on existing rows.
3. **Boundaries & contracts** — endpoint shapes, pagination/filtering semantics,
   error responses, what the front end shows in failure states.
4. **Scope edges** — what is explicitly out of scope; concrete edge-case scenarios
   ("a master exists in Canvas but its term just ended — included or not?").
5. **Sequencing & dependencies** — what blocks what; where mock data is needed
   because a dependency isn't built yet.

Invent concrete scenarios to stress-test relationships and force precision about
boundaries between concepts.

## Documentation duties (the "docs" in grill-with-docs)

### CONTEXT.md — the domain glossary

Look for `.claude/context/CONTEXT.md` in the repo. In a work repo, resolve the
shared-doc root per `WORKTREE-CONTEXT.md` before any read or write in this
section — `.claude/context/` is gitignored there. If `.claude/context/CONTEXT-MAP.md` exists, the repo has
multiple bounded contexts — follow the map to the right `CONTEXT.md`.

- When the user uses a term that conflicts with the glossary, call it out
  immediately and resolve which meaning wins.
- When fuzzy language appears, propose a precise canonical term.
- **When a term is resolved, update `.claude/context/CONTEXT.md` right then** — don't batch.
- `.claude/context/CONTEXT.md` is a glossary ONLY: terms, definitions, relationships between terms.
  No implementation details, no specs, no decisions. Format per `CONTEXT-FORMAT.md`
  in this skill directory.
- If no `.claude/context/CONTEXT.md` exists, create it when the first term is resolved — not before.

### ADRs — record decisions, don't write ADRs

Do not write ADRs during grilling. Planning-time ADRs prescribe code that doesn't
exist yet and go stale on contact with implementation. In work repos, ADRs are
written post-implementation by the `update-docs` skill, from the concrete merged
code (format and lifecycle per `docs-formats/ADR-FORMAT.md`).

When a decision made during grilling passes the ADR test — (1) hard to reverse,
(2) surprising without context, (3) a real trade-off — record it in the planning
session's `OVERVIEW.md` under `## Decisions made this session` (format per
`plan-to-jira`): what was decided, the alternatives rejected and why, and the
issue slugs it touches. That entry is the rationale source `update-docs` harvests
when it writes the ADR later. In sessions with no planning folder (e.g. pickup
flows), the decision context lives in the Jira issue text itself.

Exception: orchestrating skills that explicitly instruct inline ADR writing for
their own flows (the personal pickup/planning skills, deconstruct's
architecture-seam exception) still follow their own SKILL.md instructions, using
`docs-formats/ADR-FORMAT.md`.

## Tone

Be direct and economical. Questions should be sharp enough that answering them
genuinely advances the design. Don't pad with praise or restate the user's answer
back at length — log the decision in one line and move to the next question.
