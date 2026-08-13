# Team Brain — the team's shared brain

Shared memory for humans and agents: decisions, business context, knowledge
about the code repos, and known landmines. A git repo of markdown; agents
(Claude Code) read and maintain it; the team governs it via `CLAUDE.md` and
PRs.

**Design lineage**: Karpathy's LLM Wiki pattern (3 layers: raw sources →
derived wiki → schema) + battle-tested vault conventions (index/log,
ADR-lite, sessions, approval gate, wiki↔tracker bridge, git sync) + ideas
from gbrain (memory hygiene, shared rule files, per-person folders,
staleness) + our own deltas (repos/ with generated/curated zones, landmines/,
anti-fabrication rule, PR rule, curated Notion source).

**New team member?** Read `HOW-TO-USE.md` — one page, everything you need.
(Admin: before sharing it, replace `<BRAIN-REPO-GIT-URL>` there with this
repo's real git URL so the setup prompt works as a pure copy-paste.)

## Getting started (30-60 min)

1. **Create the repo**: copy this directory, `git init`, add the team
   remote, push. Everyone clones it (or symlinks it) next to their code
   repos.
2. **Configure**: edit `brain.config.yml` (tracker, team, areas, and the
   repo identities: name + remote + page). Clone paths are per-machine: if
   your code repos aren't siblings of the vault (`../<name>`), copy
   `brain.config.local.example.yml` to `brain.config.local.yml` (git-ignored)
   and set your paths there — or just let the skills ask and save it for you.
3. **Install the skills**: copy `skills/*.md` to wherever your Claude Code
   commands live (e.g. `.claude/commands/`). You get `/brain-open`,
   `/brain-close`, `/brain-ingest-notion`, `/brain-scan-repo`,
   `/brain-lint`.
4. **Initial scan**: run `/brain-scan-repo` on 2-3 representative repos. The
   "open questions" it emits → team session → first `decisions/` and
   `concepts/` pages.
5. **Subscribe Notion**: add the PRD and key docs to
   `sources/notion/SUBSCRIPTIONS.yml` (respecting the sensitivity rule in
   `CLAUDE.md` §8) and run `/brain-ingest-notion`.
6. **Person-by-person onboarding** (don't skip it): before giving anyone
   free access, pre-populate what concerns them (their area in the index,
   1-2 pages they'll recognize) and walk them through 2-3 flows: a
   `/brain-open` with a topic of theirs, a query that synthesizes with
   citations, and a `/brain-close` of a real session. Without that "this
   thing knows me" moment, adoption doesn't happen.

## The ritual that keeps the system alive

- `/brain-open <topic>` when starting → context + landmines + pendings.
- Work as usual.
- `/brain-close <topic>` when finishing → approval gate → wiki + tracker + push.

Plus the hygiene beat: `/brain-lint` weekly (rotate who runs it) —
staleness, contradictions, orphans, coverage. Mechanical fixes via the gate;
contradictions become decision-pending issues for a human.

If open/close gets used, the brain maintains itself. If not, it dies in
weeks — the risk of this system isn't technical, it's habit.

## Map

| What                       | Where                                        |
| -------------------------- | -------------------------------------------- |
| How to use it (1 page)     | `HOW-TO-USE.md` (everyone reads this)        |
| System rules               | `CLAUDE.md` (read first)                     |
| Team config                | `brain.config.yml`                           |
| Catalog / history          | `index.md` / `log.md`                        |
| Immutable raw sources      | `sources/` (Notion in `sources/notion/`)     |
| Derived knowledge          | `product/ architecture/ concepts/ entities/` |
| Decisions (ADR-lite)       | `decisions/`                                 |
| Code repos                 | `repos/` (generated + curated zones)         |
| Landmines                  | `landmines/`                                 |
| Session close-outs         | `sessions/`                                  |
| Templates                  | `templates/`                                 |
| Commands                   | `skills/`                                    |
