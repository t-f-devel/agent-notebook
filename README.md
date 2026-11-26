# agent-notebook

**NotebookLM-style notebooks for Mistral Vibe Work and any agent that speaks
SKILL.md**

agent-notebook turns your agent into a NotebookLM-style research workspace; the
reference is Google's Gemini Notebook, formerly
[NotebookLM](https://en.wikipedia.org/wiki/NotebookLM). Collect sources, curate
them into structured pages, capture dictations via voice mode, run
transformations that produce citable insights, and keep a compact state file
per notebook for cross-agent sync. No Docker, no database, no vendor lock-in:
notebooks are plain Markdown, Obsidian-compatible, and zip, sync, or git-push
in one step. Built for Mistral Vibe Work; the skill folder follows the open
Agent Skills standard (`SKILL.md`), so it also loads in compatible agents such
as Claude Code or Codex.

*Not affiliated with Google. NotebookLM and Gemini Notebook are Google
trademarks.*

## Privacy and data residency

The agent platform sees every notebook. Whatever your agent can read, the
platform operator can read, so its data and training policies apply while you
work there. The counterweight is the file format: notebooks are plain Markdown
that lives on your side. Sync them to your own git repository, self-hosted on
your own server (for example Forgejo) if you want zero third parties, and your
knowledge stays yours, readable without the platform.

My personal choice is Mistral, a European provider under GDPR with EU data
residency. If you run this skill on another SKILL.md-compatible agent, check
its data policies first: the notebook format protects your sovereignty over
the files, not the transport into the model.

## Installation

1. Get this repository into your workspace: clone it, download the ZIP, or
   ask your agent in chat to fetch it from GitHub.
2. Copy the skill folder into your environment's skills directory:

```
agent-notebook/  →  skills/agent-notebook/
```

The skill folder needs only its `SKILL.md` and `references/`. That is all:
skills activate through their trigger phrases in chat, and there is no further
configuration. Voice mode is optional and only needed for dictation notes.

## Setup and wiring

The skill is deliberately lean: no bootstrap machinery, no first-run wizard,
no automatic notebook detection. Two steps and your project has a durable
memory.

1. Create a notebook. In the project chat, say `notebook anlegen` /
   `create notebook`. The skill creates the notebooks root `agent-notebook/`
   with its hub entrypoint on first use, plus the notebook folder: entrypoint,
   index, structure, instructions, registries for sources, pages, notes, and
   insights, and a state file.
2. Wire the project so future sessions find the notebook on their own: put one
   pointer into your project instructions, for example:

```
Project work and context: Notebook "<slug>" in the agent-notebook hub.
For every task, consult the notebook's index.md first; binding rules are in
its structure.md and instructions.md.
```

From then on the notebook is the project's durable memory: chat answers stay
short, the files are the artifact, and everything cites its sources. Wiring
is manual on purpose; you decide which projects get a notebook.

## Daily use

Every session starts with the notebook, not with a search. At session start
the agent consults the project notebook's `index.md` (what exists) and
`state.md` (where things stand, open points, next actions) and picks up where
you left off. Saves are routed to exactly one notebook: the agent matches the
content against the hub `KNOWLEDGE.md` descriptions and names the notebook it
wrote to. If multiple notebooks match, it asks instead of guessing.

| You say | Skill does | Artifact |
|---|---|---|
| `notebook anlegen` / `create notebook` | Creates the notebooks root (first use) and the notebook folder: entrypoint, index, structure, instructions, registries, state | Notebook in `agent-notebook/<slug>/` |
| `ingest source`, or share a document | Reads the full source, never snippets; files it under `sources/` or records its library URI; writes one curated page | Page in `pages/<type>/` with provenance line |
| Ask in chat, or via semantic search | Full source context (Chat mode) or search over the consolidated pages (Ask mode) | Chat answer citing its source |
| `save dictation` (after dictating in voice mode) | Cleans the transcript without changing its meaning; no audio or duration fields exist | Note in `notes/YYYY-MM-DD-<slug>.md` |
| `summarize`, `keypoints`, or a custom transformation | Loads the target sources fully, produces a grounded result, never written into the source | Citable insight in `insights/<type>/` |
| Consolidation, lint, repair | Detects near-duplicates, broken links, orphans; dedupes only with explicit confirmation | Clean hub; one-step zip, download, or git sync |

Chat versus Ask: both modes cite what they used. Chat mode loads the full
source or page (library read or file read) when you need depth. Ask mode runs
semantic search, preferably over consolidated pages instead of raw chunks, when
you need a quick pointer.

Dictations: a voice-mode transcript becomes one note, with STT artifacts
cleaned without changing meaning, an honest engine and device field, and no
fabricated audio metadata, because voice mode provides none.

Transformations: summary, keypoints, structure, style, or a named custom
type. The skill loads the sources fully, produces a result grounded in the
source only, and stores it as a citable, independently deletable insight that
names its sources. Insights never write back into the source.

## Design: hybrid model

The structure follows a curated, exact wiki-style layout: predictable section
order, an index, slugified filenames. That keeps a large notebook readable for
LLMs at a glance and every artifact citable, findable, and independently
deletable. The processing vocabulary follows NotebookLM: sources, curation,
transformations, insights. Sources are never modified; every page carries
provenance; every insight names its sources.

## Repository layout

```
agent-notebook/
├── SKILL.md                   # Skill entry point: conventions and workflows
└── references/
    ├── structure.md           # Notebook layout, schemas, filename conventions
    └── instructions.md        # Binding behavior rules
LICENSE                        # MIT
CONTRIBUTING.md
CODE_OF_CONDUCT.md
.github/                       # Issue and pull request templates
README.md
```

## State and cross-agent sync

Each notebook keeps a compact `state.md`: a current-standing block (where the
topic stands, open points, next actions) plus a rolling monthly log; older
months rotate into `state-archive/YYYY-MM.md`. State files are part of the sync,
so any agent reading your clone, for example a CLI agent, picks up exactly
where you stand by reading one small file per notebook, and the standing
block's last-updated date tells it how fresh that picture is.
State files are written by whichever agent is currently working on the
notebook: your platform agent during platform sessions, a CLI agent during
local ones, with your git remote carrying the changes in both directions.
The rule that keeps this safe: one writing agent at a time, and sync between
handoffs.

## Compatibility conventions

- Frontmatter keys, types, and tags in English, lowercase; kebab-case with the
  established snake_case exceptions (`stt_engine`, `created_time`).
- ISO 8601 dates; filenames for notes and insights carry a `YYYY-MM-DD-` date
  prefix equal to the `created` date. Frontmatter keeps `created` date-only
  (matching Obsidian's Date property) and the exact timestamp with timezone in
  `created_time`; edits refresh an optional `updated`.
- Note and insight content keeps the language of its source; multilingual
  titles via the `aliases` frontmatter field.
- Multiple sources in page or insight frontmatter are YAML lists.
- Plain Markdown files: directly usable in Obsidian and similar tools.

## Limitations

- No offline dictation capture; no audio archiving (only the transcript
  persists).
- Voice mode provides no audio file or duration metadata; those fields are
  never fabricated.
- Data stays on your agent platform while you work there; its data and
  training policies apply (on Mistral Pro plans, opt-out available).

## References

- [Vibe Work Voice mode](https://docs.mistral.ai/vibe/work/voice-mode)
- [OpenNotebook](https://github.com/lfnovo/open-notebook) (concept reference)
- [wiki_llm](https://github.com/LeoBR84p/wiki_llm) (pipeline origin)
- [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
