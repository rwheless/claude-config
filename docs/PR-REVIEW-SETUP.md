# Setting up Claude for PR reviews

This walks a developer through using Claude Code to review a coworker's Bitbucket
pull request end-to-end: generating a handoff file from Bitbucket, then running the
`/review-pr` skill against it to produce paste-ready comments.

**Assumes:** Claude Code is already installed and configured, and the `review-pr`
and `code-review` skills are already added. (Install/setup is covered in a separate
document.)

The steps below exist because of specific permission and environment problems hit
while building this workflow — following them in order avoids re-discovering the
same issues.

---

## 1. Create a Bitbucket HTTP access token

The handoff script authenticates to Bitbucket as you, via a personal access token —
not your password.

1. In Bitbucket, click your avatar (top right) → **Manage account**.
2. In the left sidebar, click **HTTP access tokens**.
3. Click **Create token**.
4. Give it a name you'll recognize later, e.g. `claude-pr-review`.
5. Set the permission to the **lowest read-only level** that covers the projects
   you'll review (e.g. Project Read / Repository Read). The script this token is
   used for only ever issues GET requests — it never posts, resolves, or approves
   anything on Bitbucket — so it should never need write access.
6. Click **Create**, then **copy the token value immediately** — Bitbucket only
   shows it once.

## 2. Store the token where Claude Code will actually see it

This is the step that's easy to get wrong. Do **not** put this in `~/.profile` —
`.profile` is only read by *login* shells, and the shell Claude Code spawns for
tool calls is typically a non-login shell, so anything exported there will
silently never be visible to Claude.

Instead, add it to `~/.bashrc` (or your shell's non-login startup file, e.g.
`~/.zshrc` if you use zsh):

```bash
export BITBUCKET_PR_TOKEN=<paste-your-token-here>
```

Add this near your other `export` lines, not inside any block guarded by an
interactive-only check.

### Verify it before relying on it

Open a **new terminal window** (not a Claude session — a plain terminal) and run:

```bash
echo "${BITBUCKET_PR_TOKEN:-still unset}"
```

If it prints your token, the shell side is correct.

Then verify Claude Code itself sees it. **Start Claude Code from that terminal**
(type `claude`, don't launch it from a desktop icon/dock) and ask it to run:

```
echo "${BITBUCKET_PR_TOKEN:-still unset}"
```

If that also prints the token, you're set for every session started the same way.

> **Why "started from a terminal" matters:** launching Claude Code by typing the
> command in a terminal you opened means it inherits that terminal's environment,
> which already ran your shell's startup files. Launching it from a desktop
> icon/dock does not reliably go through those same startup files, so the token
> may not be visible there. Until you've verified otherwise, **prefer starting
> Claude Code sessions you'll use for PR review work from a terminal**, and run
> the handoff script (step 3) directly in a terminal yourself rather than asking
> Claude to run it, so you're never depending on how that particular session was
> launched.

## 3. Get the handoff script

The script lives in this repo at `scripts/bitbucket_pr_to_review`. Install it
somewhere on your `PATH` so it's runnable by name:

```bash
cp scripts/bitbucket_pr_to_review ~/.local/bin/bitbucket_pr_to_review
chmod +x ~/.local/bin/bitbucket_pr_to_review
```

(`~/.local/bin` is already on `PATH` if your `~/.bashrc` has
`export PATH="$HOME/.local/bin:$PATH"` — most setups here already do.)

This is a **plain copy, not a symlink** — if the script in this repo gets updated
later (bug fixes, new fields, etc.), your local copy won't automatically pick up
the change. See the "keeping your copy in sync" prompt at the bottom of this doc.

## 4. Generate the handoff file

`cd` into the repo whose PR you're reviewing, then run the script with the PR's
ID:

```bash
cd ~/GitRepos/<the-repo-you're-reviewing>
bitbucket_pr_to_review <PR-ID>
```

Project and repo are inferred from that repo's git origin remote. This writes
`pr-review-handoff-{PROJECT}-{REPO}-{ID}.md` in your current directory,
containing the PR's title, description, branches, and every open comment/task
thread (including replies nested under other comments).

**Example** — reviewing PR #14 in `duebotv3`:

```bash
cd ~/GitRepos/duebotv3
bitbucket_pr_to_review 14
```

This writes `pr-review-handoff-LDB-duebotv3-14.md` (`LDB` and `duebotv3` inferred
from `duebotv3`'s origin remote, `ssh://git@bitbucket.liberty.edu:7999/ldb/duebotv3.git`)
in the current directory, ready to hand to `/review-pr` in step 5.

If it exits with `BITBUCKET_PR_TOKEN is not set...`, go back to step 2 — the
token isn't reaching this shell.

## 5. Run the review

In Claude Code, invoke the skill against the handoff file, choosing a tier for
the comment voice:

```
/review-pr @pr-review-handoff-{PROJECT}-{REPO}-{ID}.md junior
```

or `mid` instead of `junior` for a terser, less explanatory comment style.

The skill will orient on the PR, review the diff against the target branch,
present findings for you to triage (confirm, downgrade, or add to), and — once
you explicitly confirm — write a `review-comments-{PROJECT}-{REPO}-{ID}.md` file
with paste-ready Bitbucket comments and tasks.

**Claude never posts anything to Bitbucket.** You copy the comments from that
file into Bitbucket yourself.

---

## Keeping your local script copy in sync

Because step 3 is a plain file copy, your `~/.local/bin/bitbucket_pr_to_review`
can silently drift out of date if this repo's copy gets fixed or improved later.
Periodically — or if a handoff file looks like it's missing comments you can see
live on the PR — ask Claude to check and resync it. Paste this prompt:

```
Compare my local ~/.local/bin/bitbucket_pr_to_review against
scripts/bitbucket_pr_to_review in the claude-config repo. If they differ, show
me the diff, explain what changed, and if I agree, copy the updated version
into ~/.local/bin and make sure it's still executable.
```

If you ever notice a handoff file is missing a comment you can see live on
Bitbucket, and it turns out to be a reply nested under another comment, that's a
known pattern: Bitbucket Server nests replies inside the parent comment's own
`comments` field rather than always listing them as independent activities. The
current script version already walks those nested replies — if yours doesn't,
the sync prompt above will catch it.
