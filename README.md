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
config, then produce:

- a published single-page HTML **Artifact** covering overview & mental
  model, core concepts, state machines, the main protocol/decision-flow
  diagram, per-flow procedures, safety/guardrail systems, config & limits,
  integration flows, CLI/API surface, and a module map;
- a **`MANUAL.md`** committed to the project root (configurable path) —
  the same content, version-controlled;
- optionally, a **PDF** — pass `--pdf` (`/field-manual --pdf`) or set a
  standing `field-manual pdf: true` in the project's `CLAUDE.md`. Off by
  default; needs a local Chromium/Chrome for the print-to-PDF render.

## Files

- `SKILL.md` — the skill definition: research process, section structure,
  build instructions, output contract.
- `template.html` — the fixed CSS design system and page shell the
  Artifact is built from.
- `print.html` — the standalone (self-contained, non-Artifact-hosted)
  version of that same design used for PDF rendering.

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
