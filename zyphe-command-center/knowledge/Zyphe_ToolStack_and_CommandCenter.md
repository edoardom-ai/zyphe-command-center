# Zyphe — best-in-class tool stack, automations, and the command-center question
Companion to the Operating System doc. Researched fresh for 2026, since this space changes monthly. The rule throughout: pick a lean spine and add a few best-in-class specialists, do not drown a 3-person team in tools.

The spine: **Claude** for anything accuracy-critical and for building the automations (it hallucinates far less than the others on long-form factuality, which matters when the copy is about compliance), **Amplemarket** stays the outbound brain you already pay for, and **Google Workspace/Gemini** is already in your stack (the Meet notetaker). Everything else below plugs into those.

## 1. Which AI model for which job (2026)
| Job | Use | Why |
|---|---|---|
| Customer-facing copy, research summaries, proposals, on-brand drafting | **Claude (Opus 5 / Claude Code)** | Lowest hallucination on long-form factuality (~36% vs GPT-5.5 ~86% in tests). Precision over speed. This is your workhorse. |
| Huge-context analysis (read every transcript/doc at once), multimodal (video, image, carousels) | **Gemini 3 Pro** | Biggest context window, native multimodal, cheapest top-tier reasoner, and native inside Google Docs/Gmail/Meet where you already work. |
| Autonomous multi-step agents, ChatGPT-ecosystem tasks | **GPT-5.5 / Codex** | Strongest autonomous agent; Codex leads Terminal-Bench and has the Windows sandbox story. |
| Building the automations and internal tools | **Claude Code** (top capability), **Antigravity** (free multi-agent IDE, Gemini-powered) for experiments, **OpenCode** if you want model-agnostic/self-host | Claude Code is #1 on SWE-bench; Antigravity is free in preview; this is mostly Manuel's call. |

Practical rule: Claude drafts anything that touches a customer or a regulator; Gemini chews through big document piles and makes visual content; GPT/Codex when you want a fully autonomous agent. Don't pay for three seats per person.

## 2. The automation stack, tool by tool (mapped to your Operating System)
| Automation (Part A) | Best tool(s) | Model | Rough cost |
|---|---|---|---|
| Second brain / knowledge base | Claude (Cowork + Projects) as the brain, **Notion** as the doc store, wired over MCP | Claude | Notion ~$10/user |
| Morning Cockpit, Call-Prep, Post-Call, Follow-up Guardian, Weekly Report | **Cowork Claude skills + scheduled tasks** (native), pulling HubSpot / Calendar / Granola / Amplemarket; orchestrate heavier flows in **n8n** or **Lindy** | Claude | included / n8n self-host cheap |
| Signal-to-Outreach engine (the multiplier) | **Clay** (waterfall enrichment across 150+ sources, Claygent AI research, tracks job changes, funding, news, tech signals) feeding Amplemarket; Claude drafts the 1:1 | Claude + Claygent | Clay ~$150+/mo |
| Cold-call engine | **Nooks** (parallel dialer + live AI coaching + virtual floor, good because you and Hendrik call together) or **Orum** (pure high-volume throughput, ~5x more conversations) | n/a | custom pricing |
| Outbound sending | **Amplemarket** (brain, signals) + **Smartlead** (best-in-class deliverability, unlimited inboxes, private IP rotation) on separate warmed domains | n/a | Smartlead ~$39+/mo |
| Content Atomizer | **Pressmaster.ai** (Hendrik's) + Claude/Gemini; schedule via Taplio + Buffer | Claude/Gemini | Pressmaster via Hendrik |
| Domain-safety gate | Email validation (**NeverBounce / ZeroBounce / Hunter**) + separate domains in Smartlead + enforced unsubscribe | n/a | ~$20-50/mo |
| Inbound Responder | **Lindy** or **n8n** watching #form-submission → enrich (Clay) → draft (Claude) → HubSpot deal | Claude | Lindy ~$50/mo |
| AI-SDR (aggressive scale option) | **Artisan (Ava)** or **Regie.ai** run end-to-end outbound (find accounts, message, handle replies, book) | built-in | $$$ (evaluate later) |

Note on the deliverability crackdown: Google and Microsoft tightened bulk-sender rules in early 2026 and shared-IP tools saw 30-50% deliverability drops. This is exactly why the Smartlead-on-separate-domains setup matters and why Charlene's domain-protection rule is now non-negotiable.

## 3. Channel tools
- **LinkedIn:** **Taplio** (all-in-one: AI writing, scheduling, viral-post library, light CRM) for the team, plus **AuthoredUp** (best-in-class formatting, hooks, analytics, ~$19) for the writing itself. Both are built for the 2-3x/week personal-voice cadence.
- **Instagram / X:** **Buffer** (simple, cheap, AI assistant) or **SocialPilot** (AI Pilot, all platforms); feed them from Pressmaster.
- **Newsletter:** **Beehiiv** over Substack. You keep 100% of revenue, get real growth tooling, automation, and the Boosts network; Substack takes 10% and is just a writing tool.
- **Reddit / community monitoring:** keyword alerts via **F5Bot** or **Syften** so you never miss an age-verification/KYC thread to answer.
- **SEO / content:** keep Nibby and Fausto; equip them with **Ahrefs** (keywords) and a brief tool like **Clearscope/Frase**.

## 4. The LinkedIn / X / Instagram growth playbook (2026)
Straight from current best practice, this operationalizes the marketing strategy's LinkedIn tier:
- **Founder-led is the B2B motion now.** Company handles lose; the winners drive inbound through founders' personal accounts. All four of you, not the Zyphe page.
- **Each person owns ONE narrow topic for 90 days** so the algorithm and the audience learn what you're for (e.g., you on age-verification/DSA, Michelangelo on agentic compliance and liability, Manuel on the privacy architecture).
- **Formats that win:** carousels (top engagement, ~46%) and native video under 90 seconds with a pattern-interrupt hook in the first 3 seconds.
- **Spend more time on the hook than the body.** The "see more" click is a ranking signal.
- **First 60 minutes decide reach:** the team comments and reshares each other, and you reply to every comment in the first hour.
- **Consistency beats virality:** 2-3 quality posts a week, predictably, outperforms one big hit.

## 5. The command-center / dashboard question: yes, build one
Short answer: yes, a personal GTM command center is genuinely worth it, and it directly serves both goals (reliability and growth). Build it in two layers so you get value now and power later.

**Layer 1, the knowledge hub (docs by topic and destination + the second brain).** Use **Notion** (or a well-structured Drive) as the organized home for every doc, tagged by topic and destination, wired to Claude so any draft pulls from it. This is where "all the documents organized" lives.

**Layer 2, the live dashboard (KPIs, pipeline, events, and control of Amplemarket/HubSpot/Claude).** Three-step path:
- **MVP now:** a **Cowork live dashboard** I can build for you as a persistent artifact that reads your connectors (calendar, HubSpot deals, Amplemarket stats, tasks, events) and shows your KPIs on one page, reopenable and refreshing. Zero new tools, I can start this today.
- **v1, no-code:** **Airtable Interfaces** as the structured hub (docs index, KPIs, events, pipeline) plus **n8n** to sync and to trigger Amplemarket/HubSpot actions from it. You can run this yourself.
- **v2, production control panel:** **Retool** is the right tool if you want a true command center that connects to HubSpot, Amplemarket, and Claude over their APIs with write-back control (move a deal, launch a sequence, kick off a Claude task from one screen). This is a Manuel build.

Recommended combo: **Notion for docs + Retool (or Airtable to start) for the live control panel**, with the Cowork artifact as the immediate MVP. Notion alone is not a live dashboard, and a spreadsheet alone won't "control" your other tools, which is why the two-layer split matters.

## 6. Recommended consolidated stack and rough monthly cost
Lean, prioritized for the outbound-first motion:
- Have already: Claude/Cowork, Amplemarket, Google Workspace/Gemini, Granola, Pressmaster (via Hendrik).
- Add first (outbound is the primary motion): **Smartlead** (~$39+), **Clay** (~$150+), a dialer **Nooks or Orum** (custom), email validation (~$20-50).
- Add for brand/inbound: **Taplio** (~$39-65) + **AuthoredUp** (~$19), **Buffer/SocialPilot** (~$20-30), **Beehiiv** (free to ~$40).
- Add for automation/hub: **n8n** (self-host, cheap) or **Lindy** (~$50), **Notion** (~$10), **Airtable** (~$20) or **Retool** (~$10-50/user).
Lean total is roughly **$400-700/month plus the dialer**, which is trivial against one closed mid-market deal.

## 7. What NOT to buy yet
- Enterprise GTM suites (Gong, Outreach, 6sense, Salesloft, Clari). Overkill and expensive for three people; revisit after a raise.
- Three LLM subscriptions per person. Keep the Claude spine plus Gemini in Workspace.
- An AI-SDR (Artisan/Regie) on day one. Prove the human motion first, then let an AI-SDR scale the part that works.

---
## Sources
- AI coding agents: [SSOJet comparison](https://ssojet.com/blog/ai-coding-agents-compared), [Firecrawl](https://www.firecrawl.dev/blog/best-ai-coding-agents)
- Model choice: [Fenxi: Opus 4.8 vs GPT-5.5 vs Gemini 3.1](https://fenxi.fr/en/blog/claude-opus-4-8-vs-gpt-5-5-vs-gemini-3-1-pro-business-2026/), [Vellum flagship report](https://www.vellum.ai/blog/flagship-model-report)
- Cold email / deliverability: [Hunter.io](https://hunter.io/blog/cold-email-software/), [Amplemarket](https://www.amplemarket.com/blog/best-cold-email-software-2026)
- Clay / enrichment: [Salesdorado Clay review](https://salesdorado.com/en/lead-generation/review-clay/), [HeyReach GTM stack](https://www.heyreach.io/blog/best-gtm-tools)
- Dialers: [CloudTalk parallel dialers](https://www.cloudtalk.io/blog/best-parallel-dialer/), [Nooks](https://www.nooks.ai/blog-posts/cold-calling-software-boosting-sales-one-call-at-a-time)
- Automation platforms: [n8n blog](https://blog.n8n.io/best-ai-workflow-automation-tools/), [Lindy vs Zapier vs Gumloop](https://www.lindy.ai/blog/gumloop-vs-zapier)
- LinkedIn tools: [Kleo Taplio alternatives](https://www.kleo.so/blog/taplio-alternatives), [AuthoredUp creator tools](https://authoredup.com/blog/linkedin-creator-tools)
- Social schedulers: [Zapier best social tools](https://zapier.com/blog/best-social-media-management-tools/)
- Newsletter: [Beehiiv vs Substack](https://whop.com/blog/beehiiv-vs-substack/)
- GTM/AI sales tools: [ZoomInfo AI GTM tools](https://pipeline.zoominfo.com/sales/ai-gtm-tools), [Mutiny buyer's guide](https://www.mutinyhq.com/blog/the-best-ai-sales-tools-in-2026-a-buyer-s-guide-for-b2b-gtm-teams)
- LinkedIn playbook: [Workflows.io founder-led marketing](https://www.workflows.io/blog/founder-led-marketing), [getviews LinkedIn growth guide](https://getviews.ai/blog/complete-linkedin-growth-guide-2026)
- Dashboard builders: [Retoolers Notion vs Airtable vs Retool](https://retoolers.io/blog-posts/notion-vs-airtable-vs-retool-which-tool-powers-internal-teams-best), [monday business app builders](https://monday.com/blog/vibe-coding/business-app-builder-2026/)
