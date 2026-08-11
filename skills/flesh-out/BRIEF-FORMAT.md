# IDEA BRIEF Format

The IDEA BRIEF is the single consolidated artifact a `flesh-out` session emits
when an idea ends in a **new repo**. It is one copyable chat-text block. Its
consumers are later Claude sessions: a CLI session that materializes the embedded
seed files into a fresh repo, or a planning session (e.g. `plan-with-me-personal`)
that ingests it as input context. It must therefore be fully self-describing —
no consumer is edited to expect it.

## Emission rules

- Emit as ONE block the user copies in a single gesture. Do not split across
  messages unless the platform truncates.
- Do not wrap the whole brief in a fenced code block — embedded files would need
  nested fences, which break. Use the `===== FILE =====` delimiters below instead.
- Every decision listed must have been explicitly resolved with the user. Anything
  else goes under Open questions.
- Research claims keep their links. A future session must be able to re-verify.

## Structure

```markdown
# IDEA BRIEF — {short idea name}

> You are receiving an IDEA BRIEF, the output of a flesh-out ideation session
> (format: https://raw.githubusercontent.com/rwheless/claude-config/main/skills/flesh-out/BRIEF-FORMAT.md).
> Decisions below are settled — do not re-litigate them; resolve Open questions
> first. To materialize: create each `===== FILE: path =====` section as a real
> file at that path in the new repo, verbatim.

**Date:** {YYYY-MM-DD}
**Status:** {aligned | emitted early — open questions remain}

## Goal

{The refined pitch in one paragraph: what is being built, for whom, and what
done looks like.}

## Out of scope

{Explicit exclusions agreed during the session.}

## Decisions

{One entry per resolved decision, in the order they were made.}

- **{Decision}** — {what was chosen and the one-line rationale}
  - Rejected: {alternative} — {why}

## Research findings

{Load-bearing facts discovered during the session, each with its link and a
note when version- or date-sensitive.}

- {Finding} ({link}, checked {YYYY-MM-DD})

## Risks accepted

{Risks surfaced during grilling that the user chose to accept, one line each.}

## Open questions

{Unresolved items, in dependency order. Empty only when Status is "aligned".
A resuming flesh-out session treats this as its working list.}

## Seed files

===== FILE: README.md =====
{Full file content, verbatim.}
===== END FILE =====

===== FILE: .claude/context/CONTEXT.md =====
{Glossary seed: the precise terms coined during the session, per the suite's
CONTEXT.md format — terms, definitions, relationships only.}
===== END FILE =====
```

## Section rules

- **Seed files** always includes a README.md draft. Include a
  `.claude/context/CONTEXT.md` glossary seed when the session coined terms worth
  keeping precise. Add other seeds (e.g. a docker-compose sketch, a schema
  sketch) only when the session actually resolved their content — never pad with
  boilerplate the user didn't discuss.
- File contents between the delimiters are verbatim — no commentary inside.
- Omit empty sections entirely rather than leaving placeholders, except Open
  questions, which is always present (write "None" when aligned).
