# claude-config

Public version-controlled Claude Code configuration: custom skills and shell scripts.

## What's here

- **`skills/`** — Custom Claude Code skills (slash commands). Each subdirectory is a skill with a `SKILL.md` defining its behavior.
- **`scripts/`** — Shell utilities for Claude Code. Currently includes `statusline-command.sh`, a status line renderer for the Claude Code terminal UI.

## Setup

Clone and symlink into `~/.claude/`:

```bash
git clone git@github.com:rwheless/claude-config.git <your-path>
cd <your-path>

# Replace the local directories with symlinks
rm -rf ~/.claude/skills ~/.claude/scripts
ln -s "$(pwd)/skills" ~/.claude/skills
ln -s "$(pwd)/scripts" ~/.claude/scripts
```

Then update your `settings.json` to reference the statusline script at its new path:
```
~/.claude/scripts/statusline-command.sh
```

## What's not here

- `~/.claude/CLAUDE.md` — personal global instructions (private)
- `~/.claude/settings.json` / `hooks/` — managed in a separate private dotfiles repo
