---
name: claude-md-repo-to-toon
description: Convert a repo's CLAUDE.md (and any nested CLAUDE.md / .claude/ context files inside that repo) into TOON. Use when measuring how much prompt budget a repo's Claude Code context costs, or experimenting with TOON-encoded context injection. Output goes alongside the originals — does NOT replace them.
---

# Repo CLAUDE.md → TOON

Convert all Claude Code context files inside a single repository to TOON.

## Inputs

1. **Repo path** — default: current working directory if it's a git repo; otherwise ask.
2. **Output mode** — `sibling` (default — writes `.toon` next to each `.md` inside the repo, gitignored) or `external` (writes to plugin data dir).

## What gets walked

- `<repo>/CLAUDE.md` (root).
- `<repo>/.claude/CLAUDE.md` (project-scope override, if present).
- Any `CLAUDE.md` in subdirectories — Claude Code supports per-folder context and these compose at runtime.
- Files referenced via `@<path>` imports from any of the above, **scoped to the repo**. Do not follow imports that resolve outside the repo (e.g. into `~/.claude/`).
- `<repo>/.claude/skills/*/SKILL.md` — these are markdown with frontmatter. Convert them too if the user passes `--include-skills` (default: skip; SKILL.md frontmatter has its own runtime semantics).

## Workflow

Same shape as `claude-md-user-to-toon`:

1. Walk the repo, assemble `{path: raw_markdown}`.
2. Parse each file's markdown to a structured JSON tree (headings, lists, tables, code blocks, prose).
3. Encode to TOON via the configured backend.
4. Round-trip check per file. Stop on any losslessness failure and surface the diff.
5. Per-file and aggregate token-delta report.

## Output

### `sibling` mode

Write `<file>.md.toon` alongside each `<file>.md`. Add a single line to the repo's `.gitignore` if not already present:

```
*.md.toon
```

Don't commit. Don't stage. Just write the files and tell the user where they are.

### `external` mode

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/data/repo-claude-md/<repo-name>/<timestamp>/
```

Mirrors the repo's directory layout under that root. Useful for repos the user doesn't want to clutter.

## Caveats to surface

- Prose-heavy CLAUDE.md → modest token savings. List/table-heavy → bigger.
- `@` import resolution at runtime is Claude Code's job; the TOON output is a flat snapshot, not an executable substitute. If the user wants to actually swap context delivery format, that's a separate (much larger) project.
- Code blocks inside CLAUDE.md should be preserved verbatim as string values in the JSON tree — don't re-parse them.

## What NOT to do

- Don't replace `CLAUDE.md` with the `.toon` version — Claude Code parses markdown.
- Don't follow imports out of the repo.
- Don't run this against the **whole filesystem** — that's `claude-md-bulk-to-toon`.
