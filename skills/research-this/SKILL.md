---
name: research-this
description: Manual-only research skill. Investigates a proposed technical task or feature idea (not a bug fix) inside the current project and produces a paste-ready Jira-formatted document presenting a few ranked design options for a senior developer to choose between. May grill the user to align on scope, and can scan sibling LMS repos for precedent -- always asks first. Only runs when the user explicitly invokes /research-this; never triggers automatically from another skill's workflow.
disable-model-invocation: true
argument-hint: "<Some statement or coding idea>"
---

# research-this

Input: everything after `/research-this` -- a statement or coding idea to research.
This is early-stage research, not implementation and not planning. The output is a
document a senior developer reads to pick a direction; nothing here writes code or
Jira issues directly.

## Scope gate

- If the input describes broken/unexpected behavior, it's a bug, not a research
  target -- say so and point at `debug-problems` (or `pickup-issue` if the fix is
  already understood) instead of proceeding.
- If the input is too bare to research (a title with no detail), ask ONE question
  to get a one- or two-sentence description of the idea. Save deeper questions for
  Phase 3 -- don't grill yet.

## Phase 1: Orient in this project

1. Read `.claude/context/CONTEXT.md` (domain glossary) and `docs/adr/` (or legacy
   `.claude/context/adr/`) if present. Resolve the shared-doc root per
   `../WORKTREE-CONTEXT.md` before reading either.
2. Read `.claude/context/ORIENTATION.md` if present for a repo map; it narrows
   what you need to explore fresh.
3. Delegate codebase exploration to ONE Explore agent per
   `../grill-with-docs/EXPLORATION.md`. Fold in the relevant stack bullets from
   `../grill-with-docs/STACK-WORK.md` (Spring Boot / Angular / Oracle conventions
   for this stack). Brief the agent with the proposed idea and ask specifically:
   what existing code/patterns are relevant, what would plausibly need to change,
   and what's the closest existing precedent already inside this repo.
4. Summarize what you found for the user in 5 lines max, then continue -- don't
   dump file listings.

## Phase 2: Ask before scanning sibling repos

Do NOT scan outside the current project directory without asking first. If other
LMS applications live as sibling checkouts (e.g. `~/GitRepos/halibut`, `mizzen`,
`skipjack`, `lochness`, `octopus`, `canvas-api`, `facultyportfolio`, etc.), ask the
user by name whether to check any of them for precedent on this idea:

"Want me to check any sibling repos for how they've handled this? [list the ones
that look domain-relevant]"

- **If yes:** delegate ONE Explore or general-purpose agent per repo the user
  approves (or one agent covering a short approved list). Use the same report
  contract as `EXPLORATION.md`. Fold any precedent found into the option
  write-ups in Phase 4, citing repo name + file path.
- **If no or unsure:** skip it. Note in the final document that cross-repo
  precedent was not checked, so the senior developer knows the gap exists rather
  than assuming it was covered.

## Phase 3: Grill only what changes the options

Not every idea needs grilling. Grill only when the answer would materially change
which options exist or how they'd be scoped. Follow `../grill-with-docs`'s core
rules: one question at a time, always give a recommended answer, explore the code
instead of asking when the code can answer, re-read the core rules every 10
questions.

Grill toward:
- The actual problem or opportunity, and why it matters now.
- Constraints: data volume, performance, existing consumers of anything touched,
  security/compliance.
- What's explicitly out of scope.
- Quality bar / who implements it (junior vs mid-level) -- affects how
  prescriptive the level-of-effort write-up needs to be, per `../jira-formats`
  tier calibration.

Stop when the user says it's done, or as soon as you have enough to draft options
that genuinely differ from each other. Log every resolved decision verbatim --
it goes into the document's "Decisions made during research" section unchanged.

If grilling reveals the idea is actually multiple unrelated capabilities bundled
together, say so and suggest researching them separately rather than forcing one
document to rank options across unrelated problems.

## Phase 4: Draft the options

Produce 2-4 solution options, ranked most to least practical, per
`RESEARCH-FORMAT.md`. "Don't do this" is a legitimate option (usually ranked
last, but rank it first if it's genuinely the best call) when the payoff doesn't
justify the effort, the idea conflicts with an existing ADR, or it's not
practical given the current architecture -- say why plainly, don't soften it to
avoid disappointing the reader.

Rules for every option:

- **Follow existing convention over inventing new patterns.** Default to
  whatever this repo already does for similar problems (same principle as
  `STACK-WORK.md`'s "existing conventions over invention"). If an option
  requires a new dependency, a new architectural pattern, or something not
  already established in this codebase, flag that explicitly as a deviation --
  don't bury it.
- **Conservative, standard Spring Boot / Angular style.** No speculative
  abstractions, no frameworks not already in use, no gold-plating. A option that
  needs a new generic framework to solve one concrete problem is a flag, not a
  feature.
- Each option must cover: architecture (which layers -- Angular / Spring Boot
  service / repository / DB), concrete classes and files (name real existing
  ones from exploration, and name the new ones needed, with their layer),
  level of effort in hours, and Jira shape.
- **Level of effort** uses the `../jira-formats` Original Estimate rules: hours
  on a junior-developer basis, divisible by 3.
- **Jira shape**: state whether the option fits one ticket with no sub-tasks
  (estimate <= 6h per `../jira-formats` sub-task sizing), needs sub-tasks under
  one parent issue (> 6h, each sub-task sized <= 6h), or needs multiple parent
  issues under an epic (mirror `plan-with-me`'s heuristic: more than ~3 parent
  issues is a good epic candidate). Every option needs enough detail here that
  its write-up could become a handoff document for creating the real Jira
  issue(s) later -- concrete files/classes/tables, not just intent.
- Read `../jira-formats/SKILL.md` now if it has not been read yet this session,
  so wiki-markup formatting in the output matches exactly.

## Phase 5: Write the document

Follow `RESEARCH-FORMAT.md` exactly for structure and formatting. All prose
content uses Jira wiki markup conventions from `../jira-formats` (ASCII only --
no arrows, em dashes, or smart quotes; `*bold*`; `{{monospace}}`; `h3.` headings;
`{code:lang}` blocks) since the reader may paste sections straight into a Jira
issue.

Save to `.claude/research/{YYYY-MM-DD}_{topic-slug}.md`. This directory is
gitignored the same way `.claude/jira-planning/` is in this stack -- resolve the
shared-doc root per `../WORKTREE-CONTEXT.md` first if the session is running in a
worktree, so the file lands in one physical place instead of a copy per worktree.

Tell the user the saved path and give a two- or three-line spoken summary of the
ranked options. Do not paraphrase the full document back in chat -- the file is
the deliverable.

## Tone

Inform, don't sell. Present tradeoffs honestly, including for the option you'd
personally pick. The senior developer decides; this document exists so they
decide with real information instead of a blank page.
