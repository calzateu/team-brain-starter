# sources/ — Layer 1 (raw, immutable)

The LLM reads these sources but **never edits them**. If a source changes at
its origin, re-ingest it as a new snapshot with a new date.

- Naming: `YYYY-MM-DD_<type>_<slug>.md`
- `notion/` — snapshots of subscribed Notion pages (see SUBSCRIPTIONS.yml)
- `assets/` — images, PDFs, binaries
- `interviews/`, `transcripts/`, `articles/` — created when their first input arrives

Ingest flow: raw material here first → summary in Layer 2 → update index.md,
log.md, and related pages.
