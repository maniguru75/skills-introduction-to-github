# Outbound Booking Agent — project rules for Claude

This repo turns Claude Code into an outbound sales assistant:
find ICP prospects → research → draft personalised outreach → triage replies → book meetings in Calendly.

Connected tools it relies on (MCP connectors on the user's account):
- **Lusha** — prospecting, enrichment, buying signals (`mcp__Lusha__*`)
- **Gmail** — drafts, thread search, labels (`mcp__Gmail__*`)
- **Calendly** — event types, available times, single-use links, booking (`mcp__Calendly__*`)

Skills live in `.claude/skills/`. Config lives in `outbound-agent/config/`. Working data lives in `outbound-agent/data/` (git-ignored, contains personal data).

## Hard rules (never break these)

1. **Drafts only.** Never call `mcp__Gmail__send_message`, `mcp__Gmail__reply` or `mcp__Gmail__forward`. Use `mcp__Gmail__create_draft` (with `replyToMessageId` for threaded replies). The user reviews and sends.
2. **Never send LinkedIn messages.** Only write LinkedIn copy into the review file; the user sends it by hand.
3. **Spend Lusha credits only with approval.** Searches and enrichment cost credits. Run `mcp__Lusha__account_usage` first, show the expected cost, and wait for a "yes" before any enrich call.
4. **Book in Calendly only on explicit confirmation.** `mcp__Calendly__meetings-create_invitee` is allowed only when the prospect has picked a specific slot in writing AND the user confirms. The default is to send a single-use booking link.
5. **Respect opt-outs.** Anyone who says stop / unsubscribe / not interested goes into `outbound-agent/data/suppression.csv` and is never contacted again. Check that file before drafting.
6. **No fabrication.** Personalisation must come from a source the agent actually retrieved (a Lusha signal, the company website, a news item). Record the source in the pipeline. If you have nothing real, write a generic but honest opener. Never invent mutual connections, metrics or customer names.
7. **Respect daily caps** in `outbound-agent/config/settings.yaml`.
8. **Keep private details private.** `outbound-agent/config/sender.private.md` is git-ignored. Never copy its contents into committed files, commit messages or PRs.
9. **ISACA wording.** The user is an independent trainer, not an accredited ISACA partner. Write "exam preparation for CISA/CISM…" and never "official", "accredited" or "authorised" ISACA training.
10. Show every time in the host's Calendly timezone with an inline abbreviation (e.g. "10:00 AM IST"). Never show `api.calendly.com` URIs to the user; only show `booking_url` / `scheduling_url`.

## State

`outbound-agent/data/pipeline.csv` is the single source of truth. Every skill reads it and updates the `stage`, `last_action_at` and `notes` columns. Create it from `outbound-agent/templates/pipeline.template.csv` if it is missing.

Stages: `sourced → researched → drafted → sent → replied → interested → link_sent → booked` (plus `not_now`, `objection`, `opted_out`, `bounced`, `closed_lost`).
