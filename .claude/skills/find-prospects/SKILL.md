---
name: find-prospects
description: Find people who fit the ICP and detect buying signals using Lusha, then enrich only the approved shortlist. Writes new rows to the outbound pipeline. Use when the user says "find prospects", "build a list", "who should I reach out to", or as step 1 of an outbound run.
---

# Find prospects (ICP + buying signals)

Follow `CLAUDE.md`. Searches and enrichment spend Lusha credits, so always get approval first.

## Steps

1. Load `outbound-agent/config/icp.yaml` and `settings.yaml`. Load `outbound-agent/data/pipeline.csv` and `suppression.csv` so you skip anyone already contacted or opted out.
   - The ICP has **segments**. Process only the ones in `segments_per_run`, using countries from `geo.<active_phase>`. Run steps 2–8 once per segment and write the segment `id` into the `segment` column.
   - **Geo demand check (optional, on request or on the first run):** with approval, run a small company search per country in `geo.phase_1` + `phase_2`, combined with GRC/security/risk hiring signals, and report counts per country so the user can choose `active_phase` from data rather than assumption.

2. **Resolve filters (free).** Call `mcp__Lusha__prospecting_company_filters` and `mcp__Lusha__prospecting_contact_filters`, one filter type per call (e.g. `industries_labels`, `sizes`, `seniority`, locations), to turn the ICP text into valid ids. Check each tool's schema for the exact filter type names; don't guess them.

3. **Cost preview.** Call `mcp__Lusha__account_usage`. Tell the user: "Searching ~N companies and up to `batch_size` contacts will cost roughly X credits. Proceed?" Wait for a yes.

4. **Companies with signals.**
   - `mcp__Lusha__prospecting_company_search` with the ICP company filters.
   - If `buying_signals` are set, check signal type names via `mcp__Lusha__signals_company_filters`, then use `mcp__Lusha__signals_companies_search` or `mcp__Lusha__signal_score_companies` to rank them. Optionally use `mcp__Lusha__icp_fit_score_companies`.
   - Drop `exclude_domains` and suppressed domains.

5. **Decision makers.** For the top companies, use `mcp__Lusha__decision_makers_buying_group_search` (by domain or Lusha company id) or `mcp__Lusha__prospecting_contact_search` with `companyIds` plus the title/seniority filters. Remember that prospecting search pages start at 0.

6. **Shortlist for approval.** Show a table (name, title, company, size, signal, why they fit), capped at `batch_size`. Ask the user which rows to enrich. Do not reveal contact details yet.

7. **Enrich approved rows only.** `mcp__Lusha__prospecting_contact_enrich` with the chosen ids, respecting `daily_caps.lusha_enrich_per_run`. Keep work email and LinkedIn URL. Skip anyone without a work email (mark them `linkedin_only` in notes).

8. **Write to pipeline.** Append rows with `stage=sourced`, `signal`, `signal_source` (e.g. "Lusha signal: funding, <date>"), and the Lusha ids. Pass the `conversation_id` from Lusha responses back on related calls.

9. Report: how many were found, shortlisted, enriched and added, plus credits used.

Alternative: the built-in `lusha:prospect` / `lusha:signal-prospect` skills cover the same ground. Using them is fine, as long as the results land in `pipeline.csv`.
