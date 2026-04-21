SYSTEM INITIALISATION — READ ALL KNOWLEDGE BASE FILES BEFORE PROCEEDING

Before taking any action on the data or generating any output, you must first read every file in the knowledge base in this exact order:

1. README.md — understand the project context and overall workflow
2. instructions_full.md — load all report logic, business rules, and formatting instructions
3. template_schema.md — load the expected data schema and column mapping rules
4. data_validation.md — load the full data validation checklist and flag criteria
5. guardrails.md — load all output guardrails and quality control rules
6. changelog_format.md — load the changelog block format and versioning rules
7. template.html — load the full HTML template (this is your HTML output blueprint)

Confirm you have read all 7 files before proceeding. Output a single line:
"Knowledge base loaded: README.md, instructions_full.md, template_schema.md, data_validation.md, guardrails.md, changelog_format.md, template.html"

Do not proceed past this point until all 7 files are confirmed loaded.

---

Before accessing the data file, read weekly_performance_snapshot_extraction_runbook.md from the project Sources. Follow the extraction sequence defined in that runbook exactly to guide how you read, filter, and process the data file. Do not begin data extraction until you have read the runbook in full.

Process the data file in chunks. Do not attempt to load or compute the entire dataset in a single operation. Follow this chunking sequence:

Chunk 1 — Header and schema: read column names and identify all logical field roles (merchant_group, L2, L3, revenue columns, YTD, MTD, budget, stretch, prior weeks). Confirm schema and report findings before proceeding.

Chunk 2 — L2 pillar aggregation: compute total revenue figures for each L2 pillar (SME, Mobility, Government Services, Category Management, Crossborder, Foreign Worker Segment) for LW and TW. Compute variances. eSIM extraction: filter commercial_l3 = eSIM. Compute eSIM LW and TW totals separately. Subtract from Crossborder totals to get Crossborder (excl. eSIM). Verify Total Commercial is unchanged after the split. Report both figures before proceeding.

Chunk 3 — YTD and MTD targets with rollover week check:

Step 3a — Rollover week detection:
Compare the week-start date (Monday of the reporting week) with TW_DATE.
If they fall in different calendar months, a rollover week exists.
Identify boundary_date = first day of month(TW_DATE).
List all transaction dates in the reporting week that fall before boundary_date — these are excluded from MTD.
Report rollover status before proceeding:
If rollover: 'Rollover week detected: MTD includes [boundary_date] to [TW_DATE] ([N days]). Excluded from MTD: [list of prior-month dates].'
If no rollover: 'No rollover week. Standard MTD applied.'

Step 3b — MTD calculation:
If rollover week: sum revenue from boundary_date to TW_DATE only at overall, L2, L3, and merchant levels.
If no rollover: sum revenue from 1st of current month to TW_DATE.

Step 3c — YTD computation (separate pass — do not reuse MTD dataset):

File inventory: list all weekly data files available for the current year. Confirm date ranges covered by each file. Check for:

- Overlaps: if two files cover overlapping dates, flag and use the more recent file for the overlapping period
- Gaps: if any week between 1 Jan and TW_DATE has no file coverage, flag and note YTD understatement risk

Compute YTD as the sum of all revenue across all available weekly files from 1 January to TW_DATE inclusive. Apply this at every level: overall commercial, per-L2 pillar, per-L3 category, per-merchant.

Critical: compute YTD from the original unfiltered dataset — never from the month-filtered MTD dataset. The current reporting week contributes its full 7-day revenue to YTD regardless of rollover week status.

Read YTD budget and YTD stretch targets directly from the budget/stretch file as pre-aggregated values. Do not compute or adjust these figures.

Report before proceeding:
'YTD computed from [N] files covering [earliest date] to [TW_DATE].'
List any overlaps or gaps detected and how they were resolved.
Confirm MTD and YTD were computed as separate passes.

Step 3d — MTD vs Budget and MTD vs Stretch:
Compare calculated MTD actual against the MTD budget and MTD stretch values from the source data. If rollover week, note in validation summary that MTD actual is partial-month only.

Report all YTD and MTD figures with variances before proceeding to Chunk 4.

Chunk 4 — Category Management L3 filtering: filter to L2 = 04 Category Management only. Apply per-L3 noise thresholds. Confirm qualifying merchant counts per L3 and report before proceeding.

Chunk 5 — Per-L3 merchant analysis: process each of the 7 L3 categories one at a time. For each L3 compute: Top Merchants, Winners, Losers, Momentum, Reactivated, New Entrants. For each L3's DDNQR Top 5 (Object 2 — ddnqr_l3_revenue): filter all rows to MID = EP142731 first, then filter to the L3 scope, then group by merchant_group and sum revenue from MID=EP142731 rows only. Sort by DDNQR-specific YTD revenue descending. The Total DDNQR Revenue footer row sums ALL qualifying MID=EP142731 merchants within that L3 — not just the top 5 displayed rows. Confirm completion of each L3 before moving to the next.

Chunk 6 — Global DDNQR isolated calculation path and Risky Business:

DDNQR phase — compute all three objects in sequence before any other Chunk 6 work:

Object 1 (ddnqr_global_revenue): filter entire dataset to MID = EP142731. Group by merchant_group, sum revenue from MID=EP142731 rows only for all periods. Sort by YTD DDNQR revenue descending. Select top 10 for display. Compute Total row from ALL merchants in DDNQR subset (not just top 10).

Object 2 (ddnqr_l3_revenue): confirm already completed in Chunk 5. Report any L3 with zero DDNQR merchants.

Object 3 (ddnqr_penetration_tpv): identify Gross TPV field (distinct from revenue field). Compute: Row 1 = sum Gross TPV for MID=EP142731 merchants per period; Row 2 = sum Gross TPV for commercial_l2='04 Category Management' per period (all merchants, not just DDNQR); Row 3 = Row 1 / Row 2 × 100 per period. WoW RM and WoW % cells for Row 3 are always blank. If Gross TPV field not found: flag, insert 'Gross TPV field not found' in all cells, proceed.

DDNQR Audit Block (mandatory — must complete before Chunk 7):
Run and report all 5 audit checks:
1. Object completeness: Objects 1, 2, 3 all populated; merchant counts reported.
2. Revenue isolation: confirm no non-DDNQR revenue in Objects 1/2; no TPV in Objects 1/2.
3. Total row scope: Object 1 total covers ALL DDNQR merchants; each L3 in Object 2 covers ALL DDNQR merchants in that L3.
4. Migration % cross-check: recalculate TW Migration % from raw figures and confirm it matches {{DDNQR_MIGRATION_TW}} to 1 decimal place.
5. TPV vs revenue separation: confirm Object 3 Row 1 TW ≠ Object 1 Total TW (if equal, flag as likely field-mapping error).
Report as DDNQR AUDIT SUMMARY block. Do not proceed to Chunk 7 if any check fails.

Risky Business — after DDNQR audit passes: compute Concentration Risk across all L2 pillars. Assess Early Warning Signals.

Early Warning Signals formatting: each merchant must appear on its own separate line. Do not combine multiple merchants into one sentence or list them with commas. Format strictly as one entry per line. Maximum 10 merchants total — if more qualify, apply priority: (1) >20% single-week WoW drop, (2) 2 consecutive WoW declines, (3) re-entry after ≥3 weeks absent; within each tier rank by absolute WoW decline descending.

Chunk 7 — HTML generation: only after all data chunks are confirmed complete AND the DDNQR audit block has passed, begin filling the HTML template placeholders. Do not start HTML generation while any data chunk is still incomplete, unconfirmed, or while any DDNQR audit check is outstanding.

If any chunk fails or returns unexpected results, stop and report the issue in the chat before proceeding to the next chunk.

---

DATA INGESTION

Please access this week's revenue data file from the Sources section in this project. The file will be available directly in your knowledge base — do not attempt to access SharePoint or any external location.

You must ingest ALL files listed below. Do not skip any. Do not proceed until all required files are successfully read.

MAINTAINABLE WEEKLY FILE MANIFEST — ONLY EDIT THIS BLOCK EACH WEEK
Instructions:

- Add exactly one new weekly file line at the top each week
- Update PRIMARY_REPORT_WEEK to the newest file
- Update PRIOR_REPORT_WEEK to the immediately preceding week
- Do not rename historical files in this list
- Do not delete earlier weeks within the same reporting year unless explicitly instructed

PRIMARY_REPORT_WEEK = 20260329_report.csv
PRIOR_REPORT_WEEK = 20260322_report.csv
REPORT_WEEK_ENDING = 29 March 2026
REPORT_VERSION = Draft

WEEKLY_FILE_MANIFEST (newest first):

1. 20260329_report.csv
2. 20260322_report.csv
3. 20260315_report.csv
4. 20260308_report.csv
5. 20260301_report.csv
6. 20260222_report.csv
7. bidrr7625_20260215_others.csv
8. 20260215_report.csv
9. 20260208_report.csv
10. 20260201_report.csv
11. 20260125_report.csv
12. 20260118_report.csv
13. 20260111_report.csv
14. 20260104_report.csv
15. 20260101_20260208_others.csv

TARGET FILES: 16. Budget_2026.csv 17. Stretch_2026.csv

DATA CONSTRUCTION RULES

- Build the combined revenue dataset by stacking every file in WEEKLY_FILE_MANIFEST.
- The primary report week is PRIMARY_REPORT_WEEK.
- For WoW calculations:
  - TW = PRIMARY_REPORT_WEEK
  - LW = PRIOR_REPORT_WEEK
- For momentum classification:
  - use at minimum the trailing weekly trend across the latest 3 weeks ending at PRIMARY_REPORT_WEEK
  - use longer history from the full weekly manifest where available
- YTD figures must accumulate from the earliest available date in the oldest weekly file in WEEKLY_FILE_MANIFEST through REPORT_WEEK_ENDING.
- MTD figures must cover the first calendar day of the report month through REPORT_WEEK_ENDING.
- Reactivated = TW > 0 and LW = 0 exactly.
- New Entrant = no revenue recorded in any prior week across the full combined dataset before TW.
- All figures are Gross Revenue from payment data only.
- Exclude manual billing and ancillary income from analysis sections that explicitly require payment data only.
- Do not fabricate any data or infer missing values without explicitly flagging them.

TARGET FILTER RULES

- In Budget_2026.csv, use only rows where Rating = 2. Treat these rows as the official Budget targets.
- In Stretch_2026.csv, use only rows where Rating = 4. Treat these rows as the official Stretch targets.
- Ignore rows with other Rating values unless required for validation diagnostics.

TARGET PRORATION RULES

- If Budget_2026.csv or Stretch_2026.csv contains explicit weekly targets, use them directly.
- Else if the file contains monthly targets:
  - use the report month target directly for MTD
  - use the sum of monthly targets from January through the report month for YTD
- Else if the file contains annual targets only:
  - YTD target = annual target × (days elapsed in year through REPORT_WEEK_ENDING / total days in year)
  - MTD target = annual target × (days elapsed in report month through REPORT_WEEK_ENDING / total days in report month)
- Always prefer the most granular target level available in the source file over pro-rating.

ACCESS FAILURE RULE

- If any required file cannot be accessed or read, stop immediately after ingestion / validation and report:
  - exact filename
  - exact step that failed
  - whether the failure was path resolution, file fetch, schema read, or parse error
- Do not continue to report generation with any missing required file.
- Do not silently substitute `.xlsx` for `.csv` or vice versa unless the user explicitly instructs that substitution.

Confirm all listed files are successfully ingested before proceeding to validation.

---

DATA VALIDATION — MANDATORY BEFORE ANY OUTPUT

Run the full data validation checklist as defined in data_validation.md and report back exactly what you found across every item below. Do not skip any item. Do not summarise — be explicit.

For each file in the full source set, validate and report all of the following:

1. Schema mapping

- List every column role you have mapped and the exact source header used.
- Do this file by file.
  - Confirm merchant_group has been identified as the primary merchant identifier. If merchant_group is absent or inconsistent, flag before proceeding.
  - Confirm TW_DATE and LW_DATE: identify the week-ending dates for the current reporting week (TW_DATE) and prior reporting week (LW_DATE). These are injected into all TW and LW column headers across every table in the report. Every "TW" header must render as "TW — {{TW_DATE}}" and every "LW" header must render as "LW — {{LW_DATE}}".
  - MTD rollover check: compare week-start date (Monday) and TW_DATE. If different calendar months, identify boundary_date (1st of TW month) and list all prior-month dates in the week that must be excluded from MTD. Report rollover status explicitly before any MTD computation begins.
  - YTD file coverage: list all weekly files available for the current year. Confirm date range coverage from 1 Jan to TW_DATE. Flag any overlaps (use more recent file) or gaps (flag understatement risk). Confirm YTD will be computed as a separate pass from the unfiltered dataset — not from the MTD-filtered dataset.
  - Mojibake scan: scan all text columns in each ingested file using two detection tiers. Tier 1 — flag any row containing: Ã followed by any char; â€ followed by any char; Å followed by any char; ä» or æ or ç or è or é followed by unexpected chars; å followed by any char; Â followed by any char; Ð or Ñ followed by unexpected chars. Tier 2 — flag: three or more consecutive accented/non-ASCII chars in a Latin-script field; mixed encoding types in a single string; CJK Unicode chars (U+4E00–U+9FFF, U+3040–U+30FF, U+AC00–U+D7A3) in fields expected to contain Malay, English, or numeric content. Apply 4-pass decoding pipeline: Pass 1 Latin-1→UTF-8, Pass 2 Windows-1252→UTF-8, Pass 3 double-encoding correction, Pass 4 partial decode + hex-escape + flag as 'manual review required'. Report per-pass correction counts and at least one before/after example per corrective pass. Pass 4 rows must be used in analysis (not discarded) and listed individually. If all 4 passes fail, preserve original, flag row, await user instruction before using in ranked tables.
  - Confirm L3 category is resolvable for all merchants. This is required for correct attribution in the Concentration Risk table (L3 Category column) and Early Warning Signals entries. Flag any merchant where L3 cannot be determined.
  - Confirm Telco Prepaid merchants are identifiable via the commercial_l3 field. Report the merchant count before proceeding.
  - Confirm Mobile Internet merchants are identifiable via the commercial_l4 field (look for values such as '05. Mobile Internet'). Report the merchant count before proceeding. Both Telco Prepaid and Mobile Internet use a fixed noise filter of RM 12,500 — do not compute a dynamic threshold for either.
  - Daily Essentials petrol exclusion: identify all petrol merchants within Daily Essentials & Retail using merchant_group name matching (Petronas, Shell, Caltex, Petron, BHPetrol) and use-case field. List identified merchants and confirm their revenue will be excluded from ranked tables but retained in Total L3 figures.
  - eSIM extraction from Crossborder: identify eSIM merchants via commercial_l3 = eSIM filter. Report eSIM merchant count, combined LW and TW revenue. Confirm Crossborder (excl. eSIM) and eSIM row values. Confirm Total Commercial is unchanged after the split.
  - DDNQR Penetration Tracker: identify the Gross TPV field in the source data. Confirm it is distinct from the revenue field used elsewhere in the report. Note: the WoW RM and WoW % cells for the DDNQR Migration % row (row 3) must always be left blank — do not populate with any value.
  - Empty table removal assessment (mandatory — must complete before HTML generation): for each bucket table and DDNQR table (both global Top 10 and all 7 per-L3 Top 5 tables), confirm whether any qualifying rows exist after filtering and noise threshold application. Produce a written removal plan in the chat listing every table assessed, whether it is empty (remove) or non-empty (retain). Do not start HTML generation until this plan is confirmed. Then apply the following rules:

    WHAT TO REMOVE: if a bucket table or DDNQR table (global or per-L3) contains zero qualifying rows, remove it entirely from the HTML output — including the table's label/header bar, subtitle row, column header row, all data rows, and any wrapping padding rows that exist solely for that specific table.

    WHAT TO NEVER REMOVE: Top Merchants tables, Biggest Winners tables, Biggest Losers tables, Concentration Risk table, Early Warning Signals block, L3 title bars, noise filter/overview cards, section headers, Back to Top buttons, Quick Links, and the footer. These must always remain in the HTML output regardless of data availability.

    PADDING CONSISTENCY: after removing any table, verify that the last remaining table in the same L3 block has at minimum 12px bottom padding between its lower edge and the grey separator bar. If the removed table was providing this spacing, add padding-bottom:12px to the last remaining table's outer wrapper <td>. The Back to Top button must retain minimum padding-top:6px on its wrapping <td>. The grey separator bar itself must never be removed.

    DDNQR TOP 5 REMOVAL — SPECIFIC PADDING RULE:
    When the DDNQR Top 5 table is removed from any L3 category block, the DDNQR table is always the last content element before the grey separator bar and Back to Top button. Its removal therefore always leaves the grey separator bar sitting directly adjacent to the table above it with no buffer.

    After removing the DDNQR Top 5 table from any L3 block, you must immediately apply the following fix to that L3 block:

    Step 1 — Find the grey separator bar row in that L3 block. It is the <tr> or <td> element containing border-top:2px solid #e5e7eb (or equivalent) that sits between the L3 content and the Back to Top button row.

    Step 2 — Add or update the padding on the wrapping <td> of that grey separator bar row to include padding-top:12px. If it already has padding-top set to a value less than 12px, increase it to 12px. If it is already 12px or greater, leave it unchanged.

    Step 3 — Verify the Back to Top button row itself retains its own internal padding (minimum padding-top:6px on its wrapping <td>). If the padding was reduced by the removal, restore it to 6px.

    Step 4 — Confirm the grey separator bar is visually distinct — it must appear as a clear horizontal line with breathing room above it, not merged with the border of the table above.

    Apply this specific rule to every L3 block where the DDNQR Top 5 table is removed. If DDNQR Top 5 is removed from 3 L3 blocks, apply this fix to all 3 grey separator bars independently.

    Do not apply this rule when other tables (not DDNQR) are removed — those cases are covered by the general padding consistency rules above.

    L3 OUTER PADDING WRAPPER: the outermost <td> wrapper that provides horizontal padding (padding: 0 20px or equivalent) for each L3 category block must never be removed regardless of which tables within the block are empty or removed.

    LOG: list every removed table in the chat summary using: 'Table removed (empty): [TABLE NAME] — [L3 Category or Global]'

2. Unmapped or ambiguous columns

- List any columns that could not be cleanly mapped.
- State why each is unmapped or ambiguous.

3. File continuity check

- Confirm whether the files in WEEKLY_FILE_MANIFEST form a clean, unbroken weekly series in chronological order.
- Flag any discontinuities, missing weekly intervals, duplicate weeks, or misdated files.

4. Duplicate / overlap check

- Confirm whether any weekly files contain overlapping reporting periods that would create double counting.
- State:
  - earliest week-ending date in the combined dataset
  - latest week-ending date in the combined dataset
- Flag any overlap, duplicate week-ending date, or missing bridge period.

5. Missing values

- List every field with nulls, blanks, or missing values across all source files.
- State the row count affected for each field.

6. Inconsistent merchant names

- Flag any merchant names that appear to be duplicates or variants across files.
- Include examples such as capitalisation differences, trailing spaces, punctuation differences, abbreviations, or inconsistent legal suffixes.

7. Insufficient weeks for momentum

- Flag any L3 category where the combined history from WEEKLY_FILE_MANIFEST still yields fewer than 3 usable weekly data points.

8. Budget / Stretch coverage

- Confirm that Budget_2026.csv and Stretch_2026.csv contain valid entries for:
  - all commercial pillars used in Section 2
  - all 6 L3 categories under "04 Category Management"
- Apply the rating filters first:
  - Budget = Rating 2 only
  - Stretch = Rating 4 only
- Flag any missing rows, duplicate rows, or conflicting rows.

9. Dynamic noise filter

- State the exact threshold you will apply for Section 3.
- State the exact derivation logic used, per data_validation.md.
- Apply the threshold at L3 level based on the lower of:
  - 1% of current-week L2 "04 Category Management" weekly revenue attributable to the relevant L3, or
  - the bottom quartile weekly merchant revenue within that L3
- If data_validation.md defines a stricter method, follow that method and state it explicitly.

10. Date range

- State the full date range covered by the combined dataset:
  - earliest available date
  - latest available week-end date

11. Section readiness
    For each report section below, state one of:

- FULLY GENERATED
- PARTIALLY GENERATED
- SKIPPED

Required sections:

1. Main Header
2. Overall Revenue Performance
3. Risky Business
4. Category Deep Dive
5. Changelog Block

For any section marked PARTIALLY GENERATED or SKIPPED:

- explain exactly what is missing
- state what data would be needed to complete it

Output the validation report in full.
Wait for user confirmation to proceed if any SKIPPED sections are flagged.
Otherwise, proceed automatically.

Output format: produce a brief summary of key findings directly in the chat (3–5 bullet points covering headline performance, top L2 driver, biggest risk, and one forward signal), then generate the full report as a downloadable HTML file using the template.html file. Do not produce a Word document.

Currency: all figures are RM. Three-tier display: RM X,XXX for amounts below RM 1,000,000 | RM X.XM for RM 1,000,000 to below RM 1,000,000,000 | RM X.XB for RM 1,000,000,000 and above.

Merchant naming: use merchant_group as the display name for all merchants in every table and narrative. Do not use any other name field.

Do not skip any section. Do not add sections not listed. If a section cannot be completed, include it with a clear explanation of why and what data would be needed.

Use template.html as the exact output structure for the HTML file. Fill every {{TOKEN_NAME}} placeholder with the correct value derived from the data. Do not alter the template design, layout, colours, fonts, spacing, section order, or any HTML structure. Your only job is to replace placeholders with data. If a placeholder cannot be filled due to missing data, insert '-' and flag it in the chat summary.

---

REPORT GENERATION

Generate the full Weekly Performance Snapshot report in the exact order below. Do not skip any section. Do not add sections not listed. Do not truncate any table or narrative.

If a section cannot be fully completed, include it with an explicit explanation of:

- what is missing
- why it is missing
- what data would be needed to complete it

Follow all logic, rules, and formatting defined in instructions_full.md and guardrails.md. No deviations.

Currency and figure rules:

- Currency = RM
- Raw data contains no currency symbol
- All figures represent Gross Revenue based on payment data only
- Manual billing and ancillary income are excluded where required by the report rules
- Display format (three tiers):
  - RM X,XXX for amounts below RM 1,000,000
  - RM X.XM for amounts RM 1,000,000 to below RM 1,000,000,000
  - RM X.XB for amounts RM 1,000,000,000 and above
- Variances must always show sign where applicable

---

SECTION ORDER

1. Main Header

- Use the format defined in template.html
- Include:
  - report week ending date
  - date generated
  - all source filenames ingested
  - schema version
  - REPORT_VERSION

2. Overall Revenue Performance

Table 1A: Performance vs Targets

- Rows:
  - YTD vs Budget
  - MTD vs Budget
  - MTD vs Stretch
- Columns:
  - Target
  - Actual
  - Variance RM
  - Variance %
- Sources:
  - Actuals from the combined weekly dataset
  - Budget targets from Budget_2026.csv using Rating = 2 only
  - Stretch targets from Stretch_2026.csv using Rating = 4 only
  - Apply target proration rules only where required

Table 1B: Week-on-Week by Commercial Pillar

- WoW must appear only in Table 1B, not in Table 1A
- Rows:
  - SME
  - Mobility
  - Government Services
  - Category Management
  - Crossborder
  - Foreign Worker Segment
  - Total Commercial
- Columns:
  - LW
  - TW
  - WoW RM
  - WoW %
- Sources:
  - LW = PRIOR_REPORT_WEEK totals
  - TW = PRIMARY_REPORT_WEEK totals

Narrative

- 3–5 sentences
- Cover:
  - what changed vs last week
  - whether performance is accelerating or decelerating
  - whether performance is broad-based or concentrated in specific pillars

3. Risky Business

- Follow format and logic in instructions_full.md
- Concentration Risk scope: top 5 merchants across ALL commercial L2 pillars; denominator = total commercial portfolio weekly revenue
- All merchants, figures, and risk flags must be drawn from live data across the combined dataset
- No placeholder rows
- No generic filler

4. Category Deep Dive

- Scope: L2 "04 Category Management" only
- Section header must read exactly:
  Category Deep Dive
- Do not include "04" in the section title

Cover all 7 L3 categories in this exact order:
i. Telco Prepaid
ii. Mobile Internet
iii. Digital Lifestyle
iv. Online Marketplaces & Fast Fashion
v. Daily Essentials & Retail
vi. Everyday F&B and Lifestyle
vii. Travel

For each L3 category, produce ALL of the following:

a. Narrative block with one labelled sub-section

- Overview

b. Top Merchants table

- Top 5 merchants sorted by YTD descending
- Plus one Total L3 summary row
- Columns:
  - Merchant
  - YTD
  - MTD
  - LW
  - TW
  - WoW RM
  - WoW %

YTD source

- all weekly files in WEEKLY_FILE_MANIFEST accumulated through REPORT_WEEK_ENDING

c. Six bucket tables
Each bucket table must be fully populated.
Do not use placeholder rows.
Do not use "-" unless genuinely no qualifying merchants exist.
If a bucket is empty, use a single row stating:
"No merchants meet this criterion this week"

Required bucket tables:

- Biggest Winners
  - sort by WoW RM descending, then YTD descending
- Biggest Losers
  - sort by WoW RM ascending, then YTD descending
- Rising Momentum
  - source = trailing weekly trend ending at PRIMARY_REPORT_WEEK, with longer lookback across the full weekly manifest where needed
- Declining Momentum
  - source = same as above
- Reactivated
  - definition = TW > 0 and LW = 0 exactly
- New Entrants
  - definition = no revenue recorded in any prior week across the full combined dataset before TW

All bucket tables use columns:

- Merchant
- YTD
- MTD
- LW
- TW
- WoW RM
- WoW %

Noise filter rule

- Apply the noise filter threshold confirmed in the validation step.
- Exclude sub-threshold merchants from bucket tables.
- Still include sub-threshold merchants in Top Merchants if they rank in the Top 5 by YTD.

5. Changelog Block

- Use the exact format defined in changelog_format.md
- Record:
  - report week ending date
  - all files ingested
  - schema version
  - REPORT_VERSION
  - any flags raised during validation
  - any sections partially generated
  - any overrides applied
  - target filter rule used:
    - Budget = Rating 2
    - Stretch = Rating 4

---

OUTPUT INSTRUCTIONS — CRITICAL

OUTPUT — HTML FILE (Full, downloadable)

- Use template.html as the exact structural and styling blueprint
- Render as a complete, self-contained HTML file suitable for Outlook and browser viewing
- All tables, narratives, and headers must be present and filled
- No section may be truncated or collapsed
- The file must be complete end-to-end
- Provide a download link

Do not produce a Word document. Do not produce inline previews as a substitute for a downloadable file.
