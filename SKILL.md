---
name: field-manual
description: Generate a detailed, visually-designed "field manual" documentation artifact for a codebase — architecture, state machines, flow diagrams, procedures, config tables, reference surface. Trigger when the user asks for detailed/visual project documentation, an architecture doc, or explicitly invokes /field-manual.
---

# field-manual

Produces the same field manual content in two forms every run:

1. A published HTML **Artifact** — mental model, state machines,
   main-loop/protocol diagram, per-flow procedures, safety/guardrail
   systems, config tables, CLI/API reference, module map, all diagrammed
   with mermaid, using a fixed design system so every project's manual
   looks and reads like part of the same family of documents.
2. A **Markdown file committed to the project** — the persistent,
   version-controlled source of truth, since an Artifact is a session
   deliverable and doesn't live in the repo. Same content, same section
   order, mermaid diagrams as fenced ` ```mermaid ` code blocks (GitHub and
   most Markdown viewers render these natively).

Do not skip the research phase to save time. A field manual that's wrong
about the code is worse than no field manual — it's easier to trust than
the source, and doesn't say so.

## Configuration

Default output path: **`MANUAL.md`** at the project root.

Override it per run by passing a path as the skill's argument:

```
/field-manual docs/ARCHITECTURE.md
```

To set a standing default for a project (so every future run without an
explicit argument uses it), add a line to that project's `CLAUDE.md`:

```
field-manual output: docs/ARCHITECTURE.md
```

Resolution order: explicit argument > `CLAUDE.md` override > `MANUAL.md`
default. Always confirm the resolved path with the user before writing if
a file already exists there — overwrite only after they confirm, and offer
to diff/summarize what changed instead of silently clobbering hand-edited
content.

## Process

### 1. Research the project (read-only, no assumptions)

Read, in this order:

1. `CLAUDE.md` / `AGENTS.md` / `README.md` at the repo root — the project's
   own account of what it is and its standing rules.
2. The entry points (CLI parser, API routes, main loop, MCP server, worker
   entrypoint — whatever applies) to find every distinct operation the
   system exposes.
3. The data model / schema (DB migrations, ORM models, state enums) to find
   every stateful entity and its valid transitions. A `CHECK (status IN
   (...))` constraint or an enum is the state machine — read it from the
   schema, don't infer it from prose.
4. Any orchestration/decision logic (a scheduler, a queue consumer, a
   request router) that picks what happens next — this is usually the
   single most important diagram in the manual.
5. Config/defaults (env vars, a settings module, a defaults dict) for the
   reference tables.
6. `git log --oneline -- <path>` on anything you're about to describe as
   "current behavior," when the file looks like it might have changed
   recently — a stale doc that confidently describes removed behavior is
   worse than one that hedges.
7. Playbooks/skills/runbooks the project already has (a `skills/` dir, a
   `runbooks/` dir, docstrings describing a procedure) — these often
   already contain the exact step lists a "flow" section needs; don't
   rewrite them from scratch if they exist, adapt them into the numbered
   `ol.steps` format.

If the project is small enough to read in full, read it in full rather than
sampling. If it's large, prioritize breadth (one accurate paragraph per
subsystem) over depth on one subsystem — the manual's job is orientation,
not a full code walkthrough.

**Verify before asserting.** Any specific number, limit, or claim about
external behavior (a rate limit, a documented API cap, a third-party quota)
must come from the project's own code/config/tests, not from general
knowledge or a web search taken at face value — those are frequently wrong
or unverifiable, and asserting one as fact in a doc the user will rely on is
worse than admitting it's unknown. If something is genuinely unverifiable,
say so in the doc rather than picking a plausible-sounding number.

### 2. Structure the manual

Standard section set, in this order (skip a section only if the project
genuinely has nothing for it — never pad one with generic filler):

1. **Overview** (hero) — one-sentence identity, one-paragraph thesis (the
   single rule or mental model that explains most of the system's
   behavior), 2-4 role chips naming the major actors/components.
2. **Mental model** — one diagram: the actors/systems and how they hand off
   to each other. Usually the first thing to draw, often a `flowchart`.
3. **Core concepts** — a glossary grid of the project's own vocabulary
   (its nouns: what's a "campaign," a "job," a "tenant," a "session" —
   whatever the domain calls its central entities).
4. **State machines** — one `stateDiagram-v2` per stateful entity found in
   step 1.3. Label transitions with what actually causes them (a specific
   function/command/event), not a vague verb.
5. **Main protocol/loop** — if the system has a central loop, queue, or
   request/response protocol, diagram it as a `sequenceDiagram` (actors
   over time) or `flowchart` (branching decision order). If the project has
   a scheduler/dispatcher with an explicit priority order, diagram that
   order as its own flowchart — it's usually the highest-value diagram in
   the whole doc, because it's the thing that's hardest to reconstruct by
   reading code top to bottom.
6. **Per-flow procedures** — one subsection per distinct operation found in
   step 1.2, as a numbered `ol.steps` list (imperative voice, one action
   per step, edge cases as a `.sub` line under the step they affect) and/or
   a small diagram when the flow branches non-trivially.
7. **Safety / guardrail systems** — anything the project treats as an
   invariant: kill switches, rate limits, auth boundaries, idempotency
   guards, human-approval gates. State what each protects against and what
   triggers it, in a glossary grid or its own diagrammed section if the
   recovery path is non-trivial (see the sd-growth field manual's Pushback
   section for the pattern: state the mechanism, then an explicit callout
   admitting what's still unknown/unverified about it, rather than
   asserting confidence the code doesn't back up).
8. **Config / limits reference** — a table of every tunable default found
   in step 1.5, with what it controls in one clause.
9. **Integration flows** — if the project syncs to or from an external
   system (a CRM, a payment processor, another internal service), diagram
   that handoff and call out any "never guess a match" / idempotency
   guardrails the code enforces.
10. **API/CLI/interface surface** — a table mapping every command/endpoint/
    tool to its purpose. If there are two parallel interfaces to the same
    logic (e.g. a CLI and an MCP/API layer calling the same functions),
    show them side by side and say so explicitly — it's a useful invariant
    to name.
11. **Module map** — a table of source files/directories to what each owns,
    one clause each. This is the map a new reader uses to find where to
    make a change.
12. **Footer** — generation date, and the explicit disclaimer that the code
    is authoritative over the doc if they ever disagree.

Number sections sequentially through the whole document (01, 02, 03...),
not restarted per cluster. Use `<hr class="rule">` between major clusters
(e.g. between "concepts" and "flows", between "flows" and "safety"), not
between every section.

### 3. Build the HTML artifact

1. Copy `template.html` (next to this file) verbatim as the starting
   point — it is the fixed design system for every field manual this skill
   produces: IBM Plex Sans/Mono, the ink/paper + accent/warn/stop/ok token
   set, the sticky-TOC docs shell, diagram panels, state pills, glossary
   cards, numbered-step lists, and callout boxes. Do not re-derive a new
   palette or layout — the point of this skill is that every project's
   field manual reads as the same family of document. The only exception:
   if the project already has a strong existing visual identity the user
   would recognize (a product with real brand colors), ask whether to use
   the project's own palette instead of the default one before proceeding.
2. Fill in every `{{PLACEHOLDER}}` with real content from step 1 — never
   lorem ipsum, never a generic "Section content goes here." Follow the
   TOC-and-section pairing pattern shown in the template (build the `<nav>`
   links as you write each section's `id`).
3. Write mermaid diagrams as `<pre class="mermaid">...</pre>` blocks —
   Artifacts render mermaid natively, no extra library needed.
4. Load the `artifact-design` skill's fundamentals section only for the
   cross-cutting build rules that still apply on top of the fixed template
   (theme correctness, accessibility, no lorem, `overflow-x:auto` on wide
   content) — do not let it talk you into a different palette/type/layout
   for this skill; the design decision is already made by the template.
5. Publish via the `Artifact` tool. Title: the project's name (short noun
   phrase, no explainer suffix). Description: one sentence naming what the
   doc covers. Favicon: one emoji that fits the project's domain (pick
   fresh per project, not always the same one).

### 4. Write the Markdown file

1. Resolve the output path per **Configuration** above.
2. Convert the same section content to plain Markdown — same order, same
   headings (`##` per numbered section, matching the artifact's numbering
   in the heading text, e.g. `## 05 — Main protocol`), same tables, same
   mermaid sources as ` ```mermaid ` fences. Glossary cards become a
   definition list or a table; numbered procedures become ordered lists;
   callouts become blockquotes (`> **Known unknown:** ...`).
3. Add one line under the title: `_Generated by the field-manual skill from
   the source tree as of {{DATE}}. See [{{artifact title}}]({{artifact
   URL}}) for the designed version._` — link the two together.
4. Write the file with the Write/Edit tool. This is a real commit to the
   working tree, so it follows the project's own commit-hygiene rules (see
   its `CLAUDE.md`) — don't commit it yourself unless the user's standing
   rules say to commit freely; report that it's written and let normal
   commit review happen.

## Output contract

Both the Artifact and the Markdown file must, at minimum:

- Use `template.html`'s CSS and layout shell unmodified for the Artifact
  (content only); the Markdown file carries the same content structure
  without the CSS, since Markdown has no styling of its own.
- Cover every section in step 2's list that has real material in the repo.
- Contain at least one `stateDiagram-v2` (if the project has stateful
  entities), one `flowchart` (the main decision/protocol logic), and one
  reference table (config or API surface) — a manual with zero diagrams
  has not done the job this skill exists for.
- Contain zero placeholder/lorem text and zero claims not traceable to the
  actual source tree.
- Stay in sync with each other — they are two renderings of one set of
  facts gathered once in step 1, never independently drifting drafts.

If the project genuinely lacks material for a section (e.g. no external
integrations), omit that section rather than inventing content — say
nothing rather than pad.
