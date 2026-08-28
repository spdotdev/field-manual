# field-manual

A Claude Code skill that generates a detailed, visually-designed "field
manual" documentation artifact for a codebase: mental model, state
machines, flow diagrams, procedures, safety systems, config tables, and a
CLI/API reference — published as a self-contained HTML artifact with a
fixed design system (IBM Plex Sans/Mono, sticky table of contents, mermaid
diagrams, light/dark theme support) so every project's manual reads as
part of the same family of documents.

## Install

Copy this repo into your Claude Code skills directory:

```sh
git clone https://github.com/spdotdev/field-manual.git ~/.claude/skills/field-manual
```

Or, for one project only, into that project's `.claude/skills/field-manual`.

## Use

From inside a project, ask Claude to write project documentation, an
architecture doc, or invoke the skill directly:

```
/field-manual
```

Claude will read the project's own docs, entry points, data model, and
config, then produce and publish a single-page HTML artifact covering:

- Overview & mental model
- Core concepts glossary
- State machines (one per stateful entity)
- Main protocol/loop diagram (sequence or decision-flow)
- Per-flow procedures (numbered steps)
- Safety/guardrail systems
- Config & limits reference
- Integration flows (if any)
- CLI/API surface reference
- Module map

## Files

- `SKILL.md` — the skill definition: research process, section structure,
  build instructions, output contract.
- `template.html` — the fixed CSS design system and page shell every
  generated manual is built from.

## Design system

- **Type**: IBM Plex Sans (headings/body) + IBM Plex Mono (code, state
  pills, data).
- **Color**: warm paper/near-black ink neutrals with a deep teal accent;
  amber/brick-red/sage for warn/stop/ok states — themed for both light and
  dark, following the OS preference by default.
- **Layout**: sticky left-nav docs shell, diagram panels, glossary cards,
  numbered procedure lists, reference tables.

The template is deliberately fixed across projects — swap content, not the
visual identity — so a field manual is recognizable as one regardless of
which repo it documents.
