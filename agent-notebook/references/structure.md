# Notebook Structure — Hybrid Model

One folder per notebook inside the `agent-notebook/` hub root. The structure follows a
curated, exact layout (predictable and easily readable for LLMs even with many
notes); the processing vocabulary follows NotebookLM (transformations, insights,
citable notes).

## Hub layout

```
agent-notebook/
├── KNOWLEDGE.md           # Hub entrypoint: description, list of all notebooks
└── <slug>/               # One notebook per project
    ├── KNOWLEDGE.md       # Entrypoint: description, conventions, link to index
    ├── index.md           # Global index: pages, notes, insights
    ├── structure.md       # This layout and rules (copied from references/)
    ├── instructions.md    # Binding behavior rules (copied from references/)
    ├── sources/KNOWLEDGE.md  # Registry of untouched originals / doclib URIs
    ├── pages/KNOWLEDGE.md     # Curated pages (one per source)
    ├── notes/KNOWLEDGE.md     # Dictation and manual notes
    ├── insights/KNOWLEDGE.md  # Transformation results, subfolder per type
    ├── state.md           # Current standing + rolling monthly log
    └── state-archive/     # Rotated log archives (YYYY-MM.md; created on first rotation)
```

Every `KNOWLEDGE.md` requires YAML frontmatter with `name` and `description`
(routing hint, 5-300 characters). The notebook description must name its project.
The hub `KNOWLEDGE.md` lists every notebook with a one-line description.
Registries list entries as one-line links.

## Page schema (pages/)

One curated page per source, frontmatter plus fixed section order:

```markdown
---
id: <uuid v4>
type: <entity type: document, person, tool, ...>
title: "<title, any language>"
aliases: ["<translated or alternative titles>"]
source: <doclib-uri, file path, or URL; YAML list if multiple sources>
created: <YYYY-MM-DD>
created_time: <ISO 8601 date-time with timezone>
notebook: <slug>
tags: [<english, lowercase, kebab-case>]
---

# <Title>

## Summary
## Key Points
## Details
## Related
## References
- Source: <filename, doclib:// URI, or URL>
```

## Note schema (notes/)

```markdown
---
id: <uuid v4>
title: "<title, any language>"
aliases: ["<translated or alternative titles>"]
created: <YYYY-MM-DD>
created_time: <ISO 8601 date-time with timezone>
language: <ISO 639-1>
stt_engine: <stt engine, e.g. vibe-voice-mode>
source: dictation
device: <device or "chat">
notebook: <slug>
tags: [<english, lowercase, kebab-case>]
---

<Transcript as cleaned prose, paragraphs preserved>
```

No `duration_s` or `audio_file`: voice mode provides no audio file or duration
metadata. Never fabricate them.

## Insight schema (insights/)

```markdown
---
id: <uuid v4>
type: summary | keypoints | structure | style | custom-<name>
title: "<title, any language>"
aliases: ["<translated or alternative titles>"]
source: <doclib-uri or file path; YAML list if multiple sources>
created: <YYYY-MM-DD>
created_time: <ISO 8601 date-time with timezone>
notebook: <slug>
tags: [<english, lowercase, kebab-case>]
---

<Transformation result with source references>
```

Rules: an insight is citable, always names its source(s), is never written into
the source itself, and remains independently findable and deletable.

## State schema (state.md)

state.md is the compact, always-current orientation artifact of a notebook plus
a rolling activity log. It never grows unbounded: the standing block is
rewritten in place and the log rotates monthly.

```markdown
# State — <slug>

## Current standing

Stand: <1-3 sentences: where the topic stands right now>
Last updated: <YYYY-MM-DD>

### Open points
- <one line per open point>

### Next actions
- <planned actions, numbered>

## Log <YYYY-MM>
| Datum | Aktion | Ergebnis |
|---|---|---|
| ... | ... | ... |
```

Rules:

- The standing block (Stand, Open points, Next actions) is rewritten in place
  on every change; it is never appended to. Open points appear exactly once —
  here — never per log row.
- The log is append-only, three columns: date, action, result. One row per
  real action (create, change, decision, sync, audit).
- Routine audits get one short log row (date, consistency result, new findings
  only). Unchanged findings are implied by omission; their detail lives in the
  audit note, not in state.md.
- Rotation: on the first change of a new month, move the previous month's log
  table to `state-archive/<YYYY-MM>.md` (one file per month, H2 heading with
  month and notebook slug) and start a fresh log table. Safety valve: if the
  current-month log exceeds 200 rows, archive earlier rows up to the last month
  boundary — never split within a month.
- The `state-archive/` folder is created on first rotation. Archives are
  write-once: once a month is archived, it is never edited.
- The log table heading always names the current month (`## Log 2026-11`), so
  agents can detect a missed rotation at a glance.

Date fields: `created` is always date-only (`YYYY-MM-DD`) so Obsidian's Date
property type accepts it; the exact moment with timezone lives in `created_time`
(ISO 8601 date-time). Applies to note, page, and insight frontmatter alike.
When an existing file is edited after creation, set or refresh the optional
`updated: <YYYY-MM-DD>`. Migrated legacy files with unknown exact time may
omit `created_time`.

## Filename conventions

- `YYYY-MM-DD-<slug>.md` for notes and insights; date prefix ensures
  chronological sorting and Obsidian compatibility.
- The `YYYY-MM-DD` filename prefix always equals the `created` date.
- Slugs: ASCII, lowercase, hyphens, no spaces or special characters.
- Page files: `pages/<type>/<slug>.md` (no date prefix; pages are per source).
- Insight files: `insights/<type>/<YYYY-MM-DD>-<slug>.md`.
- State archives: `state-archive/<YYYY-MM>.md`, one file per calendar month.
- Entity-type subfolders (page types) use English names: document, person, tool, ...

## Interaction modes

| NotebookLM / OpenNotebook | Implementation in the notebook |
|---|---|
| Source | sources/ (untouched) or library document (URI referenced in the page) |
| Note (manual / from chat) | notes/ |
| Transformation | Skill workflow; creates an entry in insights/ |
| Insight | Entry in insights/ with type and source reference |
| Chat (full context) | library_read or read the full page |
| Ask (RAG) | library_search, preferably over consolidated pages |

## Sync and export

All notebooks live under the single `agent-notebook/` root, so the collection
downloads, zips, or syncs with a personal git repository in one step.
`state.md` and `state-archive/` are part of the sync by design: they are the
designated artifact for cross-agent orientation. An external agent reading a
synced clone (for example a CLI agent) reads one compact state.md per notebook
to learn where a topic stands; the standing block's last-updated date lets it
judge staleness against the sync schedule. State files are written by whichever
agent is currently working on the notebook, platform or external; any other
agent reading a synced clone treats them as its orientation and syncs with the
remote before it starts writing. Keep one writing agent at a time and sync
between handoffs to avoid conflicting updates. Runtime artifacts of the sync
itself (for example reconcile-state.json) are the only excluded files.

