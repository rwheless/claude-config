# COMPONENT BRIEF Format

The COMPONENT BRIEF is what build-style-guide's Phase 8 derives from an
approved style-guide page: the inventory of components to build, written to
`.claude/style-guide/COMPONENT-BRIEF.md` in the client worktree and committed
with the branch. Its consumer is a planning session (`plan-with-me` for work,
`plan-with-me-personal` for personal) that grills over it and creates the
component-library conversion issues.

## Rules

- **Inventory only.** No issue slicing, no estimates, no AFK/HITL labels —
  planning owns those, per its own format skills. Duplicating that judgment
  here would drift when jira-formats/github-formats change.
- Only elements that exist on the approved page get entries. Checklist
  elements deferred at intake are listed under Out of scope.
- Every existing-usage claim carries a file:line citation.
- CDK recommendations state pros/cons in plain language; they are
  recommendations, not decisions.

## Structure

```markdown
# COMPONENT BRIEF — {app name}

> You are receiving a COMPONENT BRIEF, the output of a build-style-guide
> session (format: https://raw.githubusercontent.com/rwheless/claude-config/main/skills/build-style-guide/BRIEF-FORMAT.md).
> The style guide it derives from is approved — the look is settled; do not
> re-litigate it. This is planning input: slice, estimate, and label the
> conversion work per your planning flow's own rules.

**Date:** {YYYY-MM-DD}
**Derived from:** branch `style-guide` at commit {short-hash}
**Seed source:** {claude.ai/design export | reference app | current app styles}
**Styled framework to retire:** {none | Angular Material x.y | PrimeNG | …}

## Sequencing

{Ordered build sequence with the dependency rationale — e.g. field wrapper
before the controls that compose into it; overlay host before dialog,
popover, and bottom sheet.}

## Components

### {ComponentName} (`{bem-block}`)

- **Partial:** {src/styles/components/_x.scss}
- **Style-guide section:** {#anchor on the page}
- **API sketch:** {inputs/outputs/content projection, a few lines}
- **Behavior needs:** {none | CDK recommendation with pros/cons}
- **Existing usages to convert:** {file:line citations, or "new — no usages"}
- **Framework migration:** {what Material/PrimeNG/Bootstrap usage it replaces,
  with citations, or "n/a"}
- **A11y criteria:** {keyboard operation, ARIA roles/states, focus management}
- **Style-guide swap:** replace this component's raw-markup section with the
  real component when it lands (part of the same issue's acceptance).

## Framework retirement

{Present only when a styled framework is being replaced: the imports/modules/
theme files that can be deleted once the listed components land, and any usage
that maps to no planned component and needs a decision.}

## Out of scope

{Checklist elements deferred at intake; app-screen migration beyond the cited
usages; anything the user excluded during the session.}
```

## Section rules

- Omit empty sections except Out of scope (write "None" if empty).
- The style-guide swap line is on every component — the page becomes the
  living component showcase as conversions land.
