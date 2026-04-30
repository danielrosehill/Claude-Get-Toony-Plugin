---
name: setup-tooling
description: Install or verify TOON ecosystem tooling — the JS reference CLI (@toon-format/toon), Python implementations (xaviviro/python-toon, ScrapeGraphAI/toonify), the Java port (toon-format/toon-java), and adjacent JSON libraries like jdereg/json-io. Picks the right tool for the user's runtime and records what's installed in plugin config.
---

# Setup TOON Tooling

Bootstrap the local environment with one or more TOON implementations so other skills in this plugin (and the user's own scripts) can call them.

## Registered implementations

| Tool | Language | Purpose | Repo |
|---|---|---|---|
| `@toon-format/toon` | Node/TS | Reference CLI + library. Default. | <https://github.com/toon-format/toon> |
| `python-toon` | Python | Pure-Python encode/decode. Use when the project is Python and Node isn't desired. | <https://github.com/xaviviro/python-toon> |
| `toonify` | Python | ScrapeGraphAI's higher-level toolkit — convenience wrappers, dataset helpers. Use for batch / pipeline work in Python. | <https://github.com/ScrapeGraphAI/toonify> |
| `toon-java` | Java | JVM port of the spec. Use inside JVM projects. | <https://github.com/toon-format/toon-java> |
| `json-io` | Java | Not TOON itself — robust JSON ↔ Java object library. Useful as the JSON side of a TOON pipeline on the JVM (parse → re-emit → feed to toon-java). | <https://github.com/jdereg/json-io> |
| `ToonSharp` | C# / .NET | TOON encode/decode for .NET projects. | <https://github.com/0xZunia/ToonSharp> |
| `toon-php` | PHP | TOON encode/decode for PHP. Use for plain-PHP projects. | <https://github.com/HelgeSverre/toon-php> |
| `laravel-toon` | PHP (Laravel) | Laravel-flavoured wrapper around toon-php — service provider, facade, response macro. | <https://github.com/mischasigtermans/laravel-toon> |
| `toon-rs` | Rust | Rust implementation with full serde integration. | <https://github.com/jimmystridh/toon-rs> |
| `Vidscriber` | JS | Application — produces LLM-friendly JSON/TOON transcriptions of videos for editing tasks. Not a generic TOON encoder, but a downstream consumer worth knowing about for video-pipeline contexts. | <https://github.com/SplitBN/Vidscriber> |
| `error-toon` | Rust | Application — compresses verbose browser errors via TOON before pasting to LLMs. Saves 70-90% tokens on stack traces. Adjacent tool, not a library. | <https://github.com/adrozdenko/error-toon> |

## Inputs

Ask the user (or accept from arguments) which targets to set up. Sensible default: just install the JS CLI on demand via `npx`, no global state. Options:

- `--js` — confirm Node available; cache the package via `npx -y @toon-format/toon --version`.
- `--python` — install `python-toon` (and optionally `toonify`) into the user's preferred Python env.
- `--java` — fetch the latest `toon-java` JAR (and optionally `json-io`) into the plugin's tooling dir.
- `--dotnet` — add `ToonSharp` to a .NET project via `dotnet add package` (or fetch as a tool — depends on how the upstream ships).
- `--php` — install `helgesverre/toon-php` via Composer; for Laravel projects, also pull `mischasigtermans/laravel-toon` and register the service provider.
- `--rust` — add `toon-rs` to a Rust project via `cargo add toon` (confirm crate name from the repo before running) or install adjacent CLIs like `error-toon` via `cargo install`.
- `--all` — set up everything available for runtimes detected on this machine.

`Vidscriber` is an application, not a library — set it up only if the user is doing video-transcript work; otherwise skip.

## Workflow

### JS (default)

```bash
node --version || { echo "Node not found — install it first"; exit 1; }
npx -y @toon-format/toon --version
```

That's it — `npx` will cache it. Suggest `npm i -g @toon-format/toon` only if the user explicitly asks for a persistent CLI.

### Python

Ask the user which environment manager they prefer (`uv`, `pipx`, system `pip`, project venv). Default for one-off CLI use: `pipx`.

```bash
# Reference Python implementation
pipx install python-toon
python-toon --version

# Optional: ScrapeGraphAI's higher-level toolkit
pipx install toonify
```

For project-local use, install into the project's venv with `uv add` / `pip install` instead of pipx.

### Java

Tooling lives under the plugin data dir to avoid polluting `~/.m2` or the project:

```bash
TOOL_DIR="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/tooling/jvm"
mkdir -p "$TOOL_DIR"
```

Fetch the latest release JAR for `toon-format/toon-java` (and `jdereg/json-io` if requested) via `gh release download` or `curl` against the GitHub releases API. Don't guess version numbers — query the releases endpoint:

```bash
gh release list --repo toon-format/toon-java --limit 1
gh release download --repo toon-format/toon-java --pattern '*.jar' --dir "$TOOL_DIR"
```

Verify with `java -jar <jar> --help` (or whatever the project's invocation is — check the repo README first).

## Record what's installed

After setup, write a small inventory so other skills can pick the right backend without re-detecting every time:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/get-toony/config.json
```

Schema:

```json
{
  "backends": {
    "js": {"available": true, "version": "x.y.z", "invocation": "npx -y @toon-format/toon"},
    "python": {"available": true, "version": "x.y.z", "invocation": "python-toon"},
    "toonify": {"available": false},
    "java": {"available": true, "version": "x.y.z", "jar_path": "/.../toon-java-x.y.z.jar"},
    "json-io": {"available": false}
  },
  "default_backend": "js",
  "last_checked": "2026-04-30T00:00:00Z"
}
```

Other skills (`json-to-toon`, `csv-to-toon`, `batch-to-toon`) should read `default_backend` and fall back gracefully if their preferred backend isn't installed.

## Verification

Run a tiny round-trip on each installed backend with a fixture:

```json
{"hikes":[{"id":1,"name":"Blue Lake"},{"id":2,"name":"Ridge"}]}
```

Encode → decode → compare. If any backend's round-trip differs from the source, mark it `available: false` in `config.json` and tell the user which one failed.
