---
name: triage-replies
description: Check Gmail for replies from pipeline prospects, classify intent (interested, not now, objection, opt-out, out-of-office, bounce), label threads and update the pipeline. Use when the user says "check replies", "who replied", or as part of an outbound run.
---

# Triage replies

Follow `CLAUDE.md`.

## Steps

1. **Sync sent status.** For rows with `stage=drafted`, `mcp__Gmail__search_threads` with `in:sent to:<email>`. If found, set `stage=sent` and record `gmail_thread_id`. If the draft still exists and isn't sent, leave it alone.

2. **Find replies.** For rows in `sent` (or later, without a final stage), search `from:<email> newer_than:30d`. Search results only preview the oldest ~5 messages, so always call `mcp__Gmail__get_thread` to read the latest message.

3. **Classify** the latest prospect message:
   | Class | Signals | New stage | Gmail label |
   |---|---|---|---|
   | interested | asks for time, "sure", "let's talk", proposes a slot | `interested` | Outbound/Interested |
   | not_now | "next quarter", "reach out in…" | `not_now` (set `next_action_at`) | Outbound/Not-Now |
   | objection | "we use X", "no budget", "send info" | `objection` | Outbound/Objection |
   | opted_out | "unsubscribe", "stop", "not interested", "remove me" | `opted_out` + add to suppression.csv | Outbound/Opted-Out |
   | ooo | auto-reply, out of office | keep stage, push `next_action_at` | — |
   | bounce | mailer-daemon / delivery failure | `bounced` | — |
   If you're not sure, ask the user. Don't guess.

4. **Qualify the interested ones.** From the thread plus research, note in `notes`: role fit, stated pain, timing, and whether they're the decision maker. Flag weak fits for the user.

5. Apply labels with `mcp__Gmail__label_thread`. Look up label ids with `mcp__Gmail__list_labels`.

6. For `objection`, write a suggested reply as a threaded draft (`mcp__Gmail__create_draft` with `replyToMessageId` = the latest message id), using `offer.md → Objection notes`. For `interested`, hand off to `book-meetings`.

7. Report the counts per class and list the interested prospects.
