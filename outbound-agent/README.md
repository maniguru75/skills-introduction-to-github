# Outbound Booking Agent (Claude + Lusha + Gmail + Calendly)

A Claude Code project that runs an outbound sales workflow from one chat:
find ICP prospects → spot buying signals → research → write personalised emails and LinkedIn copy → track follow-ups → triage replies → book meetings in Calendly.

**Analogy:** think of Claude as an SDR who sits at your desk. Lusha is their prospect database, Gmail is their outbox, Calendly is their diary. The difference is that this SDR **never hits send**. They leave every email in your Drafts folder for you to approve.

## How it maps to the LinkedIn post

| Post claim | Skill | Tools used |
|---|---|---|
| Finds people who fit your ICP | `find-prospects` | Lusha prospecting search + filters |
| Detects buying signals | `find-prospects` | Lusha signals / signal score |
| Researches companies & decision-makers | `research-prospects` | Lusha company/contact signals, optional web search |
| Finds & enriches contact info | `find-prospects` (step 7) | Lusha enrich (after your approval) |
| Personalised messages | `draft-outreach` | Gmail drafts + LinkedIn copy file |
| Launches outreach & manages follow-ups | `draft-outreach`, `follow-ups` | Gmail drafts, threaded |
| Monitors replies, finds interested prospects | `triage-replies` | Gmail search/threads/labels |
| Qualifies & books meetings | `triage-replies`, `book-meetings` | Calendly availability, single-use links, booking |
| "One interface" | `outbound-run` | runs all of the above in order |

What's deliberately different from the post: **human approval before sending** (protects your email domain and keeps you compliant), and **no automated LinkedIn sending** (it breaks LinkedIn's terms of service). The "15,000+ conversations" figure is the post author's claim. I can't verify it, and you shouldn't expect it at first. Start small.

## Prerequisites
1. Claude Code (CLI, desktop or claude.ai/code) with these connectors connected on your account: **Lusha**, **Gmail**, **Calendly**.
2. A Lusha plan with credits. Searches and enrichment cost credits; the agent shows the cost before spending.
3. A Calendly event type for the meeting (e.g. "Intro Call", 20 minutes).
4. Preferably a **separate sending domain/mailbox** that's been warmed up. Cold email from your main domain risks your deliverability.

## Quick start
```text
1. Edit outbound-agent/config/icp.yaml      (who you're targeting)
2. Edit outbound-agent/config/offer.md      (what you sell, proof, Calendly event name)
3. Check outbound-agent/config/settings.yaml (caps, cadence, add your business address)
4. In Claude Code, from the repo root:  /outbound-setup
5. Then daily:                          /outbound-run
6. Review and send drafts in Gmail; send LinkedIn notes from outbound-agent/data/linkedin_queue.md
```
You can also run any stage alone: `/find-prospects`, `/research-prospects`, `/draft-outreach`, `/triage-replies`, `/book-meetings`, `/follow-ups`.

## Files
```
CLAUDE.md                         guardrails Claude follows every run
.claude/skills/*/SKILL.md         one skill per pipeline stage + orchestrator
outbound-agent/config/            your ICP, offer, settings (edit these)
outbound-agent/templates/         blank CSV templates
outbound-agent/data/              pipeline.csv, suppression.csv, linkedin_queue.md (git-ignored: personal data)
```

## Pipeline stages
`sourced → researched → drafted → sent → replied → interested → link_sent → booked`
plus `not_now`, `objection`, `opted_out`, `bounced`, `closed_lost`.

## Safety and compliance (please read)
- Cold email laws differ by region (e.g. CAN-SPAM in the US, GDPR/PECR in the UK/EU). I'm not a lawyer, so please check what applies to you. The templates include an opt-out line and a slot for your physical address.
- Opt-outs go into `suppression.csv` and are never contacted again.
- Personalisation must come from real, sourced facts. The agent records where each hook came from.
- Keep daily volumes low at first (the defaults are 25 new / 40 follow-ups per day). Those defaults are my own conservative starting point, not an official limit, so adjust them to your mailbox's reputation.

## Going further (optional)
- **Scheduled runs:** Claude Code's `/loop` or a scheduled task can run `/outbound-run` each morning. Drafts still wait for you.
- **CRM sync:** Lusha has `export_contacts_to_crm` if you connect a CRM.
- **Full auto-send:** possible by switching the Gmail calls to `send_message`/`reply`, but I'd only do it after a few weeks of reviewed drafts and with strict caps.
