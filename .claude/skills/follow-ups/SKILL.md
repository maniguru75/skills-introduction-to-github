---
name: follow-ups
description: Draft threaded follow-up emails for prospects who haven't replied, following the cadence in settings.yaml, and stop after max_touches. Use when the user says "do follow-ups", "chase non-responders", or as part of an outbound run.
---

# Follow-ups

Follow `CLAUDE.md`. Drafts only.

## Steps
1. Run `triage-replies` first (or confirm it ran today), so nobody who has replied gets a follow-up.
2. Pick rows with `stage=sent`, `next_action_at <= today`, `touch_count < max_touches`, and not in `suppression.csv`. Cap at `daily_caps.follow_up_drafts`.
3. For each one, `mcp__Gmail__get_thread` on `gmail_thread_id`. Confirm there's no reply from the prospect, then take the latest message id.
4. Draft with `mcp__Gmail__create_draft` and `replyToMessageId` (it stays in the same thread):
   - **Touch 2**: 2–3 lines with a new angle (a different pain or proof point from `offer.md`). Don't write "just bumping this".
   - **Touch 3 (break-up)**: 2 lines, permission to close the loop: "Should I close the file on this, or is <pain> worth a look later this quarter?"
   - Include the opt-out line.
5. Update `touch_count` and `next_action_at` (next interval, or none after the last touch). After the last touch, with no reply after the final interval, set `stage=closed_lost`.
6. Report the number of drafts created and remind the user to review and send them in Gmail.
