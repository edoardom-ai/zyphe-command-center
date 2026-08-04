# Zyphe GTM Command Center — build brief (PRD)
A production internal web app: one cockpit to run the day and the GTM engine. See everything (KPIs, pipeline, outbound, content, events, documents), get reminded, and trigger actions (Claude tasks, HubSpot/Amplemarket write-back) from one screen. This package is the handoff for Claude Code.

## Goal
Give a 2-3 person GTM team the leverage of a larger one: nothing falls through, every prospect is followed up, the CRM is always current, prep is always ready, and one click runs the automations. It plugs into the real tools and writes back to them.

## Users
- Edoardo (primary), Hendrik, Michelangelo. Google Workspace accounts (zyphe.com). Auth via Google OAuth, allowlist the zyphe.com domain.

## Non-negotiables (product principles)
- One screen to start the day; a ranked "do these 5 first."
- Read and act, not just dashboards: trigger automations and write back from the UI.
- Protect the sending domain: any list action validates and never touches the primary domain.
- One source of truth for metrics (a single MetricSet the whole app reads).
- Fully editable: panels, KPIs, targets, tags, layout.
- Draft-only fallback: where a connector lacks write scope (HubSpot edit currently blocked), actions produce a draft for human approval instead of failing.
- Mobile-first: fully responsive, and shipped as an installable PWA (home-screen icon, offline shell) with web push notifications for reminders, new signals, and hot inbound. Control it and act on it from the phone, not just the desktop.

## Tech stack (recommended, opinionated)
- Framework: Next.js 15 (App Router, React 19, TypeScript, Server Actions).
- UI: Tailwind CSS + shadcn/ui + lucide-react + Recharts.
- DB: Postgres (Neon or Supabase) via Prisma ORM.
- Auth: NextAuth (Auth.js) with Google provider, domain-restricted to zyphe.com.
- Background jobs + scheduling: Inngest (cron + event-driven: morning cockpit, signal scan, follow-up guardian, weekly report, syncs).
- AI brain: Anthropic Claude API (Opus 5 for drafting/accuracy, Sonnet for cheap ranking/summarizing).
- Hosting: Vercel (app + serverless) + Neon/Supabase (Postgres) + Inngest cloud.
- Integrations: HubSpot, Amplemarket, Google (Calendar/Gmail/Drive), Slack, Clay, Smartlead, Granola (if API available).

## Modules (panels) — build in this order
1. Cockpit (Today): do-5-first, today's calls with prep, tasks due, follow-ups needed, live signals.
2. Pipeline: deals by stage with amount/segment/next-step and hygiene flags; write-back stage + note.
3. Reminders and tasks: scheduled + manual; snooze/complete.
4. AI actions: one-click Claude skills (call prep, draft follow-ups, signal scan, weekly report, atomize content).
5. Outreach: Amplemarket/Smartlead stats, sequences, deliverability, domains.
6. Events: upcoming events, who goes, cost, deadlines.
7. Content and social: posts per voice, engagement, newsletter status.
8. Documents: everything by topic and destination, wired to Drive/knowledge base.
9. KPIs and ARR: pacing to target, pipeline coverage, weekly targets vs actual.

## Phased roadmap (milestones)
- M1 (foundation): auth, DB, Prisma schema, app shell + nav, Cockpit + Pipeline read-only from HubSpot, Reminders CRUD.
- M2 (act): AI actions (Claude drafting), Follow-up Guardian, write-back to HubSpot (behind a flag), Signals panel.
- M3 (engine): Inngest jobs (morning cockpit, signal scan, weekly report), Outreach panel (Amplemarket/Smartlead), KPIs/ARR.
- M4 (polish): Content/social, Documents index, custom layouts, alerts, mobile.

## Success criteria
- Edoardo starts every day in the Cockpit and clears the do-5-first.
- No prospect goes >N days without a logged follow-up (Follow-up Guardian).
- Weekly report generates itself and posts to Slack.
- Deal hygiene flags drop to near zero.

## Constraints
- HubSpot write is currently blocked at the account level; ship write-back behind a feature flag and default to draft-only until edit access is granted.
- Secrets live server-side only (never in the browser bundle).
