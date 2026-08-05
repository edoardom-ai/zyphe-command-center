# Zyphe GTM Command Center — architecture, logic, and build plan
The blueprint to review before we build. It also doubles as the spec you (or Claude Code) hand to the production build.

## What it is, in one line
A single cockpit to run your day and the whole GTM engine: see everything, get reminded, and trigger actions from one screen.

## What it can do (capabilities)
- **See:** KPIs and ARR pacing, pipeline, outbound stats, content and social, events, and every document organized by topic and destination.
- **Act:** trigger Claude skills and tool actions from buttons, draft today's follow-ups, generate call prep, run a signal scan, ship the weekly report, launch or pause a sequence, move a deal.
- **Remind:** scheduled nudges plus due-date scans; you set reminders, it surfaces what's aging so nothing drops.
- **Customize:** add, remove, and rearrange panels; edit which KPIs and targets you track; tag and file documents your way.

## Architecture (four layers)
1. **Connectors (data in and out):** HubSpot, Amplemarket, Google Calendar, Gmail/Superhuman, Slack, Drive/Docs, Granola, Clay, Smartlead, LinkedIn and social schedulers, and web signal sources.
2. **Logic / brain:** Claude skills plus scheduled tasks (and n8n for heavier flows). The loop is pull, rank, draft, trigger.
3. **Storage:** the knowledge base (Notion or Drive) for docs and the metric set, plus a light database (Airtable or Postgres) for KPIs, tasks, the document index, and reminders.
4. **UI:** the dashboard itself, a Cowork artifact for the MVP now, a Retool or custom web app for the production version.

```
Connectors  ->  Logic/brain (Claude skills + scheduled tasks + n8n)  ->  UI (dashboard)
   ^                              |                                        |
   |------------ write-back (actions: draft, move deal, launch) ----------|
Storage: knowledge base (docs, metric set) + light DB (KPIs, tasks, reminders)
```

## Modules (the panels), each with its data source and actions
| Panel | Shows | Data source | Actions |
|---|---|---|---|
| Cockpit (Today) | do-5-first, today's calls with prep, tasks due, follow-ups needed, signals | Calendar, HubSpot, Amplemarket, Granola | open prep, mark done |
| Pipeline | deals by stage with amount, segment, next step, hygiene flags | HubSpot | move stage, add note (write-back) |
| Outreach | Amplemarket/Smartlead stats, sequences, deliverability, domains | Amplemarket, Smartlead | pause/launch, add leads |
| Content and social | posts per voice, engagement, newsletter status | Taplio, Buffer, Beehiiv | schedule, atomize |
| Events | upcoming events, who goes, deadlines | Events sheet, Calendar | register, assign |
| Documents | everything by topic and destination, wired to the knowledge base | Drive, Notion | open, tag, file |
| KPIs and ARR | pacing to the ARR target, pipeline coverage, weekly targets vs actual | light DB + HubSpot | edit targets |
| AI actions | one-click Claude skills | Claude | run skill |
| Reminders and tasks | scheduled and manual reminders | scheduled tasks + DB | set, snooze, complete |

## The logic (how it thinks)
- **Morning:** pull from every connector, rank by urgency and value, present the five things to do first.
- **Continuous:** watch signals and the inbox, draft the response or the outreach, queue it for your review.
- **On demand:** a button triggers the matching Claude skill (call prep, follow-ups, signal scan, weekly report, content atomization).
- **Reminders:** scheduled tasks fire on cadence; a rolling scan surfaces anything aging so it never slips.

## Build options
- **Option A, Cowork live artifact (now):** I build it here. It reads your connectors, refreshes on open, and the buttons trigger Claude tasks. Great for validating the layout and logic immediately; write-back to HubSpot is limited until edit access is granted.
- **Option B, Claude Code production app (recommended for the full platform):** a real web app (React front end, a backend for API write-back, auth, a database, and scheduled jobs), or built on Retool. This is where full two-way control lives (move deals, launch sequences, kick off Claude tasks). I hand this spec plus the mockup as the blueprint, and Claude Code builds it properly.
- **Recommendation:** validate the layout and logic with the MVP here, then let Claude Code build the production platform from this exact spec. Best of both, fast feedback now, real power next.

## Constraints and notes
- HubSpot write is currently blocked, so write actions (moving deals, editing fields) run draft-only until edit access is granted; everything read-only works today.
- API keys and secrets live in the backend on the Claude Code build, not in the browser.
- Everything is designed to be editable: panels, KPIs, targets, tags, and layout.

## Phased roadmap
- **Phase 1 (MVP, this week):** Cockpit, Pipeline, Events, AI-action buttons, Reminders, read and draft.
- **Phase 2:** Outreach, Content and social, KPIs and ARR pacing, the document index.
- **Phase 3 (Claude Code):** full write-back control (sequences, deals), custom layouts, and proactive alerts.

## Mockup
See the interactive mockup rendered alongside this document for how the Cockpit view looks and feels.
