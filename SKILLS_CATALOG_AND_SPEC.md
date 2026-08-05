# Zyphe skills — catalog and spec (Cowork + command-center app)
The single source of truth for every custom Zyphe automation. Each skill is built twice: as a Cowork skill (runs in chat now) and, from this same spec, as an app module inside the command center (production). Off-the-shelf plugin skills that already cover a need are reused, not rebuilt.

## Shared rules (apply to every skill)
- Read the metric set from one place; never hardcode product numbers.
- Draft, don't auto-send: anything customer-facing or that writes to HubSpot returns a draft for approval (HubSpot edit is currently blocked).
- Domain safety: any list action validates addresses, strips .org/invalid/dupes, keeps an unsubscribe, and never sends cold from zyphe.com.
- Pull context from the knowledge base (product facts, battlecards, ICPs, value emails v3, call scripts v3, ICP research) so output stays on-message.

## Reused, not rebuilt (already available)
- zyphe-leadgen (yours) — signals/scrape/enrich/export pipeline.
- End-of-day deal reconciliation (yours) — keep, extend with hygiene rules.
- humanize-text (yours), brand-voice-enforcement — on-brand copy.
- sales:call-prep / zoominfo:meeting-prep — call prep.
- sales:call-summary — post-call actions + follow-up from a transcript.
- zoominfo:score-accounts, zoominfo:tech-stack-snapshot — scoring + technographic/competitor signals.
- sales:pipeline-review, sales:daily-briefing — generic bases the tailored skills build on.

---
## The 12 custom skills

### GTM / Sales
**1. zyphe-morning-cockpit** — Domain: GTM. Purpose: a ranked daily brief. Trigger: "morning cockpit", "start my day", or scheduled 07:00. Inputs: Google Calendar, HubSpot deals, Amplemarket replies/signals, tasks, reminders. Workflow: pull all → rank by urgency and value → output "do these 5 first" + today's calls (with prep links) + follow-ups needed + new signals. Output: a cockpit summary (chat + optional Slack DM). Success: user clears the do-5 daily. App module: `GET /api/cockpit` + Inngest `morning-cockpit` cron + CockpitPanel.

**2. zyphe-follow-up-guardian** — Domain: GTM. Purpose: never let a prospect go dark. Trigger: "follow-ups", or scheduled 18:00. Inputs: HubSpot, Gmail/Superhuman. Workflow: find deals/contacts with no touch > N days → draft the next value touch (persona/ICP-aware, from value emails v3) → queue for approval. Output: a list of drafted follow-ups. Success: zero prospects dark beyond N days. App module: Inngest `follow-up-guardian` + `Draft` table + FollowUps view.

**3. zyphe-weekly-gtm-report** — Domain: GTM. Purpose: the weekly report, automatically. Trigger: "weekly report", or Thu 09:00. Inputs: Amplemarket stats, HubSpot pipeline, activity, ARR pacing. Workflow: compile sends/opens/replies/meetings + new pipeline + stage conversion + ARR vs plan → post to Slack. Output: a branded weekly report. Success: it ships without manual work. App module: Inngest `weekly-report` + `GET /api/outreach` + slack.postMessage.

**4. zyphe-inbound-responder** — Domain: GTM. Purpose: convert inbound fast. Trigger: new message in #form-submission (or "handle inbound"). Inputs: Slack #form-submission, Clay (enrich), HubSpot. Workflow: read the lead → enrich → draft a tailored first reply + booking link → create a HubSpot deal with segment (draft). Output: a ready reply + deal draft. Success: inbound answered same day. App module: Inngest event on Slack webhook + `POST /api/actions/run`.

**5. zyphe-list-hygiene** — Domain: GTM. Purpose: protect deliverability. Trigger: "clean this list", before any send. Inputs: a lead list, validation API. Workflow: strip .org/invalid/dupes, confirm unsubscribe present, tag which domain sends → return a clean list + a removed log. Output: cleaned list + report. Success: bounce/spam under threshold, zyphe.com untouched. App module: `POST /api/lists/validate`.

### Marketing
**6. zyphe-content-atomizer** — Domain: Marketing. Purpose: one insight becomes a week of content. Trigger: "atomize this", "turn this into posts". Inputs: an insight/link/news, knowledge base, brand voice. Workflow: generate LinkedIn posts for the four voices (2-3/week each, 90-day topic, hook-first), a newsletter item, an IG carousel outline, and a Reddit answer. Output: a content pack (on-brand, humanized). Success: fills the weekly calendar. App module: `POST /api/actions/run` skill=atomize-content + ContentPanel.

**7. zyphe-newsletter-builder** — Domain: Marketing. Purpose: the weekly "Compliance Layer" issue. Trigger: "build the newsletter", or weekly. Inputs: regulatory/competitor news, knowledge base. Workflow: pick the one regulation that matters + an honest take + one link → draft the issue for Beehiiv + LinkedIn, only against a validated list. Output: a newsletter draft. Success: >30% open, >2% CTR. App module: Inngest weekly + ContentPanel.

**8. zyphe-competitor-watch** — Domain: Marketing. Purpose: turn competitor moves into content and battlecard updates. Trigger: "competitor watch", or weekly. Inputs: web/news, existing battlecards. Workflow: scan for breaches, funding, regulatory hits (Sumsub, Veriff, Persona, Jumio, etc.) → update battlecards + draft a LinkedIn hook and a displacement outreach line. Output: an update digest. Success: every major competitor event becomes a content/outreach asset within a week. App module: Inngest weekly + SignalsPanel.

### Operations
**9. zyphe-1on1-prep** — Domain: Ops. Purpose: generate a 1:1 prep doc. Trigger: "prep my 1:1 with [Michelangelo/Hendrik]". Inputs: the recurring 1:1 doc, the latest transcript, Slack, email, calendar. Workflow: read the recurring doc + latest transcript + recent Slack/email/calendar → produce updates since last time, open items, blockers, pipeline, and asks; cross-check calendar/messages before stating status. Output: a paste-in 1:1 doc. Success: accurate, no missed commitments. App module: `POST /api/actions/run` skill=one-on-one-prep.

**10. zyphe-events-tracker** — Domain: Ops. Purpose: evaluate and track events. Trigger: "compare events", "check event registrations". Inputs: the events sheet, email (rep threads), web. Workflow: pull costs/dates/deadlines → build a comparison (free/earned vs paid routes) → flag conflicts and registration deadlines → set reminders. Output: a comparison table + reminders. Success: no missed deadline; spend stays under budget. App module: `GET /api/events` + Inngest deadline scan + EventsPanel.

### Finance
**11. zyphe-arr-tracker** — Domain: Finance. Purpose: pace revenue to target. Trigger: "ARR pacing", "are we on track". Inputs: HubSpot closed/won + pipeline, the target ($8-10M). Workflow: compute ARR, net-new needed, pipeline coverage vs the ~$25-35M annual creation target, by segment. Output: a pacing snapshot with the gap. Success: leadership sees pacing weekly. App module: `GET /api/kpis` + KpiPanel.

**12. zyphe-competition-pack** — Domain: Finance/Ops. Purpose: assemble a startup-competition/application pack. Trigger: "prep [competition] application". Inputs: business plan, financials, pitch deck, video, the Battlefield/Nexus/MY START BCN requirements. Workflow: map each requirement to the right asset, flag gaps (video, financials), keep one consistent metric set and traction figures. Output: a filled application + a punch-list of gaps. Success: applications submitted on time, no fabricated figures. App module: `POST /api/actions/run` skill=competition-pack.

## Build status
Cowork skills: built in this session (kebab-case names above). App modules: to be built by Claude Code from this spec (see the endpoint/job/component mapping per skill).
