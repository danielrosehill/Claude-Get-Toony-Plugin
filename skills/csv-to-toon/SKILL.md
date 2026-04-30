---
name: csv-to-toon
description: Convert a CSV file into TOON. CSV is already token-efficient for flat tables, but TOON adds explicit array-length and field-header guardrails that improve LLM accuracy on the same data. Use this when the destination is an LLM prompt, not when CSV itself is the final consumer.
---

# CSV → TOON

Convert a CSV file into TOON. The reference CLI doesn't accept CSV directly, so this skill goes CSV → JSON → TOON.

## Inputs

1. **Source CSV** — absolute path or URL.
2. **Has header row?** — default: yes. If no, the user must supply field names.
3. **Delimiter** — default `,`. Detect TSV (`\t`) automatically by extension.
4. **Type coercion** — default: try to coerce numerics and booleans (`true`/`false`/`null`); fall back to strings. The user can disable to keep everything as strings.
5. **Output path** — default: same basename with `.toon` extension.

## Workflow

1. Validate the CSV parses cleanly:

   ```bash
   python3 -c "import csv,sys; list(csv.DictReader(open(sys.argv[1])))" <src>
   ```

2. Convert to a JSON array of objects (this is the "obvious" CSV-to-JSON shape and the one TOON's tabular form is built for):

   ```python
   import csv, json
   with open(src) as f:
       rows = list(csv.DictReader(f, delimiter=delim))
   # type-coerce numerics / booleans / nulls if enabled
   json.dump(rows, open(tmp_json, 'w'))
   ```

3. Encode to TOON:

   ```bash
   npx -y @toon-format/toon encode --input <tmp_json> --output <dst>
   ```

   For uniform CSV-derived data, TOON should pick the tabular `arr[N]{f1,f2,...}` form automatically — that's the whole point.

4. **Round-trip check** — decode TOON, re-emit as CSV with the same column order, diff against the original (after stripping trailing newlines / re-quoting differences).

5. **Token delta** — count tokens on `<src>` and `<dst>` and report. CSV is already compact, so the absolute saving is usually small but the structural guardrails matter for LLM accuracy. State both.

## When to push back

If the user wants this purely so a downstream non-LLM consumer can read TOON instead of CSV, that's almost certainly a mistake — say so and suggest staying on CSV. TOON's value is on the LLM-input side.
