---
name: claude-md-bulk-to-toon
description: Bulk-convert every CLAUDE.md (and related .claude/ context files) on the system into TOON. Walks the user-level config plus all repos under the user's repo roots, with an aggregate token-cost report across the entire Claude Code surface. Heavy operation — confirms scope before running.
---

# Bulk CLAUDE.md → TOON

System-wide conversion pass. Use to answer questions like *"how many tokens is my entire Claude Code context surface, and how much could TOON save?"*

## Confirm scope before running

This walks a lot of the filesystem. Before starting, list the discovery roots and ask the user to confirm or trim:

- `~/.claude/CLAUDE.md` and `~/.claude/context/`, `~/.claude/memory/` — user-level.
- All git repos under the user's repo roots. Defaults: `~/repos/`, `~/projects/`, `~/code/`, `~/work/`. Ask the user if they keep repos elsewhere.
- Skip dirs by default: `node_modules`, `.venv`, `venv`, `dist`, `build`, `.git`, `target`, `.next`, anything in `.gitignore`.
- Skip per-plugin-cache CLAUDE.md under `~/.claude/plugins/` (those are upstream, not the user's).

## Discovery

```bash
# User scope
find ~/.claude -maxdepth 3 -type f \( -name 'CLAUDE.md' -o -path '*/context/*.md' -o -path '*/memory/*.md' \)

# Repos
fd -HI -t f 'CLAUDE\.md$' ~/repos ~/projects ~/code ~/work 2>/dev/null \
  | grep -v '/node_modules/\|/\.venv/\|/venv/\|/dist/\|/build/\|/\.next/\|/target/\|/\.git/\|/\.claude/plugins/'
```

(Use `fd` if installed; otherwise `find`. Don't use `find /` — scoped roots only.)

Present the discovered file list to the user before processing. Let them remove paths.

## Workflow

For each file, delegate to the same parse → JSON tree → TOON encode → round-trip → token-delta pipeline used by `claude-md-user-to-toon` and `claude-md-repo-to-toon`. Don't duplicate that logic — call the shared helpers.

Run the conversion sequentially or with limited concurrency (≤4); markdown parsing is cheap but disk I/O on a large filesystem walk can stall.

## Manifest and outputs

Single batch manifest:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/state/bulk-claude-md-<timestamp>.json
```

Each row: source path, scope (`user` / `repo:<name>`), original tokens, TOON tokens, saving, status (`done` / `failed` / `skipped-no-savings`), output path.

Outputs go under:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/data/bulk-claude-md/<timestamp>/
├── user/                    # mirrors ~/.claude
├── repos/<repo-name>/...    # mirrors each repo's CLAUDE.md tree
└── report.md                # human-readable summary
```

## Final report (`report.md`)

- Total files scanned, converted, failed, skipped.
- **Aggregate token cost** of the user's full Claude Code context surface, original vs TOON.
- **Top 10 fattest CLAUDE.md files** — lets the user see where their context budget actually goes.
- Per-scope breakdown (user vs each repo).
- Manifest path for raw data.

## What NOT to do

- Don't overwrite any source `.md` file. Outputs are a parallel artefact tree.
- Don't `git add` or `git commit` anything in any of the visited repos.
- Don't follow `@` imports across repo boundaries — each repo's imports stay scoped to that repo, and `~/.claude/` is its own scope.
- Don't auto-run on a schedule. This is an ad-hoc analysis tool, not a routine.
