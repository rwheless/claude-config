# Skill roadmap — deferred work

Seed notes for future skill-building sessions. Each entry gets its own
grill-with-docs session. Written 2026-07-02 during the Sonnet-readiness audit.

## Seed entries

- **build-style-guide: Figma as a seed source.** Work clients receive designs
  in Figma; extend build-style-guide's seed intake (currently claude.ai/design
  export / reference app / current app) to extract tokens and element styling
  from Figma. Deferred 2026-07-04 during the build-style-guide grilling
  session.

- **Native blocked-by for dependencies.** The DEPENDS-ON LINE text convention is
  the sole dependency signal because the GitHub MCP exposes no issue-dependency
  tools (verified 2026-07-05: only sub-issues, labels, fields). When the MCP gains
  native blocked-by support: update plan-to-github to set the relationship
  alongside the text line, and sweep-issues-personal to read native first with
  text fallback. Deferred 2026-07-05 during the sweep-issues-personal grilling
  session.

- **update-docs: module README template — optional Key Data Structures and Configuration Reference sections.** Surfaced 2026-07-06 during an update-docs session on duebotv3 (migrator-service). The template ends at Running tests; for Spring Boot services with complex domain objects, two extra sections proved genuinely useful: Key Data Structures (field-by-field tables for core entities/DTOs) and Configuration Reference (one-row-per-property table for all `application.*` properties). Both were kept in the rewritten doc. Open questions for grilling: optional sections vs a separate "data-heavy" template variant; whether the duplication with Javadoc/OpenAPI is load-bearing (junior-friendly prose) or noise; drift signals (class-name grep for data structures, `application*.yml` changes for config reference).

- **wrap-up: PLANNING-PERSONAL.md format.** `plan-with-me-personal` sessions
  (GitHub-only, no `jira-planning/` archive) have no defined wrap-up handoff
  shape; `wrap-up` currently only knows the Jira-planning-folder format. Needs a
  `PLANNING-PERSONAL.md` counterpart covering: original request, references
  (repo files + relevant GitHub issue numbers), what's aligned vs. not-yet-aligned
  from grilling, and a resume point — without assuming a local planning archive.
  Surfaced 2026-07-10 during an ad hoc `plan-with-me-personal` wrap-up on
  astrowatch (cleanup batch), which had to invent this shape from scratch.

- **Fork Anthropic's `/code-review` plugin into a claude-config-owned skill.**
  `/code-review` (`~/.claude/plugins/marketplaces/claude-plugins-official`) is
  not ours to edit - it's unversioned locally and gets silently overwritten on
  marketplace sync. Its multi-agent finder+verifier methodology is genuinely
  solid (caught a real NPE on LDB-1333), but it has no concept of the
  reachability check `vet-review-findings` now bolts on afterward as a
  separate pass. A full fork would replicate the 8-angle parallel-agent
  methodology directly in claude-config and build the reachability check into
  the finder agents themselves, so unreachable-guard findings never get
  reported as "patch this" in the first place - no separate pass required,
  and no dependency on a plugin we can't maintain. Deferred 2026-09-11 during
  the vet-review-findings build (LDB-1333 "slop" retro); the user explicitly
  wants both the lightweight gate (done) and this fuller fork (this entry).

