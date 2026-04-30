---
name: structure-long-prompt
description: Use when the user wants to convert CLAUDE.md, a system prompt, or any long instruction file into TOON — or asks how to compress / restructure a long prose prompt. Pushes back on TOON for prose (wrong tool) and recommends XML-tag section delimiters, the Anthropic-recommended pattern for long/complex prompts. Acts as a guardrail against TOON-ifying instruction files where the format would degrade comprehension.
---

# Structure a Long Prompt (don't TOON-ify it)

This skill exists because TOON is the **wrong tool** for prose instruction files (CLAUDE.md, system prompts, long user prompts) — but the instinct to "compress my long prompt" is real and valid. This skill redirects that instinct to a better technique: **XML section tags**.

## When this skill fires

Trigger when the user says any of:

- "convert my CLAUDE.md to TOON"
- "TOON-ify my system prompt / instructions / long prompt"
- "how do I compress this long prompt"
- "shorten my CLAUDE.md"
- anything that pairs *prose instructions* with *TOON / structured-format conversion*

Do **not** fire for: data files (JSON/CSV/YAML), configuration, or repeating-record content. Those go to the other skills in this plugin.

## What to tell the user (deliver this verbatim or close)

> TOON is built for **structured data** — uniform records with shared keys (inventory rows, log entries, dataset slices). Its compactness comes from sharing field headers across many records.
>
> Instruction files are **prose directives**. There's no repeating-record structure for TOON to compress. Converting prose to TOON either forces it into awkward key-value shapes (which degrades how reliably the LLM follows the instructions) or just wraps strings in TOON scaffolding (which adds tokens and saves nothing).
>
> Will Claude *parse* TOON in a CLAUDE.md? Yes — Claude reads whatever bytes are in the file. But "parses" ≠ "follows as well as prose." LLMs are trained on enormous markdown/prose corpora; they internalize directives in that form most reliably.

## The actually-useful technique: XML section tags

For long or complex prompts where sections are getting blurred or ignored, **wrap each section in XML tags**. This is Anthropic's own recommended pattern (see the prompt-engineering guide on `<thinking>`, `<example>`, etc.).

Why it works:

- Hard section boundaries — the model can't bleed one rule into another.
- Tags are **named** — easy to reference later in the prompt ("apply the rules in `<autonomy>` here").
- Token cost is roughly identical to markdown headings.
- Mixes cleanly with markdown inside each tag.

### Example transformation

**Before** (markdown headings):

```markdown
## Autonomy Directive
Just keep going. Don't ask permission...

## Naming
Repos use Train-Case...

## Plans
Save plans to planning/ in the current repo...
```

**After** (XML-tagged sections):

```markdown
<autonomy>
Just keep going. Don't ask permission...
</autonomy>

<naming>
Repos use Train-Case...
</naming>

<plans>
Save plans to planning/ in the current repo...
</plans>
```

Markdown inside the tags still renders/reads fine. The tags give Claude unambiguous section boundaries.

### When to apply it

XML tags are a **surgical fix**, not a wholesale rewrite. Recommend them when:

- The prompt is long enough that sections are getting confused or ignored.
- A specific directive keeps being missed despite being in the file.
- Sections have very different character (rules vs. context vs. examples) and you want the model to treat them differently.
- The user wants to *reference* a section by name later in the same prompt or in a tool result.

For a short CLAUDE.md (say, < 50 lines of prose), markdown headings are fine — don't churn it.

## What this skill does, operationally

1. Confirm the user's actual goal — "compress / shorten" vs. "make it more reliably followed."
2. Explain why TOON is the wrong tool (use the block above).
3. If they want better adherence on a long prompt: offer to add XML section tags to a specific file. Read the file, propose tag names per section, apply with `Edit`.
4. If they just want it shorter: that's a content question, not a format question. Offer to identify redundant or stale directives.
5. Never produce a TOON version of a prose instruction file, even if the user insists — explain that the plugin deliberately doesn't do that, and point at the XML approach instead.

## What to leave alone

- Tabular fragments inside CLAUDE.md (e.g. a context-files table) — markdown tables are already efficient and Claude reads them fine. Don't TOON-ify them either; they're tiny.
- Memory pointer files (`MEMORY.md` index) — already a one-line-per-entry list, no benefit from restructuring.

## Other formats considered (and rejected for prose)

| Format | Verdict for instruction files |
|---|---|
| Markdown | **Best default.** Heavily represented in LLM training data. |
| XML tags | **Best for long/complex prompts** with hard section boundaries. |
| Plain prose | Fine but loses scannable structure. |
| YAML | Good for config (key: value), bad for prose — strings get awkward. |
| TOON / JSON | Wrong tool. Built for data. |
| TOML | Same problem as YAML for prose. |

The recommendation hierarchy is: **markdown → markdown + XML tags for long/complex → never a data format**.
