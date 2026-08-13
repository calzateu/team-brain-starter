---
name: "Brain: Open session"
description: Loads the team brain context (wiki + tracker pendings) when starting a session
category: Workflow
tags: [brain, wiki, session]
---

Open a work session: sync the vault, load the relevant context, and show
what's pending, **before** starting to work.

**Config**: read `brain.config.yml` at the vault root (name, tracker, areas,
Notion subscriptions). Everything team-specific comes from there, not from
this skill.

**Input**: whatever follows `/brain-open` is the session `topic` (optional).
Without a topic, load general context.

**Before anything — scope guardrail**: this command is only for work
belonging to the team that owns this vault. If the conversation is
unrelated, say so and do nothing.

**Steps**

1. **Sync the vault**
   ```bash
   git -C <vault.path> pull --rebase
   ```
   If the vault isn't a git repo yet, say so and continue (first run).
   Remind the user to `git pull` any code repos they'll touch as well.

2. **Load wiki context**
   - Read `index.md`.
   - Based on the `topic`, read the relevant pages the index suggests
     (concepts/, product/, repos/ for the area). Without a topic: `index.md`
     plus the `product/` summaries marked as primary.
   - Review the latest `log.md` entries and recent `sessions/` to know where
     the team left off.
   - If the topic touches a code repo, read its `repos/` page and the
     `landmines/` linked from it. **Landmines are always surfaced** in the
     summary when they apply to the topic.

3. **Notion freshness check (lightweight, non-blocking)**
   - Read `sources/notion/SUBSCRIPTIONS.yml`. If there are subscriptions and
     the Notion MCP is connected, compare each subscribed page's current
     `last_edited_time` against the `notion_last_edited` of its latest
     snapshot.
   - If docs are stale, report it in one line:
     "📄 N Notion docs changed since their last snapshot — run
     `/brain-ingest-notion` to update." **Do not ingest here**; this command
     is read-only.
   - If the MCP isn't connected, skip the check silently.

4. **Query the tracker (MCP)**
   - If `tracker.provider` is `linear` and the MCP is connected, fetch: my
     open issues, issues labeled `decision_pending`, issues labeled
     `blocked` (labels per config) — across **all teams in
     `tracker.teams`** (default + by_area values), grouped by team in the
     summary when more than one has results.
   - If not connected or `provider: none`, say so and continue. Never invent
     issues.

5. **Summarize "where we are"**, concisely:
   - 2-4 wiki context bullets relevant to the topic.
   - ⚠️ Applicable landmines (if any).
   - Pending decisions / active blockers / my open tasks (from the tracker).
   - The latest from `log.md` and the most recent session.
   - Notion freshness (if applicable).

**Output**
A brief, actionable summary of where we are and what's pending, ready to
start the session. Do not edit files in this command (read-only, except the
`git pull`).
