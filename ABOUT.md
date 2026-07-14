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
- Runs from **Claude Code** in the terminal, on a schedule.
- **Draft-only** autonomy: it gathers, prioritizes, drafts, and reminds — it never sends or closes anything without my click.
- Tracks not just tasks but **commitments, risks/issues, decisions, and customer updates** (account-management lens).
- Migrates 2+ years of history: Granola notes (urgent — free-tier 30-day window) and OneNote action items.
- Obsidian as a local, git-versioned durable archive.

**Out of scope (for now)**
- Auto-sending email/Slack or auto-closing Jira (revisit after trust is established).
- Reading DMs / private channels.
- Employee monitoring of any kind.

## 3. Current status

| Area | Status |
|---|---|
| Plan / architecture | ✅ Done — see `docs/chief-of-staff-plan.md` |
| Repo + instructions | ✅ This repo |
| Granola rescue (30-day risk) | ⏳ **Do first** |
| Notion schema bootstrap | ⬜ Not started |
| Harvester + Triage | ⬜ Not started |
| Daily Brief + Reminders | ⬜ Not started |
| Draft Assistant | ⬜ Not started |
| OneNote 2-yr migration | ⬜ Not started |
| Obsidian archive/sync | ⬜ Not started |

## 4. How I'll use this repo

1. Clone it and open a terminal in the folder.
2. Run Claude Code — it reads `CLAUDE.md` automatically and can read `docs/chief-of-staff-plan.md` for the full spec.
3. Work through the phases (see the plan's rollout section), starting with the Granola rescue.
4. Keep this `ABOUT.md` updated: log changes, tick off status, add to the backlog.

## 5. Changelog

_Newest first. One line per meaningful change._

- **2026-07-14** — Repo created. Plan finalized. Added item `Type` (Action/Commitment/Risk/Decision/Customer Update) + Accounts DB + richer daily-brief sections per Aditi's feedback.
- **2026-07-14** — Initial architecture drafted (Notion backbone, Claude Code runtime, draft-only).

## 6. Backlog / ideas

_Unordered parking lot. Promote items into the plan when ready._

- [ ] Confirm timezone + daily brief time before scheduling.
- [ ] List the exact Slack channels the harvester should watch.
- [ ] Define the email importance filter (direct-recipient-only vs. cc'd).
- [ ] Write the Jira JQL for "my" issues.
- [ ] Decide Obsidian vault location; enable Obsidian Git.
- [ ] Pick OneNote export method (manual vs. Graph API / community exporter).
- [ ] "Metrics pulse" — one data nugget per brief (nice-to-have).
- [ ] Flag patterns across briefs ("this risk appeared 3 days running").
- [ ] Graduate one narrow flow past draft-only once trusted.

## 7. Decisions log

_Why things are the way they are._

- **Notion over Obsidian as backbone** — best MCP write support + collaborative views; Obsidian kept as durable archive so I'm never locked in.
- **Claude Code as runtime** — max control, full MCP + file access, git-friendly, schedulable from terminal.
- **Draft-only** — safest starting posture; I stay the sender and decider.
- **One Items DB with a `Type` field** (not separate DBs) — one dedupable store, many filtered views; avoids database sprawl.

## 8. Key links

- Full spec: [`docs/chief-of-staff-plan.md`](./docs/chief-of-staff-plan.md)
- Agent instructions: [`CLAUDE.md`](./CLAUDE.md)
- Remote: https://github.com/ashethtrk/AI-CoS
