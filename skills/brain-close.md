---
name: "Brain: Close session"
description: Closes a session — archives the durable into the wiki and pushes the actionable to the tracker
category: Workflow
tags: [brain, wiki, session]
---

Close a work session: archive what's durable into the wiki and push what's
actionable to the tracker, in one step. This is the wiki ↔ tracker bridge.

**Config**: read `brain.config.yml` (tracker, labels, areas, product
decisions doc). Everything team-specific comes from there.

**Input**: whatever follows `/brain-close` is the `topic` (optional).
Without one, **derive a short kebab-case topic from the session's content**
(e.g. what dominated the conversation: `payments-refactor`,
`q3-pricing-decision`) and show it in the gate plan — the user can adjust
it there (`edit W1: topic=<x>`) like any other item.

**Before anything — scope guardrail**: only for work belonging to the team
that owns this vault. If the session wasn't about that, write nothing; say
so and stop.

**Steps**

1. **Gather the session material**
   Review the conversation and identify:
   - **Summary** of what was done / discussed.
   - **Decisions made** (closed) and **pending**.
   - **New tasks** and **blockers** discovered.
   - **Landmines discovered** (something broke, deceived, or cost dearly →
     candidate for `landmines/`).
   - `concepts/`, `entities/`, `product/`, `repos/` (curated zone) pages
     that changed.
   If something is ambiguous (decision made or pending? landmine or minor
   detail?), ask with **AskUserQuestion** before writing.

2. **Session identity**
   - `persona`: `git -C <vault> config user.name` (or ask).
   - `device`: hostname. `date`: today (YYYY-MM-DD).

3. **Approval gate — preview the plan, wait for the `go`**
   **CRITICAL:** write nothing to the vault, call no tracker MCP, and do no
   git add/commit/push until the user approves at this step. Skipping this
   gate breaks the command.

   Present a change plan with **per-line IDs**, grouped by destination:

   ```
   **Change plan — review before continuing**

   Wiki:
     W1. sessions/<date>_<person>_<topic>.md — new session page
     W2. decisions/<date>_<slug>.md — "<decision title>" (area: <x>)
     W3. landmines/<slug>.md — "⚠️ <title>" (area: <x>)
     W4. [PR] concepts/<page>.md — substantial edit → goes via branch+PR, not direct
     W*. index.md + log.md — updated automatically with what's approved

   Tracker:
     L1. CREATE [<decision_pending label>] "<title>" → team: <team> · owner: <name>
     L2. CREATE [<task label>] "<title>" → team: <team> · owner: <name>
     L3. CLOSE <ID-NN> + comment linking to W2
     L*. Product decisions doc — append of approved product/cross decisions
         (only if tracker.product_decisions_doc is configured)

   How should I proceed?
   - "go" → execute the WHOLE plan.
   - "drop W2 L3" → drop those items, execute the rest.
   - "edit L1: <new title>" → adjust and re-confirm.
   - "stop" → do nothing.
   ```

   Gate rules:
   - After drops/edits, **re-print the revised plan** with the same IDs and
     wait for confirmation again. Iterate until `go` or `stop`.
   - If a decision Wn linked to an Lk is dropped, warn that Lk loses its
     backlink and ask whether to drop it too.
   - `W*` cannot be dropped — it always reflects only what was approved.
   - Ambiguous reply → one **AskUserQuestion**, then proceed.

4. **Write the session page** (template `templates/session.md`), with
   approved items only.

5. **Update the wiki** (approved items only)
   - Decisions → `templates/decision.md`. For fe/be/infra, **cite the
     commit/PR** in the code repo as evidence. Link its issue.
   - Landmines → `templates/landmine.md`, linked from the affected repo's
     page (curated zone).
   - **PR rule (CLAUDE.md §9.4)**: substantial edits to shared pages
     (`concepts/`, `entities/`, `product/`, `architecture/`, `CLAUDE.md`) go
     on a branch `session/<date>-<topic>` and open a PR; the plan marks them
     `[PR]`. New session-owned pages (sessions/, decisions/, landmines/) and
     mechanical updates (index, log, links) go direct.
   - Update `index.md` (one line per new page) and `log.md` (prefixes
     `session |`, `decision |`) — only for what was actually written.

6. **Push the actionable to the tracker (MCP)** (approved items only)
   **Team routing**: each item's team = `tracker.teams.by_area[<item area>]`
   if configured, else `tracker.teams.default`. The plan always shows the
   resolved team per L-item; the user can override with
   `edit Ln: team=<name>`. If an item has no area and routing is configured,
   ask with **AskUserQuestion**.
   As per the plan: create tasks/decision-pending/blocked with their config
   labels, close issues for decisions made with a link to the page, and put
   the issue ID back on the page (bidirectional traceability). If
   `tracker.product_decisions_doc` is configured, append approved
   product/cross decisions to that doc (newest row on top). If the MCP isn't
   connected, list what would have been created. Never invent IDs.

7. **Commit + push the vault**
   ```bash
   git -C <vault> add -A
   git -C <vault> commit -m "session: <person> · <topic> (<date>)"
   git -C <vault> push
   ```
   If there were `[PR]` items: push the branch and open the PR (or leave the
   command ready). If everything was dropped, don't commit.

**Output**
Concise recap: pages actually created, issues created/closed, PRs opened,
items dropped/edited at the gate, pending decisions that remain, and push
confirmation.
