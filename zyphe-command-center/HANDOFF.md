# Handoff — how to start building the command center with Claude Code

## Step 1 — set up the repo
1. Create an empty folder / new git repo, e.g. `zyphe-command-center`.
2. Copy the entire `command-center/` package into it as the project root. Required files:
   - README.md (the PRD, now includes the mobile/PWA requirement)
   - CLAUDE.md (instructions for the build agent)
   - ARCHITECTURE.md (system design, integrations, endpoints, jobs)
   - SKILLS_CATALOG_AND_SPEC.md (the 12 skills → app-module mapping)
   - prisma/schema.prisma (data model)
   - .env.example (env vars)
   - package.json
3. Add a `knowledge/` folder and drop in the context the AI skills need (see Step 3).

## Step 2 — open in Claude Code and paste this prompt
> Read CLAUDE.md, README.md, ARCHITECTURE.md, and SKILLS_CATALOG_AND_SPEC.md in full. Then scaffold milestone M1 only: Next.js 15 + Tailwind + shadcn + Prisma + NextAuth (Google, restricted to @zyphe.com), the schema, seed data, the app shell with sidebar nav and the KPI bar, and the Cockpit + Pipeline + Reminders panels running on seeded data with MOCK_INTEGRATIONS=true. Do not build later milestones yet. Show me the plan before writing code.

Then proceed M2 → M3 → M4 as each milestone passes `pnpm build` and `pnpm test`.

## Step 3 — which documents to give it (two buckets)

Bucket A — the build package (required, they define what to build):
- command-center/README.md, CLAUDE.md, ARCHITECTURE.md, SKILLS_CATALOG_AND_SPEC.md, prisma/schema.prisma, .env.example, package.json

Bucket B — the knowledge base (so the app's AI skills stay on-message). Put these in `knowledge/` and have the seed load them into the KnowledgeDoc + MetricSet tables:
- Zyphe_Marketing_Strategy_Hendrik.md (strategy)
- Edoardo_OS_and_Growth_Strategy.md (the operating model + revenue math)
- Zyphe_ToolStack_and_CommandCenter.md (the tool stack)
- Hendrik_Market_Analysis_ICPs_v2.md (ICPs)
- Zyphe_ValueBased_Email_v3_EN.md + _IT (value emails)
- Zyphe_Call_Scripts_v3_EN.md (call scripts)
- Amplemarket copies / battlecards (messaging + competitor facts)
- The one locked metric set (fill MetricSet once decided with Michelangelo)

## Step 4 — secrets and connectors
- Copy `.env.example` to `.env.local`. Start with `MOCK_INTEGRATIONS=true` so the whole UI runs with no keys.
- Then add keys one integration at a time (HubSpot read first, then Google Calendar, Amplemarket, Slack, Clay, Smartlead).
- Keep `FEATURE_WRITEBACK=false` until HubSpot edit access is granted; actions stay draft-only until then.

## Step 5 — deploy
- App: Vercel. DB: Neon or Supabase (Postgres). Jobs: Inngest cloud.
- Enable the PWA (installable + web push) so it works from the phone.

## Order of operations, in one line
Repo + package → Claude Code builds M1 on mock data → add real keys per integration → M2 (AI actions + write-back flag) → M3 (Inngest jobs + outreach/KPIs) → M4 (content/docs + PWA/push) → deploy.

## Two decisions to make before or during the build
- Lock the single metric set (fixes the app-wide MetricSet).
- Get HubSpot edit access so write-back can be turned on (until then, draft-only is fine).

## Run and build in the cloud, not on your laptop
Do not build or run this on your computer. node_modules and the Next.js dev server are heavy; keep everything in the cloud.
1. Create an empty GitHub repo (no README, no .gitignore).
2. Push this repo:
   - From the zip: `cd zyphe-command-center && git remote add origin <your-repo-url> && git push -u origin main`
   - Or from the bundle (no unzip): `git clone zyphe-command-center.bundle zyphe-command-center && cd zyphe-command-center && git remote add origin <your-repo-url> && git push -u origin main`
3. Open it in GitHub Codespaces (Code -> Codespaces -> Create), or any cloud dev environment, and run Claude Code there. All dependencies and the running dev server live in the cloud, not on your machine.
4. Connect the GitHub repo to Vercel for preview deploys, so you can open the app from any device including your phone (PWA).
Only the tiny spec repo ever needs to touch your computer, and even that is optional if you build straight in Codespaces.
