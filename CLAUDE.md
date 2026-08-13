# CLAUDE.md — Team Brain Schema

> **Layer 3.** This file governs how this vault is maintained. The team edits
> it (via PR); the LLM never edits it on its own initiative. If a session
> instruction contradicts this schema, the schema wins; if the schema is
> wrong, change the schema via PR.

This vault is the **team's shared memory**: decisions, business context,
knowledge about the code repositories, and known landmines. Humans and agents
(Claude Code) read and write the same asset.

---

## 1. Three-layer model

1. **Layer 1 — `sources/`**: raw, **immutable** sources. The LLM reads, never
   edits. If a source changes at its origin, re-ingest it as a new snapshot.
   Naming: `YYYY-MM-DD_<type>_<slug>.md`.
2. **Layer 2 — the wiki** (`index.md`, `log.md`, `product/`, `architecture/`,
   `concepts/`, `entities/`, `decisions/`, `sessions/`, `repos/`,
   `landmines/`): derived knowledge the LLM creates and maintains. Every page
   cites its source via frontmatter or a Source section.
3. **Layer 3 — this file**: the schema. See the note above.

## 2. Directories and types

| Directory       | `type`         | What it holds                                                     |
| --------------- | -------------- | ----------------------------------------------------------------- |
| `sources/`      | `source`       | Immutable raw material: Notion snapshots, transcripts, articles, PDFs |
| `product/`      | `product`      | Dense summaries derived from product docs (PRD, ICP, scope)       |
| `architecture/` | `architecture` | How the system is built, cross-repo (living reference)            |
| `concepts/`     | `concept`      | Domain: mental models, business terms, frameworks                 |
| `entities/`     | `entity`       | ICPs, stakeholders, integrations, customers                       |
| `decisions/`    | `decision`     | EVERY decision made. ADR-lite: `YYYY-MM-DD_<slug>.md`             |
| `sessions/`     | `session`      | Session close-outs: `YYYY-MM-DD_<person>_<topic>.md`              |
| `repos/`        | `repo`         | One page per code repository (see §5)                             |
| `landmines/`    | `landmine`     | Negative knowledge: what breaks, deceives, or costs dearly (§6)   |

**Division-of-labor rule against neighboring tools** (don't invent a fourth
place):

- **Actionable / pending** → the tracker (Linear or other; see `brain.config.yml`).
- **Durable and queryable** → this wiki.
- **Large in-flight specs** → the team's spec tool.
- **Canonical product docs** → Notion (they enter here only as snapshots, §8).

## 3. Frontmatter for every page

```yaml
---
title: <readable title>
type: product | architecture | concept | entity | decision | session | repo | landmine | source
status: active | superseded | draft
updated: YYYY-MM-DD
source: <where it came from: path in sources/, session, PR, URL>
# Only in decisions/ and landmines/:
area: fe | be | product | cross | infra
# Only on pages produced by scanning (repos/ and derivatives):
generated: true | false
# Only on decisions that replace another:
supersedes: <path of the replaced page>
---
```

When a page becomes obsolete: `status: superseded` + link to its replacement.
**Knowledge is never deleted; it is marked as superseded.**

## 4. Conventions

- **kebab-case** names. Decisions and sources carry a date prefix.
- **Liberal wikilinks** (`[[page]]`). A link to a nonexistent page is valid:
  it marks something to be written.
- `index.md`: catalog, 1 line + one-sentence summary per page, grouped by
  category. Updated on every ingest/session.
- `log.md`: **append-only**, chronological, with consistent prefixes:
  `ingest |`, `decision |`, `session |`, `lint |`, `scan |`, `notion |`.
- Language: English throughout the vault.

## 5. `repos/` — the generated layer for code

**Small/medium repo** → a single page `repos/<name>.md`.
**Large repo (multiple modules)** → a folder `repos/<name>/` with:
- `index.md` — the **map**: purpose, module inventory (one line each),
  cross-module dependencies, commands. Always produced first.
- one page per module, `repos/<name>/<module>.md`, created **on demand** —
  the map is scanned first; modules are drilled into selectively, starting
  with the ones that matter. `lint` reports modules still unscanned.

Every repo page (single, map, or module) has two clearly separated zones:

- **Generated zone** (`generated: true`, between the markers
  `<!-- generated:start -->` and `<!-- generated:end -->`): purpose, module
  map, entry points, build/test commands, dependencies on other repos.
  Produced by `/brain-scan-repo` (map level or module level) and **fully
  regenerated** on every re-scan of that level —
  do not hand-edit; manual changes there are lost.
- **Curated zone** (outside the markers): context the code doesn't state —
  why the repo exists, decisions that shaped it (links to `decisions/`),
  associated landmines (links to `landmines/`), who knows it best.

The generated zone is disposable and cheap; the curated zone is the asset. If
they conflict, the curated zone wins and the divergence must be investigated.

## 6. `landmines/` — negative knowledge

Things that break, deceive, or cost dearly: "this innocent refactor takes
down production", "this API documents X but does Y", "don't upgrade lib Z
without reading this". One page per landmine, with: what happens, how it was
discovered (date, session, or incident), how to avoid/detect it, and links to
the related repo and decision. When a landmine is defused (root cause fixed),
mark it `status: superseded` explaining what resolved it — the landmine
history teaches too.

## 7. Operations

- **Ingest**: raw material into `sources/` first → summary/update in Layer 2
  → update `index.md`, `log.md`, and related pages. One source may touch
  5-15 pages; that's normal and desirable.
- **Query**: answer with citations to wiki pages. If the exploration produced
  a valuable synthesis, file it as a new page (good answers compound just
  like ingested sources).
- **Lint** (`/brain-lint`, periodic — weekly or after heavy ingests):
  contradictions between pages, claims superseded by newer sources, orphan
  pages, missing provenance, stale Notion snapshots (§8), `repos/` generated
  zones behind the real repo, index/log drift, unscanned modules. Mechanical
  issues get fixed via the gate; contradictions are **flagged as
  decision-pending issues, never auto-resolved**.
- **Scan** (`/brain-scan-repo`): generates/regenerates the generated zone of
  a `repos/` page and emits the list of "questions the code doesn't answer"
  as candidates for `decisions/` or `concepts/`.

## 8. Notion source (business → brain)

- **Notion is canonical** for product docs. Only **dated, immutable
  snapshots** live here, in `sources/notion/`, named `YYYY-MM-DD_<slug>.md`,
  with frontmatter that includes `notion_url` and `notion_last_edited` (the
  origin's `last_edited_time`).
- Only pages listed in `sources/notion/SUBSCRIPTIONS.yml` are synced
  (curated list; adding a doc is a deliberate human act).
- **Sensitivity rule**: only product/business pages suitable for everyone
  with access to this repo. Nothing with identifiable customer data,
  personal information, HR, legal, or financial matters. When in doubt,
  **do not sync** — ask the team.
- The value lives in Layer 2: every new snapshot must regenerate its summary
  in `product/` and touch the affected `concepts/`/`entities/`. The raw
  snapshot is backup and evidence, not daily reading material.
- **Staleness**: `lint` and `/brain-ingest-notion` compare the stored
  `notion_last_edited` against the current one in Notion; if they diverge,
  the snapshot (and its derivatives) are stale and this gets reported.

## 9. Integrity rules (non-negotiable)

1. **Anti-fabrication**: if the wiki has no relevant pages for a question,
   the answer is "the wiki has no confident answer on this" — never
   synthesize from low-relevance material as if it were team knowledge. And
   such an answer is **never filed back** into the wiki.
2. **Scope**: before writing anything to the vault, verify the content
   belongs to the team's work. If not, don't write it even if asked.
3. **Provenance**: every page states where it came from (`source:` or a
   Source section). A claim without provenance is a candidate for deletion
   in the next lint.
4. **Collisions**: `sessions/` doesn't collide (person in the filename). For
   shared pages (`concepts/`, `entities/`, `product/`, `architecture/`, this
   file), substantial changes go through a **PR**, not a direct push to
   main. Mechanical updates (index, log, links) may go direct within the
   `/brain-close` flow.
5. **Approval gate**: no skill writes to the vault, calls the tracker, or
   does git push without presenting the change plan and receiving the `go`
   (see `/brain-close`).
6. **Sync via git**: pull when opening a session, commit + push when closing,
   authored by the session's person. The vault is its own git repo.

## 10. Context budget

This file must stay **short** (< ~300 lines). Knowledge lives in the pages
and is loaded on demand via `index.md`; this schema only defines rules. If a
section grows too large, extract it to a `concepts/` page and leave the rule
plus a link here.
