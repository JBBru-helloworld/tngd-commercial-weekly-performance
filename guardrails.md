# Guardrails

## AI Constraints and Hard Rules — Weekly Performance Snapshot

These rules are non-negotiable. They apply in every session, every week, without exception. If a guardrail is triggered, resolve it before delivering the report.

---

## G1 — Data Integrity

**G1.1 — Never fabricate numbers.**
If a metric cannot be computed from the uploaded data, state this explicitly. Do not estimate, round up, or fill in from memory. Write: _"[Metric] not available — [reason]."_

**G1.2 — Never assume zero for blanks.**
A blank revenue cell ≠ zero revenue. Blanks must be flagged to the user during data validation. Do not silently treat missing values as zero.

**G1.3 — Never carry over numbers from prior sessions.**
You have no memory across sessions. Every session begins fresh. Do not reference or reuse figures from a previous report unless they appear in the currently uploaded file.

**G1.4 — Never hallucinate trends.**
A trend requires data points. If the data shows only 1 week, there is no trend. Do not use trend language ("continuing to grow", "sustained decline") without at least 2 data points to support it.

---

## G2 — Schema and Scope

**G2.1 — Do not hardcode column names.**
The schema is adaptive. Always infer column roles from headers. If column names change, re-map — do not fail silently or map to a stale assumption.

**G2.2 — Category Deep Dive is L2 "04" scope only.**
Never include other L2 categories in Section 3. If no "04 Category Management" rows exist in the data, flag this immediately — do not substitute another L2 category.

**G2.3 — Never mention merchant names in L3 commentary.**
L3 is for category-level narrative only. Merchant names belong exclusively in L4 tables and bucket lists. This is a structural rule, not a style preference.

**G2.4 — Noise filter must be applied and documented.**
Do not generate Section 3 without first applying the noise filter and stating the threshold used. An undocumented threshold is a guardrail violation.

**G2.5 — Always use merchant_group as the merchant identifier.**
The source data contains a field called merchant_group which is the standardised and consistent naming for all merchants across weeks. Always use merchant_group as the merchant name displayed in all tables, bucket lists, and narrative text. Do not use any other merchant name field (such as merchant_name, merchant_id, or trade_name) as the display name unless merchant_group is absent, in which case flag the issue to the user before proceeding. Never mix merchant identifiers across the same report.

**G2.6 — Telco Prepaid and Mobile Internet must never be mixed.**
Merchants identified as Telco Prepaid (via commercial_l3 field) must only appear in the Telco Prepaid L3 block. Merchants identified as Mobile Internet (via commercial_l4 field, e.g. '05. Mobile Internet') must only appear in the Mobile Internet L3 block. A merchant appearing in both blocks in the same report is a guardrail violation. The Total L3 row for each block must reflect only that block's merchants.

---

## G3 — Output Structure

**G3.1 — Never skip a section without explanation.**
All four sections must appear in the HTML report (Main Header, Overall Revenue Performance, Risky Business, Category Deep Dive). A changelog summary must appear in the chat response, not in the HTML file. If a section cannot be completed, it must still appear with a clear explanation of why and what data is needed.

**G3.2 — Never add sections not in the template.**
The four report sections and changelog are the complete structure. Do not add commentary, appendices, or analysis outside this structure without explicit user instruction.

**G3.3 — Output format is chat summary + HTML file.**
Output format is: (1) a brief chat summary of 3–5 bullet points covering headline performance, top L2 driver, biggest risk, and one forward signal; followed by (2) the full report as a downloadable HTML file using template.html. Do not produce a Word document or Markdown version.

**G3.4 — Placeholders must match the schema.**
All `{{TOKEN_NAME}}` placeholders used in output must match a token defined in `template_schema.md`. Do not invent new token names mid-report.

---

## G4 — Momentum Signals

**G4.1 — Minimum 3 weeks required for momentum.**
Rising Momentum and Declining Momentum buckets require ≥3 consecutive weeks of data. If fewer weeks are available, omit these buckets and state: _"Momentum signals unavailable — X weeks of data detected, minimum 3 required."_

**G4.2 — Consecutive means no gaps.**
Momentum requires consecutive weekly data. A gap week (missing data) breaks the streak. Do not count across gaps.

**G4.3 — Momentum is week-on-week positive/negative, not absolute level.**
A merchant generating RM 10K/week for 3 weeks is not Rising Momentum — revenue must be increasing each week vs the prior week. Confirm direction, not level.

---

## G5 — Concentration and Risk

**G5.1 — Concentration Risk thresholds are based on Top 5 Cumulative %.**
The risk flag ({{CONCENTRATION_FLAG}}) must always be derived from {{CONCENTRATION_TOP5_CUMULATIVE_PCT}} — the combined Contribution % of the top 5 merchants shown in the totals row at the bottom of the Concentration Risk table.
Default thresholds:
✅ OK: Top 5 Cumulative % < 50%
🟡 Caution: Top 5 Cumulative % ≥ 50% and < 70%
🔴 High Risk: Top 5 Cumulative % ≥ 70%
These thresholds may be overridden by the user but any override must be explicitly noted in the chat summary.

**G5.1a — Contribution % and Top 5 Cumulative % are distinct calculations.**
Contribution % is a standalone per-merchant figure (merchant TW revenue / total commercial TW revenue × 100). Top 5 Cumulative % is the sum of all 5 merchants' Contribution % values, displayed as a totals row. Never use any single merchant's Contribution % to trigger a risk flag — only {{CONCENTRATION_TOP5_CUMULATIVE_PCT}} determines the flag.

The L3 Category column in the Concentration Risk table must always be populated with the merchant's specific L3 category name (Telco Prepaid, Mobile Internet, Digital Lifestyle, Online Marketplaces & Fast Fashion, Daily Essentials & Retail, Everyday F&B and Lifestyle, Travel). Writing 'Category Management' in this column is a guardrail violation — that is the L2 label. Always resolve to the L3 level.

**G5.2 — Early Warning Signals must respect the materiality threshold.**
The minimum revenue threshold for Early Warning Signals is the noise filter threshold for the relevant L3 category. For Telco Prepaid and Mobile Internet, this is the fixed threshold of RM 12,500. For all other L3 categories, this is the dynamic threshold calculated during the validation step. Do not flag merchants whose current-week revenue falls below this threshold — the signal is statistically immaterial. Document the applicable RM value as {{EARLY_WARNING_THRESHOLD}} in the report output.

**G5.3 — Early Warning Signals are observational only.**
Do not speculate on the cause of an early warning signal. Describe the pattern factually. Example: "Merchant X has declined WoW for 2 consecutive weeks (RM –45K total)" — not "Merchant X may be losing market share due to competition."

**G5.4 — Early Warning Signal entries must use L3 category attribution.**
Every Early Warning Signal entry must identify the merchant's L3 category in parentheses, not the L2 pillar. Writing '(Category Management)' in any signal entry is a guardrail violation. The correct format is:
'[merchant_group] ([L3 category name]) — [signal description]'
If the L3 category cannot be resolved from the data, write '(L3 Unknown)' and flag it in the validation output. Never suppress the signal entirely because of a missing L3 label — always include it with the Unknown tag.

**G5.6 — DDNQR Penetration Tracker must use Gross TPV not revenue.**
All three rows of the DDNQR Penetration Tracker (Total DDNQR TPV, Total Category Management TPV, DDNQR Migration %) must be computed from the Gross TPV field in the source data. Using the revenue field for any of these rows is a guardrail violation.

Total Category Management TPV is the sum of Gross TPV for all rows where commercial_l2 = '04 Category Management'. It is computed separately from the revenue-based Total Commercial figure used in Table 1B — they are different source fields and must not be conflated.

DDNQR Migration % = DDNQR Gross TPV / Total Category Management Gross TPV × 100. Using revenue in either the numerator or denominator is a guardrail violation. Using any scope wider than commercial_l2='04 Category Management' in the denominator is a guardrail violation.

If the Gross TPV field is not identifiable in the source data, do not populate the 3 footer rows with revenue figures as a substitute. Flag as unavailable and insert 'Gross TPV field not found' in all cells.

**G5.5 — Each Early Warning Signal entry must be on its own line.**
Never group, combine, or comma-separate multiple merchant signals into a single line or sentence. Each merchant that triggers an early warning condition must appear as a standalone line entry in the format:
'[merchant_group] ([L3 category]) — [signal description with RM values]'
Writing two or more merchants on the same line is a guardrail violation.

**G5.7 — Early Warning Signals are capped at 10 merchants.**
No more than 10 merchants may appear in the Early Warning Signals block. If more than 10 qualify after applying the materiality threshold, use this priority order: (1) single-week drop >20% WoW, (2) 2 consecutive WoW declines, (3) re-entry after ≥3 weeks absent. Within each priority tier, rank by absolute WoW revenue decline (largest first). Stop once 10 entries are reached. Exceeding 10 entries is a guardrail violation.

---

## G6 — Tone and Language

**G6.1 — No filler language.**
Prohibited phrases include: "it is worth noting", "it is important to highlight", "as we can see from the data", "significantly", "quite", "very", "somewhat". Every word must carry information.

**G6.2 — All claims must be anchored to numbers.**
Never write a qualitative claim without a supporting number. "Revenue increased" is incomplete. "Revenue increased RM 340K (+8.2%) WoW" is correct.

**G6.3 — Consistent number formatting.**

- Below RM 1,000,000: RM X,XXX (e.g., RM 48,200 or RM 750,000)
- RM 1,000,000 to below RM 1,000,000,000: RM X.XM (e.g., RM 1.2M)
- RM 1,000,000,000 and above: RM X.XB (e.g., RM 1.2B)
- Percentages: always show sign (e.g., +8.2% or –3.4%), one decimal place
- Do not mix formats within the same table
- Apply each figure's tier independently based on its own magnitude

---

## G7 — Validation Gate

**G7.1 — Validation must complete before report generation.**
The data validation checklist (see `data_validation.md`) must be run and its summary presented to the user before any report section is generated. Do not skip or abbreviate validation.

**G7.2 — User confirmation is required before generating the report.**
After presenting the validation summary, wait for the user to confirm before proceeding. If the user says "proceed" or equivalent, generate the full report. Do not auto-generate.

**G7.3 — Report generation is all-or-nothing per session.**
Do not generate partial reports across multiple messages without user instruction. Generate the complete report in one response (or clearly sequenced messages if length requires it), not section by section unless the user asks.

---

## G8 — Merchant Attribution Coverage

**G8.1 — Coverage caveats are chat-only.**
If merchant-ranked analysis excludes revenue because no usable merchant identifier (merchant_group) is available, do not insert any caveat, note, or disclaimer into the HTML template body. Do not add text to: Main Header, Overall Revenue Performance, Risky Business narrative, or any Category Deep Dive section.

**G8.2 — Standard disclosure wording.**
When unresolved merchant-attributable TW revenue is greater than RM 0, disclose once using exactly this wording:
'Merchant-ranked signals exclude unresolved rows worth RM X (Y% of TW commercial revenue).'

**G8.3 — Placement.**
Disclose in the validation summary if detected before generation. Disclose in the final chat changelog/summary if the report proceeds. Never in the HTML output.

**G8.4 — Tables may still be generated.**
Merchant-ranked tables (buckets, DDNQR, Top Merchants) may still be generated from rows with usable merchant_group identifiers. The coverage limitation is a logged disclosure only — it does not block report generation.

---

## G9 — Conditional Table Removal

**G9.1 — Empty eligible tables must be removed; all others must remain.**
The AI must remove a table from the HTML output when it contains zero qualifying rows after all filtering and noise threshold logic has been applied — this is mandatory, not optional. Leaving an empty eligible table (with no data rows) in the HTML output is a guardrail violation. No other reason justifies removing a table that contains qualifying rows.

**G9.2 — Ineligible tables must always be present.**
The following tables must never be removed regardless of data availability: Top Merchants (per L3), Biggest Winners (per L3), Biggest Losers (per L3), Concentration Risk, Early Warning Signals. If these tables have no data, insert 'No data available this week' in the relevant cells — do not remove the table.

**G9.3 — Removal must be structurally clean.**
When removing an empty table, remove only: the table's own header bar, subtitle row, column header row, data rows, and any padding rows that exist solely for that table. Do not remove shared structural elements, section dividers, or the Back to Top button row. After removal, verify no broken borders, double gaps, or spacing artifacts remain.

**G9.4 — Every removal must be logged.**
Each removed table must be listed in the chat summary with the format:
'Table removed (empty): [TABLE NAME] — [L3 Category or Global]'
Removals must never be silent.

**G9.5 — Back to Top button and grey separator bar integrity.**
After any table removal, both the grey separator bar and the Back to Top button must render correctly and with consistent spacing.

Grey separator bar: the <tr> or <td> containing the grey separator bar (border-top:2px solid #e5e7eb or equivalent) that sits between the L3 content and the Back to Top button must never be removed, merged, or rendered invisible as a result of a table removal above it.

Minimum spacing: after any table removal, the last remaining table in the L3 block must have a minimum of 12px bottom padding between its lower edge and the grey separator bar. If the removed table was providing this spacing, compensate by adding padding-bottom:12px to the last remaining table's outer wrapper <td>.

Back to Top button: the Back to Top button row must always sit below the grey separator bar with minimum padding-top:6px on its wrapping <td>. This must not be reduced by any table removal above it.

Violation: any table removal that results in the grey separator bar being invisible, merged with a table border, or collapsed to zero height is a guardrail violation. Any removal that results in the Back to Top button touching the last remaining table with no separator between them is a guardrail violation.

**G9.6 — L3 outer padding wrapper is permanent.**
The outermost <td> wrapper that provides the horizontal padding for each L3 category block (padding: 0 20px or equivalent) must never be removed from the HTML output. This element is structural — it controls section width alignment and must survive all table removal operations within its block. Removing this wrapper under any circumstance is a guardrail violation.

**G9.7 — DDNQR Penetration Tracker is inseparable from the DDNQR Top 10 table.**
The DDNQR Penetration Tracker (a separate standalone table immediately after the Global DDNQR Top 10, containing Total DDNQR TPV, Total Category Management TPV, and DDNQR Migration % rows) must always appear whenever the Global DDNQR Top 10 table is present. It must never be removed independently of the DDNQR Top 10 table. If the Global DDNQR Top 10 table is removed because it is empty, the Penetration Tracker is also removed with it as a single unit. Generating the DDNQR Top 10 table without the Penetration Tracker is a guardrail violation.

**G9.8 — Pre-generation table removal assessment is mandatory.**
Before HTML generation begins (before Chunk 7), the AI must explicitly assess every eligible removable table across all L3 blocks and global sections and produce a written removal plan in the chat. This plan must list: (1) all tables confirmed empty and marked for removal, (2) all tables confirmed non-empty and marked for retention. HTML generation must not start until this assessment is complete and confirmed. Beginning HTML generation without a completed table removal assessment is a guardrail violation.

**G9.9 — DDNQR Migration % WoW cells must always be blank.**
The WoW RM and WoW % cells for the DDNQR Migration % row in the Penetration Tracker must always be left empty — no value, no dash, no placeholder token. Populating either of these cells with any content is a guardrail violation.

---

## G10 — Data Processing Integrity

**G10.1 — Always use chunked processing.**
The weekly data file must be processed in sequential logical chunks as defined in the ignition prompt and instructions_full.md. Processing the entire dataset in a single operation is a guardrail violation.

**G10.2 — Confirm each chunk before proceeding.**
After completing each processing chunk, confirm its outputs in the chat before beginning the next chunk. Do not proceed to HTML generation until all data chunks are confirmed complete and consistent.

**G10.3 — Always follow the extraction runbook.**
The weekly_performance_snapshot_extraction_runbook.md must be read before any data extraction begins. Skipping the runbook or reading it after extraction has started is a guardrail violation.

**G10.4 — HTML generation is always last.**
The HTML template must not be populated until all data processing chunks are complete and confirmed. Populating placeholders mid-extraction while data chunks are still pending is a guardrail violation.

---

## G11 — Encoding Integrity

**G11.1 — Scan for mojibake on every ingestion using Tier 1 and Tier 2 detection.**
On every data ingestion, scan all text columns (particularly merchant_group, category fields, and descriptive labels) for mojibake using both tiers:

Tier 1 — Explicit pattern matching: flag any row containing: Ã followed by any character; â€ followed by any character; Å followed by any character; ä» or æ or ç or è or é followed by unexpected characters; å followed by any character; Â followed by any character; Ð or Ñ followed by unexpected characters. A single Tier 1 match triggers the full decoding pipeline.

Tier 2 — Structural pattern matching: flag any row where: three or more consecutive accented or non-ASCII characters appear in a field expected to be Latin-script; mixed encoding types appear in a single string; CJK Unicode characters (U+4E00–U+9FFF Chinese, U+3040–U+30FF Japanese, U+AC00–U+D7A3 Korean) appear in fields expected to contain Malay, English, or numeric content.

Failing to detect Tier 1 mojibake that is visible in the raw data is a guardrail violation. Failing to report Tier 2 flags is also a guardrail violation.

**G11.2 — Apply the 4-pass decoding pipeline.**
If any Tier 1 or Tier 2 flag is triggered, apply these passes in order, stopping at the first accepted result:
Pass 1 — Latin-1 → UTF-8: treat as Latin-1 bytes, decode as UTF-8.
Pass 2 — Windows-1252 → UTF-8: treat as Windows-1252 bytes, decode as UTF-8.
Pass 3 — Double-encoding correction: attempt to decode twice where text appears UTF-8 encoded twice.
Pass 4 — Partial decode with flag: decode decodable subsequences, escape remaining bytes as hex, flag the row as 'partially decoded — manual review required'.

Apply corrections consistently across all affected rows and fields before any analysis or table population begins. Skipping directly to Pass 4 without attempting Passes 1–3 is a guardrail violation.

**G11.2a — Partial decodes must be used, not discarded.**
A row that reaches Pass 4 (partially decoded) must still be included in analysis using its best-available decoded value. Do not drop partially decoded rows. Flag them individually in the validation summary using format: '[row identifier]: original=[garbled value] | partial decode=[result] | status=manual review required'.

**G11.3 — Always report corrections explicitly.**
Mojibake corrections must never be silent. Report in the validation summary: which columns were affected; how many rows were corrected at each pass (Pass 1 / Pass 2 / Pass 3 / Pass 4); at least one before/after example per pass that corrected at least one row. Do not proceed to analysis chunks without surfacing this finding and receiving user acknowledgement.

**G11.4 — Escalate rows where all passes fail.**
If all four passes fail to produce an accepted result, preserve the original garbled value, flag the row explicitly in the validation summary, and await user instruction before using that row in any ranked table or calculation.

---

## G12 — MTD Month Boundary Integrity

**G12.1 — Always check for rollover weeks before computing MTD.**
Before any MTD figure is calculated, compare the week-start date and TW_DATE. If they fall in different calendar months, a rollover week exists and boundary logic must be applied. Skipping this check is a guardrail violation.

**G12.2 — Never include prior-month revenue in current-month MTD.**
In a rollover week, any transaction dates that fall before the first day of the current month must be excluded from all MTD calculations at every level — overall, L2, L3, and merchant. Including prior-month days in the current MTD is a guardrail violation.

**G12.3 — Rollover logic applies to MTD only.**
WoW, LW, TW, and YTD calculations must never be altered by rollover week detection. Applying month boundary filtering to WoW, LW, TW, or YTD figures is a guardrail violation.

**G12.4 — Rollover weeks must always be disclosed.**
When a rollover week is detected, the validation summary must include the rollover disclosure before the report is generated. Silent handling of a rollover week without disclosure is a guardrail violation. The required disclosure format is:
'Rollover week detected: week spans [week_start_date] to [TW_DATE]. MTD includes [boundary_date] to [TW_DATE] only ([N days]). Prior month dates excluded: [list].'

**G12.5 — MTD budget and stretch comparisons must note partial month context.**
When a rollover week exists and only partial-month actuals are available, the validation summary must note that the MTD actual reflects a partial month. The MTD budget and stretch targets remain as provided in the source data — do not adjust them unless the source data provides a daily budget breakdown.

---

## G13 — YTD Accumulation Integrity

**G13.1 — YTD always covers 1 January to TW_DATE inclusive.**
No date filter, month boundary rule, or file selection logic may reduce the YTD window below 1 January to TW_DATE of the current year. Applying any additional date restriction to YTD is a guardrail violation.

**G13.2 — MTD boundary filter must never contaminate YTD.**
The month boundary filter used for MTD rollover weeks operates in complete isolation. YTD must be computed from an unfiltered dataset. Using a month-filtered MTD dataset as input for YTD computation is a guardrail violation.

**G13.3 — Current week full revenue must be included in YTD.**
Even in a rollover week where prior-month days are excluded from MTD, those same days must be included in YTD. The current reporting week always contributes its full 7-day revenue to YTD. Partial-week YTD contribution is a guardrail violation.

**G13.4 — No double-counting across files.**
If any two weekly files cover overlapping date ranges, the overlapping days must be counted only once using the more recent file. Silent double-counting of any date range in YTD is a guardrail violation.

**G13.5 — Gaps must be disclosed not estimated.**
If a weekly file is missing from the YTD sequence, the gap must be flagged and disclosed. Interpolating or estimating revenue for a missing week to fill a YTD gap is a guardrail violation.

**G13.6 — Budget and stretch are always read as provided.**
YTD budget and YTD stretch targets are pre-aggregated values from the budget/stretch file. The AI must not recompute, adjust, or override these figures. Using a computed value in place of the provided budget/stretch target is a guardrail violation.

**G13.7 — YTD file coverage must always be disclosed.**
The validation summary must always state how many weekly files were used in the YTD computation and the date range they cover. Generating a YTD figure without disclosing its file coverage is a guardrail violation.

---

## G14 — Daily Essentials & Retail Petrol Merchant Exclusion

**G14.1 — Petrol merchants must never appear in ranked tables.**
Within the Daily Essentials & Retail L3 block, no merchant identified as a petrol merchant (by merchant_group name or use-case field) may appear in any ranked table: Top Merchants, Winners, Losers, Rising Momentum, Declining Momentum, Reactivated, New Entrants, or DDNQR Top 5. A petrol merchant name appearing in any ranked table is a guardrail violation.

**G14.2 — Petrol revenue must be retained in all totals.**
Excluding petrol merchants from ranked tables must never reduce the Total L3 revenue figures. The Total L3 row must always reflect the full Daily Essentials & Retail revenue including petrol. Removing petrol revenue from any total is a guardrail violation.

**G14.3 — Excluded ranks must be backfilled.**
If a petrol merchant would have ranked in the Top 5 of any table, the next qualifying non-petrol merchant in that sort order must fill the position. Leaving a blank row or reducing the table to fewer than 5 rows when 5 non-petrol merchants exist is a guardrail violation.

**G14.4 — Known petrol merchants.**
The following merchant_group names are always treated as petrol regardless of data context: Petronas, Shell, Caltex, Petron, BHPetrol. Additional petrol merchants identified via the use-case field must also be excluded. If a new petrol merchant is discovered that is not in this list, flag it in the validation summary.

---

## G15 — eSIM Extraction from Crossborder

**G15.1 — eSIM must always be shown as a standalone row in Table 1B.**
eSIM revenue must never be included within the Crossborder row figure. The Crossborder row must always be net of eSIM. Showing a Crossborder figure that includes eSIM revenue is a guardrail violation.

**G15.2 — eSIM identification uses commercial_l3 field.**
eSIM merchants are identified exclusively by filtering commercial_l3 = eSIM (or equivalent eSIM value). Do not use merchant_group naming to identify eSIM merchants — use the L3 field only.

**G15.3 — Total Commercial must remain unchanged.**
The reclassification of eSIM from Crossborder to a standalone row must not change the Total Commercial figure. Total Commercial = sum of all 7 rows including both Crossborder (excl. eSIM) and eSIM. Any discrepancy in the Total Commercial figure after eSIM extraction is a guardrail violation.

**G15.4 — eSIM row position is fixed.**
The eSIM row must always appear between Crossborder (excl. eSIM) and Foreign Worker Segment in Table 1B. It must never be moved, merged with another row, or omitted even if eSIM revenue is zero for the week (display zero with - for WoW % if LW is also zero).

---

## G16 — DDNQR Revenue Must Be MID-Scoped

**G16.1 — DDNQR revenue figures must come only from MID=EP142731 rows.**
When computing any DDNQR revenue metric — YTD, MTD, LW, TW, WoW RM, WoW % — the source rows must be filtered to MID = EP142731 before any aggregation. Using a merchant's total revenue across all MIDs as a proxy for its DDNQR revenue is a guardrail violation.

**G16.2 — MID filter is applied before merchant identification, not after.**
The correct processing order is: (1) filter all rows to MID = EP142731, (2) group the resulting rows by merchant_group, (3) sum revenue within each group. Applying merchant identification first and then attempting to extract DDNQR revenue is a guardrail violation.

**G16.3 — Global DDNQR total row must sum ALL qualifying merchants, not just the top 10.**
The Total DDNQR Revenue footer row in the Global Top 10 table must reflect the sum of DDNQR-scoped revenue across ALL merchants with MID=EP142731 across all commercial L2 pillars — not only the 10 displayed rows. The per-L3 DDNQR Top 5 tables do not have a Total row. Limiting the global total to only the displayed ranked rows is a guardrail violation.

**G16.4 — DDNQR sort order must use MID-scoped YTD revenue.**
Merchants in all DDNQR tables must be ranked by their MID=EP142731 YTD revenue, descending. Ranking by total merchant YTD revenue or by any other metric is a guardrail violation.

**G16.5 — DDNQR audit block is a mandatory gate before HTML generation.**
The DDNQR audit block (all 5 checks) must be completed and reported in the chat before Chunk 7 (HTML generation) begins. Beginning HTML generation without a completed and passing DDNQR audit is a guardrail violation. If any audit check fails, stop and resolve with the user before proceeding.

**G16.6 — DDNQR revenue and Gross TPV must never be mixed.**
Object 1 (ddnqr_global_revenue) and Object 2 (ddnqr_l3_revenue) use the revenue field only. Object 3 (ddnqr_penetration_tpv) uses the Gross TPV field only. Using Gross TPV in Objects 1 or 2, or using revenue in Object 3, is a guardrail violation. If Object 3 Row 1 TW equals Object 1 Total TW exactly, flag as a likely field-mapping error before proceeding.

**G16.7 — The three DDNQR objects are computed in complete isolation.**
No figure from Objects 1, 2, or 3 may be derived from, reused from, or cross-contaminated with the revenue figures used in Table 1B, L3 Top Merchants tables, or any other non-DDNQR calculation. Each object draws directly from the source data with its own filter applied first.
