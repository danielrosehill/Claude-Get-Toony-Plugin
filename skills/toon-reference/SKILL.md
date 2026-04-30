---
name: toon-reference
description: Reference for the TOON (Token-Oriented Object Notation) format — what it is, when it helps, when it doesn't, and how to invoke the @toon-format/toon CLI. Read this before using the other skills in this plugin if the user is unfamiliar with TOON.
---

# TOON Reference

**TOON (Token-Oriented Object Notation)** is a compact, lossless re-encoding of the JSON data model designed for LLM input. It combines YAML-style indentation for nesting with CSV-style tables for uniform arrays. Spec: <https://github.com/toon-format/spec>. Reference implementation: <https://github.com/toon-format/toon>.

## Why convert to TOON

- **~40% fewer tokens** than pretty JSON on mixed-structure benchmarks; comparable accuracy or better.
- **Lossless** for the JSON data model — round-trips back to the same JSON.
- **LLM-friendly guardrails** — `[N]` array lengths and `{fields}` headers give the model an explicit schema.

## When TOON helps

- Uniform arrays of objects (table-shaped data) — biggest savings.
- Mixed structures with some uniform arrays.
- Anything you'd otherwise drop into a prompt as JSON.

## When TOON does NOT help (or hurts)

- **Pure flat tabular data** — CSV is smaller than TOON; the structural overhead isn't worth it.
- **Deeply nested, non-uniform structures** — compact JSON often wins on tokens.
- **Latency-critical local models** — measure first; some quantized models process compact JSON faster.

If the user's data is one of those, say so before converting.

## CLI invocation

The reference implementation is a Node CLI: `@toon-format/toon`.

Install on demand (no global pollution):

```bash
npx -y @toon-format/toon --help
```

Common forms:

```bash
# JSON → TOON (stdin/stdout)
cat data.json | npx -y @toon-format/toon encode > data.toon

# TOON → JSON (round-trip)
cat data.toon | npx -y @toon-format/toon decode > data.json

# File in / file out
npx -y @toon-format/toon encode --input data.json --output data.toon
```

If the user prefers a persistent install, suggest `npm i -g @toon-format/toon` and then use `toon` directly. Don't install globally without asking.

## Output spot-check

After conversion, ALWAYS:

1. Round-trip: decode the produced `.toon` back to JSON and `diff` (after canonicalising key order) against the original. Must be lossless.
2. Show the user a token-count delta — count tokens on the original vs the TOON output (e.g. via `tiktoken` or `npx ttok`) and report the saving in absolute and percentage terms.

If the round-trip fails, surface the diff to the user and do not silently proceed.

## File extension and media type

- Extension: `.toon`
- Provisional media type: `text/toon` (UTF-8)

## Storage

This plugin writes nothing persistent. If the user wants to cache conversion artefacts (rare), put them at:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/cache/
```
