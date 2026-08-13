---
name: "Brain: Ingest Notion"
description: Syncs subscribed Notion docs (PRDs, product context) into the brain as snapshots + summaries
category: Workflow
tags: [brain, wiki, notion, ingest]
---

Bring the product docs that live in Notion into the brain: detect which ones
changed, snapshot them to `sources/notion/`, and regenerate their Layer 2
summaries. This is the **business → brain** lane.

**Config**: `brain.config.yml` → `notion.subscriptions_file` and
`notion.snapshots_dir`.

**Input**: optionally one or more `slug`s after `/brain-ingest-notion` to
limit to those docs. Without input, review all subscriptions.

**Requirement**: the Notion MCP connected. If it isn't, say so, explain how
to connect it, and stop. Never invent content.

**Steps**

1. **Read the subscription list** (`SUBSCRIPTIONS.yml`). If empty, explain
   how to subscribe a doc (add an entry with slug, notion_url, summary_page)
   and recall the **sensitivity rule** (CLAUDE.md §8): only product/business
   content suitable for the whole team — no PII, customer data, HR, legal,
   or financial material. When in doubt, don't subscribe.

2. **Detect changes**
   For each subscription (or the requested slugs):
   - Query the page's current `last_edited_time` via MCP.
   - Find its most recent snapshot in `sources/notion/` (by `slug` in
     frontmatter) and compare against its `notion_last_edited`.
   - Classify: **new** (no snapshot), **changed**, **up to date**.
   If everything is up to date, report it and finish.

3. **Approval gate — same as /brain-close**
   Present the plan and wait for the `go`:
   ```
   **Ingest plan — review before continuing**

   N1. [new]     prd → snapshot sources/notion/<date>_prd.md
                 → regenerate product/prd-summary.md
                 → touch: [[concepts/...]] (estimated after reading)
   N2. [changed] icp → new snapshot <date>_icp.md
                 → regenerate product/icp-summary.md
   N*. index.md + log.md — automatic with what's approved

   go / drop Nn / stop
   ```
   Gate rules: identical to /brain-close (re-print after drops, N* not
   droppable separately, AskUserQuestion on ambiguity).

4. **Snapshot (Layer 1)** — for each approved doc:
   - Fetch the full content via MCP and **normalize it to clean markdown**:
     nested blocks flattened into headings, embedded databases as markdown
     tables if small or `[database omitted: <name>]` otherwise, Notion
     comments excluded.
   - **Sensitivity scan before writing**: if the content contains PII,
     identifiable customer data, or HR/legal/financial material, do NOT
     write the snapshot; report the finding (category, not the data) and
     suggest reviewing the subscription.
   - Write `sources/notion/YYYY-MM-DD_<slug>.md` using the
     `templates/notion-snapshot.md` template (include the exact `notion_url`
     and `notion_last_edited`). **Never edit previous snapshots**; mark them
     `status: superseded`.

5. **Derive Layer 2** — where the value lives:
   - Regenerate the subscription's `summary_page`: a dense summary (1-2
     pages max) of what the team and the agents need to know, citing the
     snapshot. If the summary_page already existed, highlight **what
     changed** vs. the previous version (a "Changes in this ingest"
     section).
   - Update the `concepts/` and `entities/` affected by the changes (this
     may touch several pages; that's normal). Substantial changes to shared
     pages follow the PR rule of CLAUDE.md §9.4.
   - If a doc change **contradicts an active decision** in `decisions/`, do
     NOT mark it superseded on your own: report it as a conflict and suggest
     creating a `decision-pending` issue in the tracker.

6. **Bookkeeping + commit**
   - `index.md`: line per new page. `log.md`: `notion |` entries.
   - Commit: `git add -A && git commit -m "notion: ingest <slugs> (<date>)" && git push`.

**Output**
Recap: snapshots created, summaries regenerated (with the key conceptual
diff in 1-2 lines each), Layer 2 pages touched, conflicts with decisions
detected, and docs that were already up to date.
