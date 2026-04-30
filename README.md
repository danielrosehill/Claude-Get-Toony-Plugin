# Claude Get Toony

A Claude Code plugin for converting JSON, CSV, YAML, and other structured data into [TOON](https://github.com/toon-format/toon) (Token-Oriented Object Notation) — a compact, lossless re-encoding of the JSON data model that uses ~40% fewer tokens than JSON when fed to LLMs.

## Skills

- `toon-reference` — what TOON is, when it helps, when it doesn't, and how to invoke the CLI. Read first.
- `setup-tooling` — install/verify TOON implementations across runtimes (JS, Python, Java, .NET, PHP, Rust) and record what's available in plugin config.
- `json-to-toon` — single-file JSON → TOON, with round-trip lossless check and token-saving report.
- `csv-to-toon` — single-file CSV → TOON via JSON. Useful when feeding tabular data to an LLM with explicit field/length guardrails.
- `batch-to-toon` — folder-level conversion with manifest, round-trip checks, resumability, and aggregate token-saving report.
- `claude-md-user-to-toon` — convert `~/.claude/CLAUDE.md` and its imported context files into TOON (sibling artefact, not a replacement).
- `claude-md-repo-to-toon` — convert a repo's CLAUDE.md (and nested ones) into TOON for context-cost analysis.
- `claude-md-bulk-to-toon` — system-wide pass: walks user-level config plus every repo under the user's repo roots and reports aggregate context-token cost.

## Ecosystem tools the plugin can install

| Runtime | Tool | Repo |
|---|---|---|
| Node / TS | `@toon-format/toon` (reference) | <https://github.com/toon-format/toon> |
| Python | `python-toon` | <https://github.com/xaviviro/python-toon> |
| Python | `toonify` | <https://github.com/ScrapeGraphAI/toonify> |
| Java | `toon-java` | <https://github.com/toon-format/toon-java> |
| Java | `json-io` (JSON pipe partner) | <https://github.com/jdereg/json-io> |
| .NET | `ToonSharp` | <https://github.com/0xZunia/ToonSharp> |
| PHP | `toon-php` | <https://github.com/HelgeSverre/toon-php> |
| PHP / Laravel | `laravel-toon` | <https://github.com/mischasigtermans/laravel-toon> |
| Rust | `toon-rs` | <https://github.com/jimmystridh/toon-rs> |
| Rust (CLI) | `error-toon` (browser error compression) | <https://github.com/adrozdenko/error-toon> |
| JS (app) | `Vidscriber` (video → JSON/TOON) | <https://github.com/SplitBN/Vidscriber> |

## Storage

Plugin config, tooling install dir, batch manifests, and cache live under:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/
```

User-facing TOON outputs go wherever the user requests (default: alongside the source).

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install get-toony@danielrosehill
```

## License

MIT
