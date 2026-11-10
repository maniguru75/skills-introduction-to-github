---
name: book-meetings
description: Turn interested replies into booked Calendly meetings. Creates a single-use Calendly link, offers real open slots, drafts the threaded reply, and books directly only on explicit confirmation. Use when the user says "book meetings", "send booking links", "schedule with interested prospects", or after triage-replies finds interested prospects.
---

# Book meetings (the Calendly step)

Follow `CLAUDE.md`, rules 1, 4 and 8 above all.

## Setup per run
- `mcp__Calendly__users-get_current_user` → host timezone and name.
- `mcp__Calendly__event_types-list_event_types` → find the URI of `settings.yaml → calendly.event_type_name`.

## For each row with `stage=interested`

1. **Real availability.** `mcp__Calendly__event_types-list_event_type_available_times` with the event type URI, `start_time` = now (must be in the future), `end_time` = now + `look_ahead_days` (31 days max). Pick `slots_to_offer` spread across different days. Convert them to the host timezone for display, with an inline abbreviation.

2. **Single-use link.** `mcp__Calendly__scheduling_links-create_single_use_scheduling_link` with `max_event_count=1`, `owner=<event type URI>`, `owner_type="EventType"`. Save the returned `booking_url` in the pipeline. Make one link per prospect only. Reuse the stored link if the row already has one.

3. **Draft the reply** in the same thread: `mcp__Gmail__get_thread` → latest message id → `mcp__Gmail__create_draft` with `replyToMessageId`. Body (plain text, under 80 words):
   - Thank them and refer to what they said.
   - "Here are a few times that work on my side: <slot 1>, <slot 2>, <slot 3>."
   - "Or grab whatever suits you here: <booking_url>"
   - Set `stage=link_sent`.

4. **Direct booking (only when they named a time).** If the prospect's message clearly picks one of the offered slots:
   - Re-check that the slot is still in `list_event_type_available_times`.
   - Show the user: prospect name, email, slot in local time. Ask "Book this in Calendly? (yes/no)".
   - Only on "yes": `mcp__Calendly__meetings-create_invitee` with `event_type`, the UTC `start_time` exactly as returned, and `invitee` {name, email, timezone (ask the user or infer from the thread; if unsure, ask)}. Include `location` if the event type requires one (see `mcp__Calendly__locations-list_user_meeting_locations`). The host is the connected user; never swap host and invitee.
   - Set `stage=booked` and `meeting_time`, and label the thread Outbound/Booked.

## Detect self-booked meetings
`mcp__Calendly__meetings-list_events` (host user, upcoming), then `mcp__Calendly__meetings-list_event_invitees` for each event. Match invitee emails against `link_sent` rows. A match means `stage=booked`, `meeting_time` is set, and the thread gets the Booked label.

## Report
A table of booked meetings (name, company, time with timezone) and the link_sent rows still waiting.
