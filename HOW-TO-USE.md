# How to use the Team Brain (1 page)

The brain is our shared memory: decisions, product context, repo knowledge,
and known landmines. You talk to it through Claude Code. **You never need to
edit files by hand** — you ask, review, and approve.

## First time? (once, ~5 min — Claude does the setup for you)

Open Claude Code in your working directory (where your code repos live) and
paste this — replace the URL if your admin hasn't already:

> *"Set up our team brain for me. Repo: `<BRAIN-REPO-GIT-URL>`.*
> *1) Clone it here as a sibling of my code repos (skip if already cloned).*
> *2) Install its `skills/*.md` as my Claude Code commands — pick the right
> location (project or user level), and tell me exactly where you put them
> and why.*
> *3) Read its `CLAUDE.md` and `HOW-TO-USE.md`.*
> *4) Then start my first session following `skills/brain-open.md` with
> topic `onboarding`, and give me the lay of the land."*

Claude will do all of it and report what it did and where. If one of your
code repos lives somewhere unusual, the skills will ask for its path once
and remember it (`brain.config.local.yml` — yours, git-ignored). The team
config (`brain.config.yml`) is already set up — admin only.

From then on, day to day is just:

> *"/brain-open payments-refactor"* — or in plain words:
> *"Open a brain session, I'm working on the payments refactor."*

## The ritual (this is 90% of it)

```
/brain-open [topic]     ← when you START working
      ... work as usual ...
/brain-close [topic]    ← when you FINISH
```

The topic is optional in both: a bare `/brain-open` gives you the general
state of things; a bare `/brain-close` figures out the topic from your
session and files everything accordingly (you'll see it in the plan before
approving).

- **open** shows you: relevant context, ⚠️ landmines, pending decisions,
  your open tasks. Read-only — it changes nothing.
- **close** finds the decisions, tasks, and discoveries in your session and
  proposes saving them. Nothing is written until you say **go**.

## The approval gate (how every write works)

Before writing anything, Claude shows a numbered plan:

```
W1. new decision page ...        L1. create Linear issue ... → team: X
go        → do everything        drop W1   → skip that item
edit L1: <change>  → adjust      stop      → do nothing
```

You can answer in plain language too ("yes but skip the issue"). **Nothing
is ever saved without your approval.**

## The other three commands

| Command | What it does | Plain-language equivalent |
| --- | --- | --- |
| `/brain-ingest-notion` | Pulls changed PRDs/docs from Notion, updates summaries | "Sync the PRD from Notion" |
| `/brain-scan-repo <name>` | Maps a code repo (big repos: map first, modules on demand) | "Add the payments repo to the brain" |
| `/brain-lint` | Weekly hygiene: finds stale/contradictory/orphaned pages | "Check the brain's health" |

Slash commands and plain sentences both work — commands guarantee the full
protocol, sentences are fine day to day.

## Rules everyone should know

1. **Durable → brain. Actionable → Linear. Product docs → Notion.** Don't
   invent a fourth place.
2. **If it cost you an hour, it's worth 3 lines.** Decisions, dead ends, and
   landmines go in at close — that's the whole point.
3. The brain says **"no confident answer"** when it doesn't know. That's a
   feature, not a failure — it means you can trust what it *does* say.
4. Notion stays the source of truth for product docs; the brain keeps
   summaries + snapshots. Never paste sensitive data (customers' personal
   info, HR, legal, financial) into it.
5. New here? Paste the setup prompt above, then ask someone to watch your
   first open → work → close cycle. Ten minutes, and you'll get it.

**For PMs and business folks**: same ritual, no code needed. Open a session,
discuss/decide, close it — your decisions land in the wiki and in Linear,
and technical decisions that affect product show up in your Product
Decisions feed automatically.

*Details and rules: `CLAUDE.md`. Setup: `README.md`.*
