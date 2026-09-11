# RESEARCH-FORMAT

Template for the document `research-this` produces. File-level structure uses
Markdown headers so the document is readable as a file; everything inside a
section's body uses Jira Wiki Markup (per `../jira-formats`) since the reader
may paste any section straight into a Jira issue description. No Unicode
special characters anywhere -- no arrows, em dashes, or smart quotes.

```markdown
# Research: {Idea title, imperative or noun phrase}

**Type:** Research / Design Options
**Date:** {YYYY-MM-DD}
**Requested scope:** {the idea as given, one line}
**Cross-repo precedent checked:** {Yes -- list of repos checked | No -- user declined | Not applicable}

## Problem / Opportunity

{Prose. What problem this solves or what opportunity it captures, and why it
matters now. Must use valid Jira Wiki Markup syntax -- *bold* for UI/routes,
{{monospace}} for code identifiers.}

## Decisions made during research

{Bulleted list of resolved decisions from grilling, verbatim, in the order
resolved. If grilling was skipped because the scope was already unambiguous,
write a single bullet saying so and why.}

* {Decision 1}
* {Decision 2}

## Precedent found

h3. In this repo

{Bulleted findings with file:line citations, or "None found -- this is new
ground for this codebase."}

h3. Other LMS applications

{Only include this sub-heading if the user approved a cross-repo scan in Phase 2.
Bulleted findings with repo name + file path citations, or "Checked {repos} --
no directly applicable precedent found." Omit the whole sub-heading if the user
declined the scan; the header-level "Cross-repo precedent checked" field already
records that.}

## Option 1: {Title} -- {Most Practical | Worth Considering | Speculative | Don't Do This}

h3. Summary

{One paragraph: what this option is, in plain terms.}

h3. Architecture

{Which layers this touches (Angular / Spring Boot service / repository / DB),
and how they fit together for this option. Name the pattern being followed
(e.g. "controller -> service -> repository, same as {{ExistingController}}").}

h3. Classes / Files

{Bulleted list. Name real existing files/classes this option reuses or modifies,
and the new ones it needs, each tagged with its layer.}

* {{ExistingService.java}} (modify -- service layer)
* {{NewFeatureRepository.java}} (new -- repository layer)

h3. Level of Effort

**Estimate:** {hours, divisible by 3, junior-developer basis}

{One or two sentences on what drives the estimate -- what's straightforward,
what's the risky or time-consuming part.}

h3. Jira Shape

{One of:
- "Fits in one ticket, no sub-tasks needed (estimate <= 6h)."
- "One parent issue with N sub-tasks (estimate > 6h; sub-tasks sized <= 6h each,
  listed below)."
- "Needs an epic with N parent issues (multiple vertical slices)."
Then list the sub-tasks or parent issues by one-line title if estimate > 6h,
enough detail that each line could seed a real Jira issue later.}

h3. Tradeoffs / Risks

{Bulleted. Honest costs of this option -- complexity, coupling, what it doesn't
solve, what could break, deviations from existing convention if any.}

## Option 2: {Title} -- {tag}

{Same sub-headings as Option 1.}

## Option N: Don't do this

{Only include when it's a genuine candidate, ranked wherever it actually belongs
(not always last). State plainly why: payoff doesn't justify effort, conflicts
with an existing ADR (name it), or the idea isn't practical given current
architecture. Skip the Classes/Files and Jira Shape sub-headings for this option
-- there's nothing to build.}

## Recommendation

{Prose. Which option the research points toward and why, in terms the senior
developer can act on immediately. If genuinely a toss-up between two options,
say so and name the deciding factor that would break the tie.}

## Open questions for the senior developer

{Bulleted list of anything that needs human judgment beyond what research or
grilling could resolve -- product priorities, risk appetite, timeline
pressure, cross-team dependencies.}

* {Question 1}
```

## Recommendation-strength tags

Use exactly one per option, matching `improve-codebase-architecture`'s badge
vocabulary so ranking language stays consistent across research documents:

- **Most Practical** -- the option research points toward; use once per document.
- **Worth Considering** -- viable, real tradeoffs against the top pick.
- **Speculative** -- possible but thin on precedent or unusually risky.
- **Don't Do This** -- effort/payoff or architectural fit fails; explain why.

## Formatting checklist before saving

- ASCII only -- no `->` written as an arrow glyph, no em dashes, no curly quotes.
- `*bold*` for UI elements, routes, endpoint paths.
- `{{monospace}}` for class names, file names, DB objects, identifiers.
- `h3.` for sub-headings inside a section body -- never Markdown `###` there.
- `{code:java}` / `{code:sql}` / etc. for any code snippet, always with a
  language tag, closing bare `{code}`.
- Every enumerable risk or sub-task gets its own bullet, not a compound bullet.
