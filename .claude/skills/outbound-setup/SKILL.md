---
name: outbound-setup
description: One-time setup and health check for the outbound booking agent. Verifies Lusha, Gmail and Calendly connections, picks the Calendly event type, creates Gmail labels and the pipeline files. Use when the user says "set up outbound", "check my outbound setup", or before the first outbound run.
---

# Outbound setup

Read `CLAUDE.md` rules first. This skill only reads and creates config; it sends nothing.

## Steps

1. **Config check.** Read `outbound-agent/config/icp.yaml`, `offer.md`, `settings.yaml`. If they still hold placeholder values (`<...>`, "Placeholder ICP"), list what's missing and ask the user to fill them in before going on.

2. **Calendly.**
   - `mcp__Calendly__users-get_current_user` → note the name, timezone, user URI and `scheduling_url`.
   - `mcp__Calendly__event_types-list_event_types` for that user. Match the name in `offer.md` → "Calendly event type name". If there's no match or more than one, show the active event types and ask the user to pick.
   - Write the chosen name into `settings.yaml` → `calendly.event_type_name`. Keep the URI only in memory or in notes, never shown to the user.

3. **Gmail.**
   - `mcp__Gmail__list_labels`. For each label in `settings.yaml → gmail.labels` that's missing, create it with `mcp__Gmail__create_label`.
   - Do not send a test email.

4. **Lusha.**
   - `mcp__Lusha__account_usage` → report the credit balance and the cost per search/enrich action.

5. **Data files.** If missing, copy `outbound-agent/templates/pipeline.template.csv` → `outbound-agent/data/pipeline.csv` and `suppression.template.csv` → `outbound-agent/data/suppression.csv`.

6. **Report** a short checklist: ✅/❌ per connector, chosen event type, booking page URL, Lusha credits, and anything the user must still do (e.g. add `compliance.physical_address`).

If any connector call fails, say which one and stop. Don't work around a missing connector.
