# CLAUDE.md — build instructions for the Zyphe GTM Command Center
You are building a production internal web app for a 2-3 person GTM team at Zyphe (a privacy-first KYC/KYB/AML company). Read README.md and ARCHITECTURE.md first. This file is your operating manual.

## What you are building
A single-cockpit dashboard that reads from and writes back to the team's tools (HubSpot, Amplemarket, Google, Slack, Clay, Smartlead), runs scheduled automations, and lets the user trigger Claude-powered actions (drafting, ranking, reports) from the UI. See README.md for modules and the phased roadmap. Build milestone by milestone (M1 → M4); do not scaffold everything at once.

## Stack and conventions
- Next.js 15 App Router, TypeScript strict, React Server Components by default; client components only where interactivity requires.
- Tailwind + shadcn/ui. Use lucide-react icons. Charts with Recharts.
- Prisma + Postgres. All DB access through `src/lib/db.ts`. Never write raw SQL in components.
- Auth with NextAuth Google provider; restrict sign-in to `@zyphe.com`. Gate every route and API handler.
- Background jobs with Inngest in `src/inngest`. Cron and event-driven functions only; no long-running work in request handlers.
- AI calls go through `src/lib/ai/claude.ts`. Use Opus for anything customer-facing or accuracy-critical, Sonnet for cheap ranking/summarizing. Always pass the shared knowledge base + the locked MetricSet as context so drafts stay on-message.
- Every third-party integration is a typed client in `src/lib/integrations/<name>.ts` with a narrow interface. No integration logic in components or pages.

## Golden rules
- Secrets are server-only. Never import an integration client into a client component. Read env via `src/lib/env.ts` (zod-validated).
- Write-back is gated. Any mutation to an external system (move a HubSpot deal, launch a sequence) goes through an action handler that checks `FEATURE_WRITEBACK` and the connector's granted scopes. If write is not permitted, produce a draft record in the `ActionLog`/`Draft` table for human approval instead of failing.
- Domain safety: any lead-list operation must validate addresses and must never send from or modify the primary `zyphe.com` domain; cold sends route through configured secondary domains only.
- One MetricSet: never hardcode metrics (verification time, cost savings, etc.) in components; read them from the `MetricSet` table so the whole app stays consistent.
- Idempotent syncs: all sync jobs upsert by external id and are safe to re-run.
- Everything editable: panels, KPI targets, tags, and reminders are user-editable and persisted.

## Build order (do not skip)
1. Scaffold Next.js + Tailwind + shadcn + Prisma + NextAuth. Add `src/lib/env.ts`, `src/lib/db.ts`, `src/lib/auth.ts`.
2. Implement the schema in `prisma/schema.prisma` (provided), migrate, seed with sample Zyphe data (Juventus, Scalapay, Banco Inter, 5 Pillars deals; sample events, reminders).
3. App shell: sidebar nav for the 9 modules, top KPI bar, responsive.
4. M1 panels: Cockpit (reads deals, tasks, reminders, calendar), Pipeline (HubSpot read), Reminders CRUD.
5. Integrations layer: start with HubSpot (read), Google Calendar (read), then Amplemarket, Slack, Clay, Smartlead. Each behind the typed client interface in ARCHITECTURE.md.
6. AI actions: `POST /api/actions/run` dispatches to named skills (call-prep, draft-follow-ups, signal-scan, weekly-report, atomize-content). Each returns a draft the user reviews.
7. Inngest jobs: morningCockpit (cron 07:00 Europe/Rome), signalScan (hourly), followUpGuardian (daily 18:00), weeklyReport (Thu 09:00), syncHubSpot/syncAmplemarket (every 15 min).
8. Remaining panels (Outreach, Events, Content, Documents, KPIs) per roadmap.

## Testing and quality
- Vitest for unit tests on integration clients and skill handlers (mock the external APIs).
- Type-check and lint clean before each milestone.
- Seed data must let the whole UI render without any live API keys (offline-friendly demo mode via `MOCK_INTEGRATIONS=true`).

## Definition of done per milestone
- The milestone's panels render with real or seeded data, all routes are auth-gated, no secret is exposed client-side, and `pnpm build` + `pnpm test` pass.
