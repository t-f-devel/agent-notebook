---
name: agent-notebook
description: Set up and operate a NotebookLM-style notebook for any agent that speaks SKILL.md (built on Mistral Vibe Work; plain Markdown, no server, no database). Use for creating a notebook, ingesting sources as curated pages, saving dictation notes (voice mode), and running transformations that produce citable insights. Triggers are recognized in any language; examples include "notebook anlegen"/"create notebook", "ingest source", "save dictation", "transform source", "summarize", "keypoints", "insight erstellen"/"create insight".
---

# Agent Notebook

A NotebookLM-style notebook built entirely on plain Markdown: one folder per
notebook inside the `agent-notebook/` root, curated pages per source, dictation
notes via voice mode (for example Mistral Vibe Work voice mode), and
transformations that produce citable insights. The notebook is just Markdown
files your agent reads and writes; there is no server or database to run.

Load `references/structure.md` for the notebook layout and all schemas, and
`references/instructions.md` for binding behavior rules (citation, language,
index care) before performing any notebook operation.

## Conventions (binding)

- Frontmatter keys and enumerated values are English and lowercase:
  `type: summary | keypoints | structure | style | custom-<name>`.
- Tags are English, lowercase, kebab-case.
- Trigger phrases are intent-based and recognized in any language;
  trigger phrases quoted in this skill are examples.
- Dates are ISO 8601; filenames use a `YYYY-MM-DD-` date prefix plus a slugified
  title (ASCII, lowercase, hyphens), for example `2026-10-11-test-note.md`.
  In frontmatter, `created` is date-only (`YYYY-MM-DD`, matching Obsidian's
  Date property) and `created_time` carries the full ISO 8601 date-time with
  timezone. This keeps files compatible with external systems such as Obsidian.
- Note and insight content stays in the language of its source; titles may be any
  language. Use the `aliases` frontmatter field for multilingual titles, e.g.
  `aliases: ["Idea: minor characters", "Idee: Nebendarsteller"]`.
- Sources are never modified, moved, or deleted.
- Every page carries a provenance line; every insight names its source(s).
  Never fabricate content.
- After every change, update `index.md` and `state.md` in the notebook:
  rewrite the current-standing block in place and append one row to the
  monthly log (state schema in `references/structure.md`).

## Layout: notebooks hub

All notebooks live in a single hub root, one folder per notebook. This keeps
every notebook under one root for download, zip export, or git sync:

```
agent-notebook/
├── KNOWLEDGE.md           # Hub entrypoint: description and list of notebooks
├── <slug>/                # One notebook per project
│   ├── KNOWLEDGE.md       # Notebook entrypoint: description, conventions, index link
│   ├── index.md           # Global index: pages, notes, insights
│   ├── structure.md       # Copied from references/structure.md
│   ├── instructions.md    # Copied from references/instructions.md
│   ├── sources/KNOWLEDGE.md
│   ├── pages/KNOWLEDGE.md
│   ├── notes/KNOWLEDGE.md
│   ├── insights/KNOWLEDGE.md
│   ├── state.md           # Current standing + rolling monthly log
│   └── state-archive/     # Rotated log archives (YYYY-MM.md; created on first rotation)
└── <another-slug>/
```

The hub `KNOWLEDGE.md` lists every notebook with a one-line description so
routing can find the right notebook; the `notebook` frontmatter value of each
note and insight is the bare `<slug>`.

## Setup (create a new notebook)

1. If the `agent-notebook/` root does not exist, create it with a
   hub `KNOWLEDGE.md` (frontmatter: name `notebooks`, description naming the
   hub and its purpose; body: list of notebooks, initially the new one).
2. Create `agent-notebook/<slug>/` with the layout defined in
   `references/structure.md` (KNOWLEDGE.md entrypoint, index, structure,
   instructions, sources/, pages/, notes/, insights/, state.md with an empty
   standing block and an empty log table for the current month). The notebook
   description must name the project it belongs to.
3. Copy `references/structure.md` into the notebook as `structure.md` and
   `references/instructions.md` as `instructions.md`.
4. Add the notebook to the hub `KNOWLEDGE.md` list.
5. Confirm setup to the user with root path, notebook slug, and layout.

## Resolve the target notebook

Every save operation needs exactly one target notebook. Resolve it explicitly
before saving anything, never implicitly:

1. Read the hub `KNOWLEDGE.md` and match the content (source, note, or
   transformation request, together with its project or user context) against
   each notebook's one-line description.
2. Exactly one clear match: save to that notebook and name it in the
   confirmation.
3. Multiple plausible matches: list the candidates and ask the user to choose.
   Never guess.
4. No match: propose creating a new notebook (see Setup) and wait for explicit
   confirmation. Never create a notebook silently.
5. An explicit user instruction (named notebook or project context) always
   overrides the routing proposal.

## Ingest a source

1. Resolve the target notebook per "Resolve the target notebook" above.
2. Resolve the source: a document library URI (for example doclib://...), a URL,
   or an uploaded file. Library documents stay in the library; record their
   URI in the notebook source registry. Files are copied unchanged into
   `sources/`.
3. Read the full source (library_read or file read) — never work from snippets.
4. Create a curated page `pages/<type>/<slug>.md` with the page frontmatter
   schema and the fixed section order: Summary, Key Points, Details, Related,
   References (provenance line last).
5. Update `index.md`, the pages registry, and `state.md`.

## Save a dictation

1. Trigger: a voice-mode transcript plus a request to save it as a note.
2. Resolve the target notebook per "Resolve the target notebook" above.
3. Clean STT artifacts without changing meaning; derive title, tags, language.
4. Save as `notes/YYYY-MM-DD-<slug>.md` with the note schema in
   `references/structure.md` (`source: dictation`, the `stt_engine` value
   naming the engine used; no fabricated duration or audio fields).
5. Update `index.md` and the notes registry; confirm filename, title, tags.

## Run a transformation

1. Trigger: transform/summarize/keypoints/structure/style/custom requests.
2. Resolve the target notebook per "Resolve the target notebook" above.
3. Load the target source(s) fully, then produce the result grounded in the
   source only.
4. Save as `insights/<type>/<YYYY-MM-DD>-<slug>.md` with the insight schema
   (English type value, source URI(s) in frontmatter).
5. Update `index.md` and the insights registry; confirm briefly.

## Maintenance

Consolidation and lint follow the wiki pipeline stages (consolidate, lint,
repair): detect near-duplicates, check for broken links, orphans, and missing
sections; dedupe or delete only with explicit user confirmation.

## Sync and export

All notebooks live under the single hub root, so the entire notebook
collection can be downloaded, zipped, or synced with a personal git repository
in one step. `state.md` and `state-archive/` are part of the sync by design:
they are the designated artifact for cross-agent orientation. An external
agent reading a synced clone (for example a CLI agent) reads one compact
`state.md` per notebook to learn where a topic stands; the standing block's
last-updated date lets it judge staleness against the sync schedule.
State files are written by whichever agent is currently working on the
notebook, platform or external. Any other agent reading a synced clone treats
them as its orientation and syncs with the remote before it starts writing.
Keep one writing agent at a time and sync between handoffs to avoid
conflicting updates.
