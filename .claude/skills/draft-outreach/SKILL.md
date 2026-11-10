---
name: draft-outreach
description: Write personalised first-touch emails as Gmail drafts plus LinkedIn message copy for prospects in stage "researched". Never sends. Use when the user says "draft outreach", "write the emails", or as step 3 of an outbound run.
---

# Draft outreach (Gmail drafts + LinkedIn copy)

Follow `CLAUDE.md`. Drafts only. Never send email or LinkedIn messages.

## Inputs
`offer.md` (value prop, proof, tone, CTA), `settings.yaml` (caps, opt-out line), `pipeline.csv` rows with `stage=researched`, and `suppression.csv`.

## Email 1 rules
- Under 120 words, plain text (use the `body` field, no Markdown).
- Use the prospect's `segment` to pick the pains and CTA from `offer.md` (`regulated_enterprise`, `training_buyer` or `partner`). Skip any proof line still marked TODO.
- Structure: hook (from `personalisation_hook`, or an honest generic line if `none`) → the pain → one line of value plus one real proof point → a soft question CTA ("Worth a 20-minute chat next week?").
- Don't put the Calendly link in Email 1. It goes in only after they show interest (see `book-meetings`). This keeps first-touch emails link-light and lands them in the inbox more often.
- Add `compliance.opt_out_line`, and `physical_address` if one is set.
- Subject: 2–5 words, lowercase, no clickbait.

## Steps
1. Skip anyone in `suppression.csv`. Stop at `daily_caps.new_first_touch_drafts`.
2. For each prospect, `mcp__Gmail__create_draft` with `to=[email]`, `subject`, `body`, `includeSignature=true`. Save the returned draft `id` and `threadId` into `gmail_draft_id` and `gmail_thread_id`.
3. Write the LinkedIn copy (a connection note under 300 characters, plus a follow-up DM under 500 characters) into `outbound-agent/data/linkedin_queue.md`, under a heading per prospect with their `linkedin_url`. The user sends these by hand.
4. Set `stage=drafted`, `touch_count=1`, `next_action_at` = today + first `follow_up_schedule_days`.
5. Show a review table (name, subject, first line, hook source) and remind the user: "N drafts are waiting in Gmail → Drafts. Review and send them yourself. Then run `/outbound-run` again and I'll mark sent ones as `sent`."

The built-in `lusha:outreach-sequence` skill can be used for wording, but the output must still go to Gmail drafts and the pipeline as described here.
