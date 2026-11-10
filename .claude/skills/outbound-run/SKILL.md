---
name: outbound-run
description: Run the full outbound workflow end to end (triage replies, book meetings, follow-ups, find new prospects, research, draft) and print a daily summary. Use when the user says "run outbound", "daily outbound", "/outbound-run", or gives a target audience and asks Claude to handle the outreach.
---

# Outbound run (orchestrator)

Follow `CLAUDE.md`. Run the stages in this order. Inbound work comes first, because a reply is worth more than a new lead.

1. **Health check**: if `outbound-agent/data/pipeline.csv` doesn't exist or the config holds placeholders, run `outbound-setup` and stop.
2. **triage-replies**: sync sent status, classify replies, apply labels, handle opt-outs.
3. **book-meetings**: interested → slots + single-use link draft; detect self-booked meetings.
4. **follow-ups**: threaded follow-up drafts for non-responders that are due.
5. **find-prospects**: only if today's new first-touch cap isn't reached yet. It asks before spending Lusha credits.
6. **research-prospects**: on the new rows.
7. **draft-outreach**: first-touch drafts plus LinkedIn copy.

If the user gave a target audience in the request (e.g. "RevOps leaders at 50–200 person SaaS in the UK"), show the proposed changes to `icp.yaml` for this run and confirm them before step 5.

## Daily summary (end of run)
```
Outbound run: <date>
Replies:     X interested · X not now · X objections · X opted out
Meetings:    X booked (list with local times) · X links waiting
Drafts:      X first-touch · X follow-ups · X objection replies → review in Gmail Drafts
LinkedIn:    X messages in outbound-agent/data/linkedin_queue.md
Lusha:       X credits used · X remaining
Pipeline:    sourced X · researched X · drafted X · sent X · interested X · booked X
Your to-dos: send the drafts, send the LinkedIn notes, prep for the meetings
```
