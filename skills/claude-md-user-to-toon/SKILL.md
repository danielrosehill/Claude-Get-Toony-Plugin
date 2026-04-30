---
name: claude-md-user-to-toon
description: Convert the user-level `~/.claude/CLAUDE.md` (and any imported / nested context files it references) into TOON. Parses markdown structure (headings, lists, tables, key-value pairs) into a JSON tree, then encodes to TOON. Saves the converted bundle as a sibling artefact — does NOT overwrite CLAUDE.md, since Claude Code expects markdown.
---

# User CLAUDE.md → TOON

Convert the user-level Claude Code instructions to TOON for token-saving experiments.

## What gets converted

1. **Primary file**: `~/.claude/CLAUDE.md`.
2. **Imported context**: anything referenced from CLAUDE.md via `@<path>` or markdown links into `~/.claude/context/`, `~/.claude/memory/`, etc. Walk the references one level deep. Don't follow links outside `~/.claude/`.
3. **Memory index**: `~/.claude/projects/-home-daniel/memory/MEMORY.md` (and any per-user memory dir Claude Code uses) — convert in the same pass.

## Caveat to surface to the user up front

CLAUDE.md is **prose-with-markdown**, not JSON-shaped data. TOON's win comes from uniform tabular arrays. For prose, expect modest savings (5–15%); the gains come mostly from bullet lists, tables, and key-value sections. If the user's CLAUDE.md is mostly paragraphs, say so before proceeding.

Claude Code itself expects CLAUDE.md to be markdown — **never replace the original with TOON**. The output is an alternate artefact for prompt-stuffing experiments, not a live config swap.

## Workflow

1. **Read** the primary file plus any one-level imports. Assemble an in-memory map: `{file_path: raw_markdown}`.
2. **Parse markdown to JSON** for each file. Use a structure like:

   ```json
   {
     "path": "~/.claude/CLAUDE.md",
     "sections": [
       {
         "heading": "Working Philosophy",
         "level": 2,
         "prose": "...",
         "lists": [["..", ".."]],
         "tables": [{"headers": [...], "rows": [...]}],
         "subsections": [...]
       }
     ]
   }
   ```

   Use `markdown-it` (Node) or `mistune` / `markdown-it-py` (Python). Don't roll your own parser.

3. **Encode to TOON** via the configured backend (read `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/config.json` for the default; fall back to `npx -y @toon-format/toon`).
4. **Round-trip check** — decode TOON, compare against the parsed JSON tree. If lossless, proceed. If not, surface the diff and stop.
5. **Token delta** — count tokens on the original markdown vs the TOON bundle. Report per-file and aggregate.

## Output location

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/data/user-claude-md/<timestamp>/
├── CLAUDE.md.toon
├── context/<file>.toon          # for each imported context file
├── memory/<file>.toon
├── manifest.json                # input paths, token deltas, parser version
└── round-trip-check.txt         # PASS / per-file diffs if any
```

Tell the user the dir, the aggregate token saving, and how to feed the TOON bundle into a prompt experiment manually. Don't auto-modify any Claude Code config.

## What NOT to do

- Don't overwrite `~/.claude/CLAUDE.md`.
- Don't convert and then `import` the TOON file from CLAUDE.md — Claude Code's import system expects markdown.
- Don't follow `@` references outside `~/.claude/` (e.g. project-level CLAUDE.md files in `~/repos/`) — that's the job of `claude-md-repo-to-toon`.
