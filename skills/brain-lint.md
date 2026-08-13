---
name: "Brain: Lint"
description: Runs the brain's hygiene pass — staleness, contradictions, provenance, orphans, coverage — and fixes what's mechanical via the gate
category: Workflow
tags: [brain, wiki, lint, hygiene]
---

Run the hygiene pass over the vault: detect what's stale, contradictory,
orphaned, unsourced, or uncovered, report it by severity, fix the mechanical
part (with approval), and turn the judgment calls into tracker issues. This
is the brain's "dream cycle", human-triggered.

**Config**: `brain.config.yml` (tracker for filing conflicts, Notion
subscriptions, repos).

**Input**: optional scope after `/brain-lint`: `notion` | `repos` | `links` |
`index` | `all` (default `all`).

**Cadence**: weekly, or after a heavy ingest/scan. Anyone can run it.

**Principle — fix vs. flag**: this skill FIXES only mechanical issues
(index/log sync, marking clearly superseded snapshots, link repairs it can
prove). Anything requiring judgment — especially contradictions — is
FLAGGED, never auto-resolved. A lint that silently "fixes" a contradiction
is corrupting memory, not cleaning it.

**Steps**

1. **Load the ground truth**: `CLAUDE.md` (the rules), `index.md`, `log.md`,
   `brain.config.yml`, and the file tree.

2. **Run the checks** (per scope):

   **Staleness**
   - Notion: for each subscription, compare current `last_edited_time` (via
     MCP, if connected) against the latest snapshot's `notion_last_edited`.
   - Repos: for each `repos/` page with a generated zone, compare its
     scanned SHA against the repo's current HEAD (when the local clone is
     resolvable); report how far behind.
   - Derived pages: Layer 2 pages whose `source:` points to a snapshot that
     is now `superseded` → the summary may describe an old reality.

   **Contradictions** (flag, never fix)
   - Pages making conflicting claims about the same thing.
   - Active `decisions/` contradicted by a newer source or a newer decision
     that doesn't declare `supersedes:`.

   **Provenance**
   - Pages without `source:` frontmatter or a Source section.
   - Decisions (fe/be/infra) without a commit/PR citation.

   **Graph health**
   - Orphans: pages with no inbound wikilinks and no index entry.
   - Broken wikilinks: report as a "most-wanted pages" list (a broken link
     is valid — it marks something to write — but the aggregate shows what
     the team keeps reaching for and nobody has written).
   - Supersede chains: `superseded` pages still linked from active pages as
     if current.

   **Index/log integrity**
   - Pages missing from `index.md`; index entries pointing at missing pages;
     index one-liners that no longer match the page's content.

   **Coverage**
   - Large-repo maps whose module wikilinks have no page yet (unscanned
     modules checklist).

3. **Report — grouped by severity, with IDs**
   ```
   **Lint report — <date> (scope: <scope>)**

   🔴 Contradictions (judgment needed)
     C1. decisions/2026-05-10_x.md vs sources/notion/2026-08-01_prd.md — <one line>

   🟡 Stale
     S1. product/prd-summary.md derives from a superseded snapshot (PRD edited 2026-08-03)
     S2. repos/api-core/index.md scanned @ abc123, repo now 47 commits ahead

   🟠 Provenance
     P1. concepts/billing-model.md has no source

   ⚪ Graph / index / coverage
     G1. entities/old-partner.md orphaned (no inbound links, superseded 3 months ago)
     G2. Most-wanted missing pages: [[concepts/rate-limiting]] (linked from 4 pages)
     G3. repos/api-core: modules `auth`, `webhooks` unscanned
   ```

4. **Approval gate — plan of actions** (same protocol as /brain-close)
   Propose per finding, honoring fix-vs-flag:
   ```
   **Lint action plan**

   Mechanical (this skill fixes, with your go):
     F1. index.md — add missing entries / remove dead ones (from G*)
     F2. mark sources/notion/2026-06-01_prd.md status: superseded (newer snapshot exists)

   Suggested runs (I won't run them from here):
     R1. /brain-ingest-notion prd            (fixes S1)
     R2. /brain-scan-repo api-core           (fixes S2)

   Flags → tracker:
     T1. CREATE [decision-pending] "Resolve: <C1 summary>" → team per area routing

   Needs a human author (can't be fixed mechanically):
     H1. P1 — someone who knows billing-model's origin should add its source

   go / drop Xn / stop
   ```

5. **Execute approved items only**, then bookkeeping: `lint |` entry in
   `log.md` with counts (found/fixed/flagged), commit
   `lint: <scope> (<date>)` + push. If the tracker MCP isn't connected,
   list the flags instead of filing them. Never invent IDs.

**Output**
The report, what was actually fixed, issues filed, suggested follow-up runs,
and the items left for humans — short enough to read, complete enough to
act on.
