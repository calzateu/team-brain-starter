---
name: "Brain: Scan repo"
description: Generates or regenerates the generated zone of a code repo's page and emits the questions the code doesn't answer
category: Workflow
tags: [brain, wiki, repos, scan]
---

Scan a code repository and produce/update its page in `repos/`: the
**generated zone** (regenerable, cheap) plus the list of **questions the
code doesn't answer** (the input for the curated layer). This is a repo's
entry door into the brain.

**Config**: `brain.config.yml` → `repos` (team-shared identity: name,
remote, page) + `brain.config.local.yml` (per-machine clone paths,
git-ignored).

**Input**: `/brain-scan-repo <name|path> [module]`.
- Without `module`: **map level** — scan the repo overview only.
- With `module`: **module level** — deep-scan that one module.

**Two-level scanning for large repos** (map first, drill later):
- On the first scan, assess size: if the repo has many distinct modules
  (rule of thumb: >5, or the user says so), propose the folder layout
  `repos/<name>/` with `index.md` (the map) + one page per module created
  on demand. Small repos stay a single `repos/<name>.md`.
- **Map level never deep-reads modules**: it inventories them (one line
  each: name, purpose guess, size signal) and maps cross-module
  dependencies. Cheap and fast by design.
- Module pages are created only when explicitly requested
  (`/brain-scan-repo <name> <module>`), each with its own generated/curated
  zones. Suggest which 1-3 modules to scan first based on the session's
  needs — never scan all modules unprompted.

**Path resolution** (local paths differ per machine — never commit them):
1. `brain.config.local.yml` → `repos.<name>.path`.
2. Convention: `../<name>` relative to the vault (sibling clone).
3. Ask the user for the path; offer to save it into
   `brain.config.local.yml` (create the file from
   `brain.config.local.example.yml` if it doesn't exist).
If the repo name isn't in `brain.config.yml` at all, also offer to add its
identity there (name, remote via `git remote get-url origin`, page) — that
part IS committed. If the resolved clone's remote doesn't match the
configured `remote`, warn before scanning (wrong repo or fork).

**Steps**

1. **Repo reconnaissance**
   From the local path: read the README, manifests (package.json,
   pyproject, go.mod, etc.), directory structure (2 levels), CI, and the
   repo's own CLAUDE.md / AGENTS.md if present. Record the current commit
   SHA (`git rev-parse --short HEAD`). Don't read the whole repo: sample the
   main modules.

2. **Compose the generated zone**
   - Map level (`templates/repo.md`): purpose, module inventory,
     cross-module dependencies, entry points, build/test commands, external
     services, and the stamp "Scanned at `<sha>`, `<date>`".
   - Module level (`templates/repo-module.md`): the same treatment scoped to
     the module — its internal map, entry points, contracts with sibling
     modules — plus a link back to the repo map.
   Honesty rule: if something can't be determined from the code, write
   "not determinable from the code" — do NOT fill it with a guess.

3. **Compose the unanswered-questions list**
   The most valuable output of the scan: concrete questions the code doesn't
   answer and only the team can. E.g.: "why are there two different HTTP
   clients?", "is module X deprecated or in use?", "which customer motivated
   flag Y?". They go into the "Open questions" section of the curated zone,
   as candidates for `decisions/` or `concepts/`.

4. **Approval gate** (same protocol as /brain-close):
   ```
   **Scan plan — <repo>[/<module>] @ <sha>**

   R1. repos/<repo>.md (or repos/<repo>/index.md, or repos/<repo>/<module>.md)
       — generated zone new/regenerated (summarized diff if it existed)
   R2. Open questions (N): numbered list
   R3. [optional] CREATE [decision-pending] issues per question → tracker
   R4. [map level, large repo] suggested next module scans (1-3, with why)
   R*. index.md + log.md

   go / drop Rn / stop
   ```
   If the page already existed: the generated zone is **fully replaced**
   between the `<!-- generated:start/end -->` markers; the **curated zone is
   never touched** by this command. If you detect that the curated zone
   contradicts the scan (e.g. it mentions a module that no longer exists),
   report it as a finding — don't edit it.

5. **Write and record** (approved items only): page with `generated: true`
   and `source: /brain-scan-repo @ <sha>`, line in `index.md`, `scan |`
   entry in `log.md`, commit `scan: <repo> @ <sha>` + push.

**Re-scan**: same command, same level — map re-scans touch only the map;
module re-scans touch only that module. `lint` suggests re-scanning when a
generated zone falls behind the real repo, and reports modules listed in the
map that have no page yet (coverage checklist).

**Output**
Recap: page created/regenerated, summary of the generated-zone diff, the
open questions (numbered, ready to discuss as a team), and issues created if
R3 was approved.
