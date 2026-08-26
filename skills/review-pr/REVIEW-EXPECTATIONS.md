# Review Expectations

> Distilled from the internal "LMS Team PR Expectations" wiki doc (last refreshed
> 2026-07-02). When the user provides an updated copy, propose a diff to this file.

Only what a reviewer needs at review time. Author-side process (Teams chat,
merging checklist, approval mechanics) is intentionally omitted.

## Comment ethos

- Focus on the **code, never the developer**. Be kind. Leave your ego at the door.
- Comments must be **objective, specific, and outcome-focused**.
- **Back every suggestion with facts** — not personal preference.
- Use comments for small or self-explanatory feedback. For larger or more
  complex feedback, **communicate directly with the author** rather than through
  comments (this is the "Discuss-directly" disposition).

## Review Focus lenses

- **Complexity** — Can you understand the code? Will a different developer (or the
  author, later) still follow it? Flag "workaround" or "temporary" code that needs
  a permanent fix.
- **Consistency** — Does it follow working, established patterns already in the
  project? Is there an opportunity to reuse existing code or libraries?
- **Conventions** — Are industry-wide and library/framework/language conventions
  followed?
- **Documentation** — Missing documentation is a blocker for approval. Code changes
  that require doc updates but don't include them are **Blocking**.
- **Error handling** — Is error handling consistent across the codebase? Are edge
  cases handled? If an error-handling gap exists but is out of scope, is it at least
  referenced by a TODO?
- **Naming** — The team prefers **longer, more specific names** that are easier to
  understand. A naming concern should be raised as a comment that starts a team
  discussion about how to rename.
- **Scalability** — Does it refactor/rewrite without much impact? Can it handle
  future scenarios that are slightly different?
- **Security** — Does the PR follow the security standards? See `SECURITY-CHECKLIST.md`.
- **Tests** — **Missing tests are a blocker.** Tests must be meaningful and
  purposeful. Rare exceptions where no test is relevant: client styling or
  configuration.
  - A test that exists is not the same as a test that discriminates. If two or
    more asserted values in the same test could be swapped, sourced from the
    wrong field, or are coincidentally equal, and the test would still pass,
    treat it as unproven coverage. Check that fixture values are unique across
    whatever the assertions are meant to tell apart.
  - When a test's own setup (a wrapping `@Transactional`, a mock, an open
    session) differs from how the code actually runs in production, check
    whether that difference would hide the exact regression the test is
    supposed to catch.

## Blocking issues

Issues that impact functionality, security, or maintainability and must be
addressed by the author before approval:

- **Core functionality** — Code doesn't meet the intended functionality per the PR
  description or acceptance criteria.
- **Security issues** — Introduces vulnerabilities that expose personal/sensitive
  data or compromise the system.
- **Code standards violations** — Deviates majorly from established team, project,
  or style conventions.
- **Code smells** — Poor structure, readability, or antipatterns.
- **Regression** — Unintended side effects, or breaks existing functionality.
- **Performance issues** — Negatively impacts performance or resource usage (e.g.
  network requests, database calls, large in-memory lists, loops).
- **Failing tests** — Changes cause existing tests to fail. (As a reviewer, do not
  run suites — CI owns this — but call out changes that clearly break tests.)
- **Missing tests** — A change with testable behavior and no test.
- **Documentation** — Changes that require doc updates but haven't been made.

For each blocking finding, leave a comment tagged as blocking **with an
explanation of why** it is blocking, so it can be discussed.

## Non-blocking issues

Issues that do not impact functionality, security, or maintainability, and can be
handled by a separate PR, minor update, or as constructive feedback:

- **Style preferences** — Style differing from your own; minor spacing/indentation.
- **Documentation nitpicks** — Minor inconsistencies, typos, non-impacting issues.
- **Missing but optional features** — Core functionality is met but an optional
  feature mentioned in passing is absent.
- **Minor refactor opportunities** — Improvements that don't impact functionality
  and can be a separate PR.
- **Unrelated improvements** — Suggestions unrelated to this PR, for a separate PR.

## PR hygiene a reviewer can verify

- **Title** — contains the Jira issue number and a summary.
- **Description** — states what changed, why, and how to test.

A hygiene miss becomes a PR-level note; it is not a blocker on its own.
