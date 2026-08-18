---
name: jira-formats
description: The team's Jira issue templates and conventions — user story format, technical story format, bug format, improvement format, sub-task format, acceptance criteria style, and formatting rules. Read this whenever writing or revising Jira issue documentation, before drafting any story, sub-task, or acceptance criteria. Used by plan-to-jira; also consult it during grilling when proposing issue breakdowns so proposals match the team's conventions.
---

# jira-formats

Templates and conventions derived from the team's real issues. Match them exactly — the goal is that pasted issues are indistinguishable from ones the lead wrote by hand.

## Issue types in use

- **User Story** — user-observable capability, testable end to end. Title: `As a(n) {role}, I can {capability}`.
- **Technical Story** — backend/infrastructure work with no direct UI surface. Title: imperative description of the work.
- **Bug** — corrects unexpected behavior. Use when grilling a ticket or report about something not working as intended that requires code changes to fix. Title: imperative description of what is being fixed.
- **Improvement** — non-behavioral enhancement to the codebase or user experience. Use for stylistic changes, small refactors, and other changes that do not affect existing behavior or features. Title: imperative description of what is being improved.
- **Sub-task** — created under a story only when the story's estimate exceeds 6 hours. Sub-tasks exist to break that work into pieces that can each be completed in 6 hours or less.

## Worktree directory

Resolved per `WORKTREE-LOCATION.md` (in `~/.claude/skills/`) — a sibling of the
repo, `../<repo>-worktrees/{JIRA-SLUG}/`, by default; inside the bare repo
directory itself, `<bare-root>/{JIRA-SLUG}/`, when the repo is a
bare-repo-and-worktrees setup. Branch name is always the exact Jira slug. Used
by `pickup-issue` and `update-docs`.

## Formatting conventions (all issue types)

- **No Unicode special characters.** Use only ASCII in issue content. Never use arrows (→, ←, ↑, ↓), em dashes (—), or smart/curly quotes — Jira renders these as `?`. Replace arrows with plain words (`to`, `from`, `then`, `via`); replace em dashes with ` - ` (space-hyphen-space).
- All description field content **MUST** use valid Jira Wiki Markup syntax — not Markdown.
- `*text*` (single asterisks) for bold: UI elements, menu items, routes, endpoint paths.
- `{{text}}` (double curly braces) for monospace/inline code: code identifiers, DB objects, naming conventions.
- A `{{...}}` span can silently render as "Unknown macro: {content}" instead of monospace text if an installed Jira/Confluence app registers a macro whose shortname matches the span's content with non-alphanumeric characters stripped (observed with `{{loadMore}}` - and confirmed the match survives added punctuation: `{{(loadMore)}}` still triggered "Unknown macro: {(loadMore)}", since `loadMoreError()` and `loadMoreError` did NOT trigger it, only content that strips down to exactly the registered shortname). Adding symbols inside the braces does NOT fix this. If a `{{word}}` span renders broken, drop the `{{}}` formatting entirely and write the identifier as plain text instead - do not try to disambiguate by adding punctuation inside the braces.
- Naming conventions may use `{placeholder}` syntax inside `{{}}` to mark variable parts (e.g., `{{findBy{PropertyName}()}}`), but avoid this when the placeholder contains `.` (Spring property paths) or any other `{...}` that Jira would parse as a macro — it will break the span.
- `-text-` (hyphens) for strikethrough: descoped requirements — keep visible rather than deleting.
- Cross-reference other issues by key when known, otherwise by planning-file slug.
- Code blocks: opening tag on its own line, code content indented for the language, closing `{code}` on its own line. Always use a language specifier — never bare `{code}` as an opening tag. Supported types: `java`, `sql`, `yaml`, `javascript`, `js`, `json`, `xml`, `bash`, `sh`, `groovy`, `html`, `css`, `python`. Closing tags are always bare `{code}` regardless of the opening language.
- Section sub-headings within the description body use Jira heading syntax: `h3. Other Helpful Information`, `h3. Testing`, `h3. Technical Acceptance Criteria`, `h3. Steps to Reproduce`, etc. — never Markdown `###`.
- **Never embed `{...}` inside any inline formatting span.** Jira parses `{word}` as a macro call regardless of surrounding characters — `*GET /api/blueprints/{id}*` breaks because `{id}` triggers macro parsing mid-bold-span, and `{{/route/{id}}}` breaks for the same reason inside a monospace span. For REST path parameters and Angular route segments, always use colon-style (`:id`) instead of curly-brace style within `*...*` or `{{...}}`: write `*GET /api/blueprints/:id*` and `{{/blueprints/:id}}`. When the curly-brace form is required (e.g., Spring property placeholders like `{property.name}`), move the whole expression into a `{code}` block. Asterisks and underscores inside `{{}}` generally render safely (Jira only triggers bold/italic on whitespace-bounded `*text*` and `_text_`), but when in doubt, prefer a `{code}` block.

## Acceptance criteria rules

The `## Acceptance Criteria` section is written for the team's tester (QAE). Their workflow is to deploy a version of the application after a parent issue is completed, stage any scenarios by modifying data in relevant database tables, and then manually test for correct behavior. That means verifying UI functionality (buttons, navigation, display), verifying database state (correct creates, updates, or deletes), or verifying that an endpoint returns the expected response given certain inputs.

- A bulleted list under an `## Acceptance Criteria` heading.
- Every bullet starts with **"Should"** and describes one independently testable behavior.
- Phrase from the observer's perspective: what the user sees (user stories) or what the system does (technical/bug/improvement stories).
- Cover the failure path, not just the happy path.
- Each enumerable variant gets its own bullet rather than one compound bullet.
- ACs define done for the whole story; sub-tasks do not get their own AC sections.
- Bullets must use **plain text only** — no Jira Wiki Markup (`{{}}`, `*bold*`, `{code}`). Each bullet is pasted as the title of a separate Jira "Acceptance Criteria" issue, and issue titles do not render markup.

## Original Estimate rules

- Expressed in hours. Must be divisible by 3 (e.g. 3, 6, 9, 12, 15...).
- Represents the hours a junior developer would be expected to spend completing the work.
- For issues with sub-tasks, the parent estimate equals the sum of all sub-task estimates.

## User Story template

```markdown
# As a(n) {role}, I can {capability}

**Issue Type:** User Story

**Original Estimate:** {hours, divisible by 3}

## Description

{Prose, ordered by how the user encounters the feature: entry point and navigation,
then main view and behavior, then interactions (filtering, paging, links), then error states.}

{Back end contract paragraph: the endpoint(s) this story needs, in prose — method
and path in bold, query parameters in bold, page sizes, filter semantics.}

{Dependency paragraph, if any: name the blocking issue, state what it will deliver,
and prescribe the interim approach for THIS issue (mock data, stub), including a
sample SQL query or fixture to generate realistic data.}

h3. Other Helpful Information

{TypeScript interface(s) for the front end model.}

{Sample JSON payload with realistic values.}

h3. Testing

{Optional. Include only when there is non-obvious setup or context someone needs
before they can execute the acceptance criteria — e.g., which DB records to seed,
which account or role to use, which feature flag to toggle. Omit if the AC steps
are self-explanatory.}

h3. Technical Acceptance Criteria

{Bulleted list of developer-verifiable completion criteria — things the tester
cannot confirm manually. Examples: specific classes or methods exist with correct
contracts, data transformations produce the right output, edge cases are handled
in code, DB migration runs cleanly, integration points are wired correctly, a
scheduled job logs the expected outcome, a service returns the right value for a
known input. Must use valid Jira Wiki Markup syntax.}

## Acceptance Criteria

* Should ...
* Should ...

## Sub-tasks

{Only when the parent estimate exceeds 6 hours — see sizing rules below.}

## Attachments

{Optional. Relative links to Mermaid diagrams etc., when present.}
```

## Technical Story template

```markdown
# {Imperative description of the work}

**Issue Type:** Technical Story

**Original Estimate:** {hours, divisible by 3}

## Description

{Prose: the trigger/schedule, the data sources (tables/views in monospace), the
external APIs and how they're queried (naming conventions as bulleted list of
{{placeholder}} patterns when there are several), filtering/exclusion rules, and
persistence behavior (insert vs update semantics).}

h3. Other Helpful Information

{Reference any existing code the developer should read before implementing —
name the file, class, or method and note what the new implementation
preserves, simplifies, or replaces. Then include pseudocode in a {code:java}
(or appropriate language) block showing the skeleton of the new
implementation: the key method calls, branching logic, and persistence
pattern. Junior developers should be able to orient and start coding from
this section alone.}

h3. Testing

{Optional. Include only when there is non-obvious setup or context someone needs
before they can execute the acceptance criteria — e.g., which DB records to seed,
which account or role to use, which feature flag to toggle. Omit if the AC steps
are self-explanatory.}

h3. Technical Acceptance Criteria

{Bulleted list of developer-verifiable completion criteria — things the tester
cannot confirm manually. Examples: specific classes or methods exist with correct
contracts, data transformations produce the right output, edge cases are handled
in code, DB migration runs cleanly, integration points are wired correctly, a
scheduled job logs the expected outcome, a service returns the right value for a
known input. Must use valid Jira Wiki Markup syntax.}

## Acceptance Criteria

* Should ...
* Should ...

## Sub-tasks

{Only when the parent estimate exceeds 6 hours.}

## Attachments

{Optional. Sequence diagram link for multi-system flows — scheduled jobs calling
external APIs warrant one.}
```

## Bug template

```markdown
# {Imperative description of what is being fixed}

**Issue Type:** Bug

**Original Estimate:** {hours, divisible by 3}

## Description

{Prose context: what component or feature is affected and under what conditions
the bug occurs.}

h3. Steps to Reproduce

{Numbered steps a developer or QAE can follow to observe the bug.}

h3. Expected Behavior

{What should happen.}

h3. Actual Behavior

{What currently happens instead.}

h3. Other Helpful Information

{Optional: relevant stack traces, screenshots, log output, or code references.}

h3. Testing

{Optional. Include only when there is non-obvious setup or context someone needs
before they can execute the acceptance criteria — e.g., which DB records to seed,
which account or role to use, which feature flag to toggle. Omit if the AC steps
are self-explanatory.}

h3. Technical Acceptance Criteria

{Bulleted list of developer-verifiable completion criteria — things the tester
cannot confirm manually. Examples: specific classes or methods exist with correct
contracts, data transformations produce the right output, edge cases are handled
in code, DB migration runs cleanly, integration points are wired correctly, a
scheduled job logs the expected outcome, a service returns the right value for a
known input. Must use valid Jira Wiki Markup syntax.}

## Acceptance Criteria

* Should ...
* Should ...

## Sub-tasks

{Only when the parent estimate exceeds 6 hours.}

## Attachments

{Optional.}
```

## Improvement template

```markdown
# {Imperative description of what is being improved}

**Issue Type:** Improvement

**Original Estimate:** {hours, divisible by 3}

## Description

{Prose: what is being improved, why it is being improved, and what the desired
end state looks like. Reference existing code being changed where relevant.}

h3. Other Helpful Information

{Optional: relevant code snippets, before/after examples, or style references.}

h3. Testing

{Optional. Include only when there is non-obvious setup or context someone needs
before they can execute the acceptance criteria — e.g., which DB records to seed,
which account or role to use, which feature flag to toggle. Omit if the AC steps
are self-explanatory.}

h3. Technical Acceptance Criteria

{Bulleted list of developer-verifiable completion criteria — things the tester
cannot confirm manually. Examples: specific classes or methods exist with correct
contracts, data transformations produce the right output, edge cases are handled
in code, DB migration runs cleanly, integration points are wired correctly, a
scheduled job logs the expected outcome, a service returns the right value for a
known input. Must use valid Jira Wiki Markup syntax.}

## Acceptance Criteria

* Should ...
* Should ...

## Sub-tasks

{Only when the parent estimate exceeds 6 hours.}

## Attachments

{Optional.}
```

## Sub-task format

Inside the parent issue file, under `## Sub-tasks`:

```markdown
### {PENDING-N}. {One-sentence summary of the sub-task}

**Original Estimate:** {hours, divisible by 3}

#### Description

{Prose: what this sub-task delivers, which layer(s) it touches, what it must NOT
include (left for a later sub-task), and what it can rely on from earlier
sub-tasks. Name concrete files/classes/tables where known. Must use valid Jira
Wiki Markup syntax.}
```

Sub-task `PENDING-N` numbers continue the same sequence used for parent issues in the session — do not restart at 1. On slug fill-in, replace `PENDING-N` in the header with the real Jira key; keep the period and summary unchanged. Result: `### LDB-1332. {summary}`.

## Sub-task sizing rules

- A parent issue estimated at 6 hours or less does not need sub-tasks.
- Split a parent issue into sub-tasks only when its estimate exceeds 6 hours.
- Each sub-task must be sized to 6 hours or less (and must still be divisible by 3 per the estimate rules).
- Parent issues must represent a complete vertical slice — they should cover all relevant layers (DB, service, controller, Angular) for a self-contained, testable capability. Do not split work horizontally across parent issues by layer; that creates integration gaps and makes behavior hard to verify until all pieces land.
- The layer-by-layer ordering (DB first, then service, then controller, then Angular) applies to sub-tasks within a vertical-slice parent issue — it is the implementation sequence, not the issue decomposition strategy.
- Order sub-tasks so each is testable or demonstrable when merged (mock or stub the not-yet-built layers above).

## Tier calibration

- **Junior**: prescribe more — name files, classes, endpoints, and patterns to copy; include sample payloads and queries; spell out edge cases in the description.
- **Mid-level**: state intent, contracts, and constraints; leave implementation approach open; flag the risky parts rather than prescribing solutions.
- Never write the tier into the issue text itself — tier suggestions live only in the planning session's `OVERVIEW.md` Issues list.
