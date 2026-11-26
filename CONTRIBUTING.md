# Contributing to agent-notebook

Thanks for your interest in contributing. This project is a NotebookLM-style
notebook system built on plain Markdown and the SKILL.md format: no Docker,
no database, no lock-in.

## Ways to contribute

- Bug reports and documentation fixes
- Improvements to the skill instructions, schemas, or workflows
- Language coverage: trigger phrases are intent-based and work in any
  language; examples in additional languages are welcome

## Before you start

Open an issue first for anything larger than a typo fix, so we can align on
the approach before you invest time.

## Making changes

1. Fork the repository and create a feature branch
   (`feature/<short-kebab-case-name>`).
2. Keep changes focused: one topic per pull request.
3. Follow the repository conventions:
   - Markdown is the single source of truth; no binary formats.
   - Frontmatter keys, enumerated values, tags, and filenames are English,
     lowercase, kebab-case (snake_case exceptions: `stt_engine`,
     `created_time`).
   - Dates are ISO 8601; filenames for notes and insights use a
     `YYYY-MM-DD-` date prefix.
   - Never include personal data, credentials, or private notebook content
     in examples.
4. Commit messages: imperative subject line, short body explaining the why.

## Pull requests

- Describe the change and the rationale; reference the issue number.
- Note whether the local skill copy and the repository are both updated.
- Maintainers review and merge pull requests as time permits.

## License

By contributing, you agree that your contributions are licensed under the MIT
License.

