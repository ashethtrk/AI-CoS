# AI Chief of Staff — About This Project

> **What this file is:** the human-readable logbook for the project — scope, status, what's changed, and what's next. Read this to understand *why* and *where we are*.
> **What it is NOT:** the build instructions. Those live in [`CLAUDE.md`](./CLAUDE.md) (directives for the agent) and the full spec in [`docs/chief-of-staff-plan.md`](./docs/chief-of-staff-plan.md).
>
> **The two-file split, in one line:** `ABOUT.md` is for *me* (the human) to track scope/decisions/backlog; `CLAUDE.md` is for *the agent* to know how to build and operate. Keep them separate so instructions stay clean and the log stays honest.

---

## 1. The one-liner

A personal **AI Chief of Staff** that aggregates action items from Granola, email, Slack, and Jira, tracks them in Notion, reminds me about what matters, and drafts responses — so I start each day prepared instead of in triage mode.

## 2. Scope

**In scope**
- Notion is the single system of record (full switch from OneNote).
- Runs from **Claude Code** in the terminal, on a schedule (launchd, laptop-open hours).
- **Draft-only** autonomy: it gathers, prioritizes, drafts, and reminds — it never sends or closes anything without my click.
- Tracks not just tasks but **commitments, risks/issues, decisions, and customer updates** (account-management lens).
- Migrates 2+ years of history: Granola notes (urgent — free-tier 30-day window) and OneNote action items.
- Obsidian as a local, git-versioned durable archive — **from day 1** (the Granola rescue populates the vault).

**Out of scope (for now)**
- Auto-sending email/Slack or auto-closing Jira (revisit after trust is established).
- Reading DMs / private channels.
- Employee monitoring of any kind.

## 3. Current status

| Area | Status |
|---|---|
| Plan / architecture | ✅ Done — see `docs/chief-of-staff-plan.md` |
| Repo + instructions | ✅ This repo |
| Design review + doc updates | ✅ 2026-07-14 |
| Obsidian vault setup (day 1) | ⏳ **Do first** (with Granola rescue) |
| Granola rescue (30-day risk) | ⏳ **Do first** |
| Notion schema bootstrap | ⬜ Not started |
| Harvester + Triage | ⬜ Not started |
| Daily Brief + Reminders | ⬜ Not started |
| Draft Assistant | ⬜ Not started |
| OneNote 2-yr migration | ⬜ Not started |
| Weekly Notion→vault export | ⬜ Not started |

## 4. How I'll use this repo

1. Clone it and open a terminal in the folder.
2. Run Claude Code — it reads `CLAUDE.md` automatically and can read `docs/chief-of-staff-plan.md` for the full spec.
3. Work through the phases (see the plan's rollout section), starting with the Granola rescue.
4. Keep this `ABOUT.md` updated: log changes, tick off status, add to the backlog.

## 5. Changelog

_Newest first. One line per meaningful change._

- **2026-07-14** — Brief delivery decided: Slack DM to Alvint (single carve-out to the no-send rule — to-me-only, briefs-only). Rollout compressed: build everything ASAP, tune over the following weeks (plan §10 rewritten). Owner name corrected to Alvint.
- **2026-07-14** — Design review with Claude. Cadence set: 9AM PT full brief + 1PM PT sanity-check delta, weekdays. launchd over cron (laptop sleeps). Dropped `people.md` — Notion People DB is canonical. Added cross-source dedup at triage, `state/` harvest bookkeeping, and a headless-MCP validation gate before scheduling. Standardized "Items DB" naming. Obsidian confirmed in scope from day 1 (vault `~/Documents/Obsidian/CoS`). Scaffolded `context/priorities.md`.
- **2026-07-14** — Repo created. Plan finalized. Added item `Type` (Action/Commitment/Risk/Decision/Customer Update) + Accounts DB + richer daily-brief sections per Aditi's feedback.
- **2026-07-14** — Initial architecture drafted (Notion backbone, Claude Code runtime, draft-only).

## 6. Backlog / ideas

_Unordered parking lot. Promote items into the plan when ready._

- [x] Confirm timezone + daily brief time — **9AM + 1PM PT, weekdays** (2026-07-14).
- [x] Decide Obsidian vault location; enable Obsidian Git — **`~/Documents/Obsidian/CoS`, Git from day 1** (2026-07-14).
- [ ] List the exact Slack channels the harvester should watch.
- [ ] Define the email importance filter (direct-recipient-only vs. cc'd).
- [ ] Write the Jira JQL for "my" issues.
- [ ] Pick OneNote export method (manual vs. Graph API / community exporter).
- [ ] Validate headless MCP access before turning on launchd schedules.
- [ ] Fill in `context/priorities.md`; write `context/reference-brief.md` (a gold-standard brief to match my vibe).
- [ ] "Metrics pulse" — one data nugget per brief (nice-to-have).
- [ ] Flag patterns across briefs ("this risk appeared 3 days running").
- [ ] Graduate one narrow flow past draft-only once trusted.

## 7. Decisions log

_Why things are the way they are._

- **Notion over Obsidian as backbone** — best MCP write support + collaborative views; Obsidian kept as durable archive so I'm never locked in.
- **Claude Code as runtime** — max control, full MCP + file access, git-friendly, schedulable from terminal.
- **Draft-only** — safest starting posture; I stay the sender and decider.
- **One Items DB with a `Type` field** (not separate DBs) — one dedupable store, many filtered views; avoids database sprawl.
- **2×/day cadence: full brief 9AM + sanity-check delta 1PM (PT)** (2026-07-14) — midday run is a triage-QA checkpoint on what came in since morning, not a second brief or a continuous stream.
- **launchd over cron** (2026-07-14) — this runs on a laptop that sleeps; launchd fires missed jobs on wake, cron silently skips them.
- **No `people.md`** (2026-07-14) — Notion People DB is the single source for people/watch levels; avoids two sources of truth. `priorities.md` is the only local context file.
- **Obsidian from day 1** (2026-07-14) — the Granola rescue populates the vault immediately, so learning the app starts with real content instead of an empty folder.
- **Slack DM delivery for briefs** (2026-07-14) — briefs are pushed, not pulled. Safe exception to draft-only because recipient (me) and content (briefs) are both fixed; everything else stays unsent.
- **Build ASAP, tune over weeks** (2026-07-14) — the 4-week phased rollout compressed into one build sprint; calendar time is for tuning triage/brief quality against real use, not for building.

## 8. Key links

- Full spec: [`docs/chief-of-staff-plan.md`](./docs/chief-of-staff-plan.md)
- Agent instructions: [`CLAUDE.md`](./CLAUDE.md)
- Remote: https://github.com/ashethtrk/AI-CoS
