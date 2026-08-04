# Architecture — Zyphe GTM Command Center

## System overview
```
Browser (Next.js RSC + client panels)
        |  auth: NextAuth (Google, @zyphe.com)
        v
Next.js app (Vercel)  ── API route handlers / server actions
        |                         |
        |                         +--> src/lib/ai/claude.ts (Anthropic)
        |                         +--> src/lib/integrations/* (typed clients)
        v                                         |
   Postgres (Prisma)  <----- Inngest jobs --------+  (cron + event driven)
        ^                                         |
        +------------------ upsert/sync ----------+
External systems: HubSpot, Amplemarket, Smartlead, Clay, Google (Calendar/Gmail/Drive), Slack, Granola
```

## Repo structure
```
command-center/
  CLAUDE.md  README.md  ARCHITECTURE.md  .env.example  package.json
  prisma/
    schema.prisma
    seed.ts
  src/
    app/
      (dashboard)/
        layout.tsx            sidebar nav + top KPI bar
        cockpit/page.tsx
        pipeline/page.tsx
        outreach/page.tsx
        content/page.tsx
        events/page.tsx
        documents/page.tsx
        kpis/page.tsx
        reminders/page.tsx
      api/
        cockpit/route.ts          GET aggregate for today
        deals/route.ts            GET list · PATCH stage/note (write-back)
        signals/route.ts          GET · POST action
        reminders/route.ts        CRUD
        events/route.ts           CRUD
        documents/route.ts        GET (Drive index)
        kpis/route.ts             GET/PATCH
        outreach/route.ts         GET (Amplemarket/Smartlead)
        actions/run/route.ts      POST { skill, input } -> draft
        auth/[...nextauth]/route.ts
        inngest/route.ts          Inngest serve endpoint
    components/
      panels/  (CockpitPanel, PipelinePanel, SignalsPanel, RemindersPanel,
                AiActionsPanel, OutreachPanel, EventsPanel, ContentPanel,
                DocumentsPanel, KpiPanel)
      ui/      (shadcn components)
      shell/   (Sidebar, TopBar, MetricCard)
    lib/
      env.ts        zod-validated process.env
      db.ts         Prisma client singleton
      auth.ts       NextAuth config (Google, domain allowlist)
      ai/claude.ts  Anthropic wrapper + skill prompts
      skills/       call-prep.ts, draft-follow-ups.ts, signal-scan.ts,
                    weekly-report.ts, atomize-content.ts
      integrations/ hubspot.ts amplemarket.ts smartlead.ts clay.ts
                    google.ts slack.ts granola.ts
    inngest/
      client.ts
      functions/ morning-cockpit.ts signal-scan.ts follow-up-guardian.ts
                 weekly-report.ts sync-hubspot.ts sync-amplemarket.ts
```

## Integration clients (typed interfaces)
Each file exports a small, typed surface. Keep the vendor SDK/HTTP details inside; panels/skills call only these.

- `hubspot.ts`: `listDeals()`, `getDeal(id)`, `updateDealStage(id, stage)`, `addDealNote(id, note)`, `listContacts()`, `flagHygiene(deal)` → returns hygiene issues (missing amount/segment/next-step, stale). Write methods respect `FEATURE_WRITEBACK`.
- `amplemarket.ts`: `getSequenceStats()`, `getReplies()`, `getSignals()`, `listSequences()`, `pauseSequence(id)` (gated).
- `smartlead.ts`: `getCampaignStats()`, `getInboxHealth()`, `listDomains()`.
- `clay.ts`: `runEnrichment(accounts)`, `getSignals(icp)` (funding, hiring, tech, news).
- `google.ts`: `listTodayEvents()`, `listThreadsAwaitingReply()`, `listDriveDocs(folder)` (Calendar, Gmail, Drive scopes).
- `slack.ts`: `postMessage(channel, blocks)`, `readChannel(id)` (for the weekly report + #form-submission watch).
- `granola.ts`: `getLatestTranscript(meeting)` (if API available; else omit).

## API endpoints (contract)
- `GET /api/cockpit` → `{ do5, todaysCalls, tasksDue, followUpsNeeded, signals }`.
- `GET /api/deals` / `PATCH /api/deals` → list; update stage or note (write-back, gated).
- `GET /api/signals` / `POST /api/signals` → list; action = draft outreach for a signal.
- `POST /api/actions/run` → `{ skill: "call-prep"|"draft-follow-ups"|"signal-scan"|"weekly-report"|"atomize-content", input }` returns `{ draftId, output }`.
- CRUD: `/api/reminders`, `/api/events`, `/api/kpis`.
- `GET /api/outreach`, `GET /api/documents`.

## Scheduled jobs (Inngest)
- `morning-cockpit` cron `0 7 * * *` (Europe/Rome): pull calendar + HubSpot + Amplemarket + signals, rank, build the do-5-first, post a summary to Slack DM.
- `signal-scan` cron `0 * * * *`: pull Clay/Amplemarket signals across ICPs, upsert, draft outreach for high scores.
- `follow-up-guardian` cron `0 18 * * *`: find prospects dark > N days, draft next touch, queue.
- `weekly-report` cron `0 9 * * 4` (Thu): compile Amplemarket + pipeline + activity + ARR pacing, post to Slack.
- `sync-hubspot` / `sync-amplemarket` every 15 min: idempotent upserts.

## AI skills (src/lib/skills)
Each skill: takes structured input, pulls context from DB (knowledge base + MetricSet), calls Claude, returns a draft persisted to `Draft`/`ActionLog`. Skills: `call-prep`, `draft-follow-ups`, `signal-scan`, `weekly-report`, `atomize-content`. Reuse the canonical assets (value emails v3, call scripts v3, ICP research) as prompt context.

## Auth and security
- NextAuth Google; `signIn` callback rejects non-`@zyphe.com`.
- All API handlers call `requireUser()`.
- Env validated in `src/lib/env.ts`; server-only. `MOCK_INTEGRATIONS=true` returns seeded fixtures so the app runs with no keys.
- Feature flags: `FEATURE_WRITEBACK` (default false until HubSpot edit granted).
