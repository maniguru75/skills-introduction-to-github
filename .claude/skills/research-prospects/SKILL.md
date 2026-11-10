---
name: research-prospects
description: Research target companies and decision-makers for prospects in stage "sourced" and record one real, sourced personalisation hook per prospect. Use when the user says "research these prospects", "research my pipeline", or as step 2 of an outbound run.
---

# Research prospects

Follow `CLAUDE.md`, rule 6 (no fabrication) above all.

## For each row with `stage=sourced`

1. **Company context.** `mcp__Lusha__prospecting_company_enrich` (if not already done) or `mcp__Lusha__companies_search` by domain, for size, industry, description and tech where present. `mcp__Lusha__signals_companies_get` gives recent signals.
2. **Person context.** `mcp__Lusha__signals_contacts_get` for job changes or promotions.
3. **Web (optional).** If a web search tool is available in the session, look for one recent, relevant item (a news story, product launch, job post). Only use what you actually read.
4. **Write into the row:**
   - `research_summary`: 2–3 factual lines about what they do and what's changed.
   - `personalisation_hook`: one sentence tying a real fact to a pain from `offer.md`.
   - `hook_source`: where the fact came from (e.g. "Lusha signal: hiring 4 AEs, 2026-10" or a URL).
   - If there's no real hook, set the hook to `none` and note "use generic opener".
   - `stage=researched`, `last_action_at=<now>`.

Work in batches of 10 and show a short summary table after each batch.
