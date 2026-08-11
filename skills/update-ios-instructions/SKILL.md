---
name: update-ios-instructions
description: Regenerate ios-instructions.md at the root of rwheless/claude-config by scanning the skills directory via the GitHub MCP and rebuilding the full raw-URL listing. Call this as the final step whenever skill files are created or modified in the claude-config repo.
---

# update-ios-instructions

Regenerate `ios-instructions.md` at the root of `rwheless/claude-config` by scanning every skill in the `skills/` directory and rebuilding the raw GitHub URL listing. This file is included verbatim in the Claude iOS system prompt so the app can fetch and follow skills without direct GitHub repository access.

## When to run

Call this skill as the **final step** whenever you:

- Create a new skill directory or any file inside `skills/`
- Modify an existing skill file
- Delete a skill or a skill file

Do not wait for the user to ask — regenerate and commit as part of completing the skill-writing work.

## Process

1. **List the skills directory** via the GitHub MCP:
   `GET /repos/rwheless/claude-config/contents/skills`

2. **For each skill directory** (sorted alphabetically by skill name):
   - List its contents via MCP.
   - Collect all `.md` files. Order: `SKILL.md` first, remaining files alphabetically.
   - Build the raw URL for each file:
     `https://raw.githubusercontent.com/rwheless/claude-config/main/skills/<skill-name>/<filename>`

3. **Build the full file content** using this exact format:

```
## My Skills (from github.com/rwheless/claude-config)

When a task matches one of these skills, fetch and follow its instructions.
Base URL: https://raw.githubusercontent.com/rwheless/claude-config/main/skills

### Available skills:

**<skill-name>**
- <raw-url-to-SKILL.md>
[- <raw-url-to-other-file.md> ...]

**<next-skill-name>**
...

When using a skill, fetch the SKILL.md first. For skills with additional files, fetch the supporting files as needed during the task.
```

4. **Push the file** via the `push_files` GitHub MCP tool:
   - owner: `rwheless`
   - repo: `claude-config`
   - branch: `main`
   - path: `ios-instructions.md`
   - commit message: `chore: regenerate ios-instructions.md`

No PR needed — commit directly to `main`.
