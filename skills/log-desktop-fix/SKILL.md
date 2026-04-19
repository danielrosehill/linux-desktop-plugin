---
name: log-desktop-fix
description: Log a desktop fix, configuration change, or system administration action to a structured, dated log directory. Use after completing a significant system change — installed a package, adjusted KDE/GNOME settings, fixed a hardware issue, removed a broken KWin script, etc. Captures problem / root cause / fix in a consistent format, with optional git commit+push if the log directory is a git repo.
---

# log-desktop-fix

Write a short, structured log entry for a desktop administration action the user just completed. The goal: future sessions (and the user) can look back and see what was changed, why, and how to reproduce or reverse it.

## When to trigger

- Immediately after finishing a non-trivial system change (config edit, package install/remove, hardware fix, window-manager tweak, audio/display/network adjustment).
- When the user says "log this", "save this", "write this up", "document this fix", or similar.
- Proactively after you've walked the user through a fix — ask once if they'd like it logged, then invoke.

Do **not** invoke for trivial, read-only operations (listing files, checking status, running a benchmark that made no changes).

## Where logs are written

Resolve the log directory in this order:

1. `$DESKTOP_OPS_LOG_DIR` environment variable, if set.
2. A plugin setting `log_dir` if present in `.claude/settings.json` under this plugin's namespace.
3. `~/desktop-fixes/` (created if missing).

Inside that root, create a dated subfolder `YYYY-MM-DD/` and write one markdown file per fix, named `<short-kebab-slug>.md`. Multiple fixes on the same day share the dated folder.

Use today's real date via `date +%Y-%m-%d` — do not guess.

## File format

```markdown
# <Short title> — <one-line summary of what changed>

**Date:** YYYY-MM-DD
**System:** <distro + version, DE if relevant>   <!-- from /etc/os-release + $XDG_CURRENT_DESKTOP -->

## Problem

<What the user observed. 1–3 sentences. Concrete symptoms, not speculation.>

## Root cause

<What was actually wrong. If uncertain, say so explicitly — "suspected X, confirmed by Y".>

## Fix

<The commands or settings changes that resolved it. Use fenced code blocks for anything reproducible.>

## Notes

<Optional. Reversal steps, lingering concerns, follow-ups, references.>
```

Keep the whole entry under ~60 lines. This is a log, not a manual — link out rather than restate.

## Git handling

After writing the file:

1. Check if the log directory is inside a git working tree (`git -C <dir> rev-parse --is-inside-work-tree`).
2. If yes: `git add` the new file, commit with message `Log: <short title>`, and `git push` if an upstream is configured.
3. If no remote / no upstream: commit locally and mention that push was skipped.
4. If not a git repo at all: just write the file and tell the user where it landed.

Never `git init` a new repo without asking — the user may have intentional reasons for the directory to be untracked.

## OS detection

Pull the "System" field from:

- `/etc/os-release` → `PRETTY_NAME` (distro + version)
- `$XDG_CURRENT_DESKTOP` (DE, if set and relevant to the fix)
- `uname -r` (kernel, only if the fix is kernel/driver-related)

Skip fields that aren't relevant to the specific fix — a font install doesn't need a kernel version.

## What NOT to log

- Secrets, API keys, tokens, passwords — even in passing references. If the fix involved rotating a credential, describe the action generically ("rotated the API key for service X") without the value.
- Personal file paths outside `$HOME` structure that would leak org/machine identity if the user's log repo is public.
- Speculation. If you don't know the root cause, say "root cause not confirmed" — don't invent one.

## Example invocation

User: "save an update in the repo for logging desktop updates"

Skill writes `$DESKTOP_OPS_LOG_DIR/2026-04-19/kwin-tiling-scripts-removed.md` with the problem/cause/fix structure, commits as `Log: KWin tiling scripts removed (dragging fix)`, pushes, and reports the path.
