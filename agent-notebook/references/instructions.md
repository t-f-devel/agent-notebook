# Instructions — Agent Notebook

Binding behavior rules for working in agent notebooks (hybrid model).

## Language

- Frontmatter keys, enumerated values, tags, and filenames: English, lowercase.
  Tags and filenames are kebab-case; frontmatter keys are kebab-case with the
  established snake_case exceptions (`stt_engine`, `created_time`).
- Note and insight content: language of its source (German or English or other).
  Titles may be in any language; use the `aliases` frontmatter field for
  multilingual titles (Obsidian-compatible), e.g.
  `aliases: ["Idea: minor characters", "Idee: Nebendarsteller"]`.
- Dates: ISO 8601; filenames with `YYYY-MM-DD-` date prefix. In note and insight
  frontmatter, `created` is date-only (`YYYY-MM-DD`, so Obsidian's Date property
  type accepts it); the exact moment with timezone lives in `created_time`
  (ISO 8601 date-time).

## Citation and source fidelity

- Every curated page (pages/) carries a References line with provenance
  (filename, doclib URI, or URL).
- Every insight references its source(s) in the frontmatter (`source`) and, where
  useful, points to specific passages in the text.
- Statements without source coverage are marked as such or omitted; never
  fabricate content.
- Sources (sources/ and library documents) are never modified.

## Target notebook resolution

- Every note, curated page, and insight is saved to exactly one target
  notebook, resolved explicitly before saving: match the content and its
  project or user context against the one-line descriptions in the hub
  KNOWLEDGE.md.
- Exactly one clear match: save there and name the chosen notebook in the
  confirmation.
- Multiple plausible matches: list the candidates and ask the user to choose;
  never guess.
- No match: propose creating a new notebook; create only after explicit user
  confirmation. Never create a notebook silently.
- An explicit user instruction (named notebook or project context) overrides
  the routing proposal.

## Response formats

- Keep chat answers short; the durable artifact is the file (note, page,
  insight), not the chat output.
- After every create or change: update `index.md` and update `state.md` per the
  state schema in structure.md — rewrite the standing block (Stand, open
  points, next actions, last updated) in place and append one log row (date,
  action, result). Never repeat open points per log row.
- Routine audits append one short log row only (date, consistency result, new
  findings); unchanged findings are implied by omission and live in the audit
  note.
- Rotation on month change: move the previous month's log to
  `state-archive/<YYYY-MM>.md`; archives are write-once.
- Editing an existing note, page, or insight sets or refreshes its optional
  `updated: <YYYY-MM-DD>` frontmatter date.
- Confirmation to the user: filename, title, type/tags, storage location;
  uncertainties are named, never guessed.

## Dictation conventions

- Trigger: dictation transcript plus a request to save it.
- Clean STT artifacts without changing meaning or content.
- Schema per references/structure.md: `stt_engine: <stt engine, e.g. vibe-voice-mode>`,
  `source: dictation`; no fabricated duration or audio fields.
- One note per dictation; never merge dictations without explicit instruction;
  stored in notes/.

## Transformation conventions

- Insight types: summary, keypoints, structure, style, custom-<name>.
- Load the source fully (library_read), never transform from snippets.
- Store in insights/<type>/; citable and independently deletable; never write
  into the source itself.

## Mode separation (Chat vs. Ask)

- Full source context: library_read or read the full page (Chat mode).
- Semantic search: library_search, preferably over consolidated pages instead
  of raw chunks (Ask mode).
- Both modes cite the source they used.

