---
name: batch-to-toon
description: Convert a directory of JSON / CSV / YAML files into TOON in one pass. Per-file round-trip checks, aggregate token-saving report, and a manifest so partial runs are resumable. Use when the user has a folder of structured data destined for LLM prompts.
---

# Batch → TOON

Convert many files at once.

## Inputs

1. **Source directory** — absolute path. Non-recursive by default; pass `--recursive` to walk subdirs.
2. **File types** — default: `.json`, `.csv`, `.yaml`, `.yml`. Limit with `--ext`.
3. **Output directory** — default: `<source>/toon/`. Mirrors the input layout.
4. **Backend** — default: read from plugin `config.json` (set by `setup-tooling`); fall back to `npx -y @toon-format/toon`.

## Workflow

1. Glob the source dir for matching files. Skip anything already present in the output dir with the same mtime (idempotent reruns).
2. Create or load a manifest at:

   ```
   ${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/state/<batch-id>.json
   ```

   Each row: input path, type, status (`pending` / `done` / `failed` / `skipped-no-savings`), original tokens, TOON tokens, saving %, error (if any).

3. For each file:
   - JSON → use `json-to-toon` skill logic (validate → encode → round-trip).
   - CSV → use `csv-to-toon` skill logic (CSV → JSON → encode).
   - YAML → parse to JSON first (`yq -o=json` or `python3 -c "import yaml,json; print(json.dumps(yaml.safe_load(open(p))))"`), then encode.
4. **Round-trip every file.** If any fails, mark `failed` in the manifest and continue — don't abort the batch.
5. **Token saving below 5%?** Mark `skipped-no-savings` and leave the original alone. TOON-ifying tiny savings is just churn.

## Final report

At the end, print:

- Total files processed / succeeded / failed / skipped.
- Aggregate tokens: total before, total after, absolute saving, % saving.
- Worst-case file (smallest or negative saving) and best-case file.
- Manifest path for inspection.

## Resumability

Re-running on the same source dir re-uses the existing manifest. `done` rows are skipped; `pending` and `failed` are retried. `--reset` forces a clean run.

## Failure modes

- YAML files with anchors / merge keys may not round-trip cleanly through JSON. Flag rather than silently lose data.
- CSV files with ragged rows (different column counts per row) won't form a uniform TOON table. Either pad or fail — surface the choice to the user once, then apply consistently across the batch.
