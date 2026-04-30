---
name: json-to-toon
description: Convert a JSON file (or JSON on stdin / pasted into chat) into TOON format using the @toon-format/toon CLI. Verifies losslessness via round-trip decode and reports the token-count saving.
---

# JSON → TOON

Convert one JSON document into TOON.

## Inputs

1. **Source** — one of:
   - Absolute path to a `.json` file.
   - Inline JSON pasted into the conversation (write it to a temp file first).
   - URL — fetch with `curl` to a temp file.
2. **Output path** — optional. Default: same basename with `.toon` extension, in the same directory as the source. For pasted inline JSON, default to a temp file and print the result.

## Workflow

1. Validate the source is well-formed JSON: `python3 -c "import json,sys; json.load(open(sys.argv[1]))" <path>`. If it fails, surface the parser error and stop — don't try to "fix" the JSON.
2. Convert: `npx -y @toon-format/toon encode --input <src> --output <dst>` (or pipe via stdin if the input is inline).
3. **Round-trip check** — decode the result back to JSON and compare against the source with key-order canonicalisation:

   ```bash
   npx -y @toon-format/toon decode --input <dst> --output /tmp/toon-rt.json
   python3 -c "import json; a=json.load(open('<src>')); b=json.load(open('/tmp/toon-rt.json')); import sys; sys.exit(0 if a==b else 1)"
   ```

   If the round-trip differs, **stop and report the diff** — do not present a lossy output as success.

4. **Token delta** — count tokens on both files and report:

   ```bash
   # Prefer ttok if available, otherwise tiktoken
   npx -y ttok < <src>; npx -y ttok < <dst>
   ```

   Report: original tokens, TOON tokens, absolute delta, percentage saving.

5. Tell the user where the file is and the token saving. If the saving is negative or trivial (<10%), point it out — TOON isn't always the right format (deeply nested or non-uniform JSON; see `toon-reference`).

## Failure modes to watch

- Input has very deep nesting and few uniform arrays → TOON savings small or negative. Mention it.
- Input contains values that aren't representable in JSON (e.g. binary blobs as base64 strings) → fine, but flag if the user expected something else.
- Input has duplicate keys at the same object level → JSON technically allows this; the TOON encoder will collapse. Warn the user.
