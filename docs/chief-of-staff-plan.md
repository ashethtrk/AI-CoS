# Chief of Staff Agent — Build Plan

**Owner:** Alvin
**Backbone:** Notion (full switch from OneNote)
**Runtime:** Claude Code (terminal, headless + scheduled)
**Autonomy:** Draft-only (gathers, prioritizes, drafts, reminds — never sends or closes without your click)
**Archive/bonus:** Obsidian (local-first durable memory + thinking layer)

---

## 0. TL;DR of the design

You're building one "brain" (a Notion workspace) fed by a small fleet of **harvester → triage → surface → draft** modules that run from Claude Code on a schedule. Everything an agent finds becomes a row in one **Action Items** database with a stable source ID so nothing is duplicated and nothing is lost. Notion is the live operational hub; Obsidian is the durable, private, git-versioned archive. The single highest-value artifact is not the code — it's your **instruction/context files** (people, priorities, definition of "important"). You tune those weekly.

**Do this first, this week (time-sensitive):** Rescue your Granola history. On the free plan only the last ~30 days are visible in-app; older notes are retained but locked. Pull them out now (three redundant methods below).

---

## 1. Architecture at a glance

```
        SOURCES (read)                 CORE BRAIN (Notion)              SURFACES (you)
  ┌───────────────────────┐      ┌──────────────────────────┐    ┌────────────────────┐
  │ Granola  (MCP)         │      │  Action Items DB  ◄──────┼────┤  Daily Brief (8am) │
  │ Email    (MCP)         │─────►│  Meetings/Notes DB       │    │  Reminders/Nudges  │
  │ Slack    (MCP)         │ HARV │  Daily Briefs DB         │───►│  Draft replies     │
  │ Jira     (MCP)         │ EST  │  People DB               │    │  Weekly context    │
  │ Calendar (MCP)         │      │  Projects/Areas DB       │    │  Weekly wrap       │
  │ Google Drive (MCP)     │      │  Living Context (page)   │    │  Ad-hoc Q&A        │
  └───────────────────────┘      └──────────────────────────┘    └────────────────────┘
                                            │
                                            ▼  (weekly one-way export)
                                   Obsidian vault (git) — cold archive + graph + thinking
```

**Data flow:** Harvesters read each source, extract candidate action items, dedupe against existing Source IDs, and drop new items into the Action Items DB "Inbox." Triage prioritizes them. Surfaces (brief, reminders, drafts) read the DB and present to you. You approve; the agent never acts on your accounts without a click.

---

## 2. The core data model (Notion)

Six objects. Build these first — everything else depends on them.

### 2.1 Items DB — the heart of the system
> **Per Aditi's feedback:** this is not just a task list. One `Type` property lets the agent track **customer commitments, risks/issues, and decisions distinctly from generic action items**, while keeping everything in one dedupable table. Notion **views** then split them (a "Commitments" view, a "Risks" view, a "Decisions needed" view, a per-account view). One store, many lenses.

| Property | Type | Notes |
|---|---|---|
| Name | Title | Phrased for its type — an action as a verb ("Reply to X"); a commitment as a promise ("Deliver pricing by 7/22"); a risk as a condition; a decision as a question. |
| **Type** | Select | **`Action`** · **`Commitment`** (esp. customer-facing) · **`Risk/Issue`** · **`Decision`** (needed/made) · **`Customer Update`** (due to a customer) |
| Status | Select | `Inbox` → `Next` → `In Progress` → `Waiting` → `Done` / `Dropped`. For Risks: `Open`/`Mitigating`/`Closed`. For Decisions: `Needed`/`Made`. |
| Priority | Select | `P0` (today/blocking) · `P1` (this week) · `P2` · `P3` |
| Source | Select | `Granola` · `Email` · `Slack` · `Jira` · `Meeting` · `Manual` |
| Source ID | Text | **Dedup key.** email msg-id / slack ts / jira key / granola note-id. Prevents re-import duplicates. |
| Source Link | URL | Deep link back to the origin |
| Owner | Select | `Me` · `Delegated` |
| Account | Relation → Accounts | The customer/account this belongs to (blank = internal) |
| Waiting On | Relation → People | Who you're blocked by (drives follow-ups + dependencies) |
| Dependency | Relation → Items | Blocks/blocked-by another item (unresolved-dependency tracking) |
| Milestone | Text | Named milestone this rolls up to (e.g., "Pilot go-live") |
| Due / Commit Date | Date | For commitments: the date you promised the customer |
| At Risk | Checkbox | Agent flips this when a commitment's due date is near and status isn't on track |
| Project/Area | Relation → Projects | |
| Next Action | Text | The single next physical step |
| Context | Text | 1–2 line why-this-matters |
| Created / Completed | Date | Created auto; Completed set on Done |
| First Seen In Brief | Date | Lets the agent flag "appeared 3 days running" |

### 2.2 Meetings/Notes DB — Granola + meeting archive
`Title · Date · Attendees (relation→People) · Source (Granola/Manual) · Summary · Transcript (link or child page) · Extracted Actions (relation→Action Items) · Folder/Tag`

### 2.3 Daily Briefs DB
`Brief (title) · Date · Status (Draft/Sent/Skipped) · Topline (1 line) · Risks/Blockers · Key Links · Body`

### 2.4 People DB — powers prioritization
`Name · Role · Relationship (Direct report / XFN partner / Leadership / External) · Slack handle · Email · Notes · Watch level (High/Med/Low)`

### 2.5 Projects/Areas DB
`Name · Status · Priority · Owner · Related Items (relation) · Related Docs`

### 2.6 Accounts DB — for customer/account-management use cases
`Account name · Health (Green/Yellow/Red) · Owner (you/CSM) · Stage (Pilot/Onboarding/Live/Renewal) · Key contacts (relation→People) · Open Commitments (relation→Items) · Open Risks (relation→Items) · Next Milestone (date) · Last Update Sent (date) · Update Cadence (e.g., weekly)`

This is what powers "customer updates due" and per-account rollups in the brief.

### 2.7 Living Context — a single page (not a DB)
Running memory the agent rewrites weekly: current top priorities, active initiatives, risks, per-report focus, org changes / OOO coverage, "watch list." This is the agent's long-term memory and the thing you'll get the most from curating.

---

## 3. The modules (your "agents")

The Notion-blog example is one agent with triggers. In Claude Code it's cleaner to make each job a **slash command / instruction file** invoked by a schedule. Think of them as one team with specialized roles.

### 3.1 Harvester (ingestion) — runs 2–4×/day
Sweeps every source, extracts candidate actions, dedupes, writes to Inbox.
- **Granola sub-harvester:** `list_meetings` → for new notes, `get_meeting_transcript`; extract action items + decisions; write meeting to Meetings DB and actions to Inbox.
- **Email sub-harvester:** `search_threads` for unread/unanswered where you're a direct recipient or @mentioned; extract "needs reply / needs decision / FYI"; skip newsletters + notifications.
- **Slack sub-harvester:** `search` your channels/threads for questions directed at you or unresolved threads you're in; extract asks. (No DMs/private unless you explicitly allow.)
- **Jira sub-harvester:** `searchJiraIssuesUsingJql` for issues assigned to you / you reported / recently updated; sync status into Action Items (read-only to start).

**Dedup rule (critical):** before creating any item, query Action Items by `Source ID`. If it exists, update; else create. This is what makes re-runs safe.

### 3.2 Triage / Prioritizer — runs after each harvest
**Classifies each item's `Type`** (Action / Commitment / Risk-Issue / Decision / Customer Update) — a promise to a customer becomes a Commitment, "we need to decide…" becomes a Decision, a flagged blocker becomes a Risk/Issue. Then links `Account` + `Project` + `People`, moves Inbox → Next/Waiting, sets Priority (§4), writes a one-line Next Action, and flips `At Risk` when a commitment's due date is near without progress. Flags anything stale (Waiting > N days) or recurring.

### 3.3 Daily Brief — weekdays 8:00 (your TZ)
**Per Aditi's feedback, the brief surfaces more than my tasks.** It's organized into fixed sections, each drawn from a filtered view of the Items/Accounts DBs:

1. **Today's meetings** — attendees, related docs/notes, 2–3 prep bullets each.
2. **⚠️ Commitments at risk** — `Type=Commitment` where `At Risk=true` or due ≤ 3 days and not on track (customer-facing ones first, grouped by Account).
3. **📅 Upcoming milestones** — `Milestone` / Accounts `Next Milestone` landing in the next 1–2 weeks.
4. **🔗 Unresolved dependencies** — items `Waiting On` someone or with an open `Dependency` link, sorted by how long they've been stuck.
5. **🧭 Decisions needed** — `Type=Decision`, `Status=Needed`, with who owns the call and by when.
6. **📣 Customer updates due** — Accounts whose `Update Cadence` says a check-in is overdue vs. `Last Update Sent`.
7. **✅ Top priorities (my actions)** — top 3–5 generic action items.
8. **✍️ Replies to draft today** + a rotating **deep-dive** (a Drive/Confluence doc or a metric).

Writes a row to Daily Briefs DB. **Informational only.**

### 3.4 Draft Assistant — on demand + flagged in brief
For items tagged "needs reply," drafts the email (`create_draft`) or Slack message (`slack_send_message_draft`) in **your** voice — saved as a draft, never sent. Brief links straight to the draft so you review → tweak → send.

### 3.5 Reminder / Follow-up — daily
Scans `Waiting On` + `Due`. Surfaces "you've been waiting 4 days on Priya re: X — want a nudge draft?" and "P1 due tomorrow, no next action set." Produces nudge drafts; you send.

### 3.6 Weekly Context Update — Mondays
Rewrites the Living Context page from the week's meetings, closed/added actions, and org signals.

### 3.7 Weekly Wrap — Fridays
End-of-week digest from spaces you participate in: wins, risks, open questions, follow-ups worth making. Writes to a page + optional email draft to yourself.

### 3.8 Ad-hoc — anytime
`claude` in the repo → ask "what's on my plate for the pricing project?" / "draft a reply to the CFO thread" / "what did I commit to in yesterday's standup?"

---

## 4. Prioritization logic (define "important" once, tune forever)

The agent ranks with a simple weighted rubric. Put the specifics in `context/priorities.md`.

- **Who** (from People DB Watch level): Leadership > Direct reports > XFN partners > everyone else.
- **Signal of urgency:** direct question to you · blocks someone else · due ≤ 48h · from someone on the watch list · flagged "decision needed."
- **Recurrence:** appeared in a prior brief and still open → bump priority ("3 days running" gets escalated).
- **Decay:** FYIs and newsletters never become action items; auto-drop after review.

Output: P0 = today/blocking, P1 = this week, P2 = soon, P3 = someday/maybe.

---

## 5. Autonomy boundaries (draft-only contract)

Bake this verbatim into `CLAUDE.md` so it's non-negotiable:

- **May read** anything you already have access to (your calendar, your email, channels you're in, your Jira, your Drive).
- **May write** freely to **your own Notion workspace** and your **Obsidian vault**.
- **May create drafts** in email and Slack — always as unsent drafts.
- **Must NOT** send email/Slack, resolve/close Jira, transition tickets, delete anything, or touch DMs/private channels — without an explicit, per-action go-ahead from you.
- **Jira starts read-only.** Graduate to "create my own tickets" once trusted; keep transitions/comments as drafts-for-approval.
- Everything is **informational + preparatory**. You remain the sender and the decider.

Graduation path (later): pick one narrow, safe flow (e.g., auto-labeling email, or creating Jira tickets from meeting actions) and promote just that one.

---

## 6. Migration plan

### 6.1 Phase 0 — Granola rescue (DO THIS WEEK) ⏰
Three redundant methods; do at least the first two.

1. **Via your connected Granola MCP (fastest, structured):** have Claude Code walk `list_meetings` / `list_meeting_folders`, then `get_meeting_transcript` for each, and write every meeting into the Notion Meetings DB **and** as a markdown file in the Obsidian vault. This also extracts historical action items into the Action Items DB.
2. **Official CSV export (belt-and-suspenders):** Granola → Settings → Profile → **Generate CSV**. Includes title, summary, and transcript; emailed to you within a few hours. Keep the file as raw backup.
3. **Local-cache export tool (captures anything the API/CSV misses):** community tools read Granola's on-disk cache and write full markdown — notably **Granary** (preserves transcripts even after Granola purges them) and **granola-export**/**granola-cli**. Run once into the Obsidian vault.

Why all three: the app hides >30-day notes, so the MCP may not surface the oldest ones; the CSV and the local-cache tools reach further back. Capture now, dedupe later.

### 6.2 Phase 1 — 2+ years of action items out of OneNote
OneNote has weak automation/MCP support, so this is an export-then-parse job:
- Export notebooks from OneNote (File → Export → Word/PDF per section) **or** use a community `onenote-md-exporter` / the Microsoft Graph API to get markdown.
- Point Claude Code at the exported files: it parses each page, extracts action items (with any dates/status cues), infers Project/Area, and **bulk-creates** rows in the Action Items DB (mostly as `Done` or `Dropped` for historical, with the live ones flagged for triage).
- Keep the raw OneNote export in the Obsidian vault as an immutable archive.

### 6.3 Phase 2 — steady state
Once migrated, OneNote is retired; new capture happens automatically via harvesters, and manual notes go straight into Notion (or Obsidian, synced).

---

## 7. Claude Code project layout

```
chief-of-staff/
├── CLAUDE.md                      # THE product: role, rules, autonomy contract, formatting
├── .mcp.json                      # granola, email, slack, jira, calendar, drive, notion
├── context/
│   ├── people.md                  # watch list + relationships (mirror of People DB)
│   ├── priorities.md              # current focus areas + "what important means"
│   └── reference-brief.md         # a gold-standard brief so it matches your vibe
├── .claude/commands/
│   ├── harvest.md                 # run all harvesters + dedupe
│   ├── triage.md                  # prioritize the Inbox
│   ├── brief.md                   # build the morning brief
│   ├── drafts.md                  # draft replies for flagged items
│   ├── followups.md               # waiting-on + due nudges
│   ├── weekly-context.md          # rewrite Living Context
│   └── weekly-wrap.md             # Friday digest
└── migration/
    ├── granola-rescue.md
    └── onenote-import.md
```

**Scheduling:** run headless from cron (macOS `launchd` or `cron`), e.g.
```
0 6 * * 1-5   cd ~/chief-of-staff && claude -p "/harvest && /triage && /brief"
0 12 * * 1-5  cd ~/chief-of-staff && claude -p "/harvest && /triage"
30 16 * * 5   cd ~/chief-of-staff && claude -p "/weekly-wrap"
0 9 * * 1     cd ~/chief-of-staff && claude -p "/weekly-context"
```
(You can also run any command interactively whenever you like.)

---

## 8. Ready-to-paste prompts for Claude Code

### 8.1 Bootstrap the Notion schema
> Create the following databases in my Notion workspace under a new "Chief of Staff" page: Items, Meetings/Notes, Daily Briefs, People, Projects/Areas, Accounts — with exactly the properties in section 2 of my plan (I'll paste it), including the Items `Type` property (Action/Commitment/Risk-Issue/Decision/Customer Update), `Account` and `Dependency` relations, and the `At Risk` checkbox. Add relations: Items↔People (Waiting On), Items↔Items (Dependency), Items↔Projects, Items↔Accounts, Meetings↔People (Attendees), Meetings↔Items, Accounts↔People. Then create saved views on Items: "Commitments," "Risks/Issues," "Decisions needed," "At risk," and "By account." Create a single "Living Context" page. Confirm each database's ID back to me.

### 8.2 Granola rescue
> Using the Granola MCP, list every meeting and folder I have. For each meeting, fetch its transcript and summary. Write each as (a) a row in my Notion Meetings/Notes DB and (b) a markdown file at ~/obsidian/CoS/granola/YYYY-MM-DD-title.md. Extract action items assigned to or committed by me and create them in the Action Items DB with Source=Granola, a Source ID = the meeting/note id, and Source Link. Skip any meeting whose Source ID already exists. Give me a count of meetings and actions imported, and flag the oldest date you could reach.

### 8.3 OneNote import
> I've exported my OneNote notebooks to markdown at ~/exports/onenote/. Parse every file, extract action items (respect any checkbox/status/date cues), infer a Project/Area, and bulk-create rows in the Action Items DB. Mark clearly-completed ones as Done, ambiguous ones as Inbox for my triage. Use Source=Manual and a Source ID derived from the file path + heading so re-runs don't duplicate. Summarize what you created grouped by Project.

### 8.4 CLAUDE.md seed (paste as the file's core)
> You are my Chief of Staff. Purpose: help me start each day calm, prepared, and on top of what matters across Granola, email, Slack, Jira, calendar, and Drive — with Notion as the system of record. Read context/people.md and context/priorities.md before every run. **Rules: draft-only.** You may read anything I can access and write only to my Notion workspace and Obsidian vault. You may create unsent drafts in email/Slack. You must NEVER send messages, transition/close Jira, delete anything, or read DMs/private channels without my explicit per-action approval. Always dedupe against Source ID before creating Action Items. Keep briefs conversational and scannable, with links everywhere. This is informational and preparatory only — I am always the sender and the decider.

---

## 9. Obsidian bonus — how it fits

Position the two tools by job, not as rivals:

- **Notion = live operational hub.** Structured databases, relations, collaborative, best MCP write support, where the agent works and where briefs land.
- **Obsidian = durable private archive + thinking layer.** Plain markdown on your disk, git-versioned (nothing ever lost, full history), fast offline, backlinks/graph for connecting ideas across two years.

**Sync model (one-way, low-friction):**
- Granola transcripts and OneNote raw exports land **directly** in the vault (immutable cold storage).
- A weekly job exports Notion (briefs, closed actions, Living Context) to markdown in the vault — so Obsidian becomes the permanent record even if you ever leave Notion.
- Use **Obsidian Git** for versioned backup, **Dataview** to query action-item frontmatter locally, and the graph view for your knowledge network.
- Keep writes primarily Notion→Obsidian to avoid two-way sync headaches. Obsidian is where you *think and keep*; Notion is where the agent *operates*.

---

## 10. Phased rollout (≈4 weeks)

- **Week 0:** Granola rescue (§6.1) + bootstrap Notion schema (§8.1) + fill People/priorities context files.
- **Week 1:** Harvester + Triage live; review the Inbox manually each day; tune dedupe + "important" rules.
- **Week 2:** Daily Brief + Reminders on a schedule; refine format against your reference brief.
- **Week 3:** Draft Assistant + Weekly Context + Weekly Wrap.
- **Week 4:** OneNote 2-year migration + Obsidian archive/sync; retire OneNote.
- **Ongoing:** the real work — update `people.md`/`priorities.md`/Living Context weekly; prune noisy sources; consider graduating one narrow flow past draft-only.

---

## 11. Open decisions to make before you start coding

1. **Timezone + brief time** for the schedule (default 8am your local).
2. **Slack scope:** which channels the harvester watches (list them in `people.md`/`priorities.md`), and confirming DMs stay off-limits.
3. **Email importance filter:** direct-recipient-only vs. also cc'd; which labels/senders count as "important."
4. **Jira JQL** for "my" issues (assignee, reporter, watched, project keys).
5. **Obsidian vault location** and whether to enable Obsidian Git now or later.
6. **OneNote export method** (manual per-section vs. Graph API / community exporter) — depends on how many notebooks/sections you have.
```

