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

DATA INGESTION — SHAREPOINT FILES

All source files are located on SharePoint at this exact folder:
https://tngd.sharepoint.com/sites/CommercialData/Shared%20Documents/NigelTest/bidrr7625/Performance%20Files

Connector path reference:
- hostname = tngd.sharepoint.com
- site_path = /sites/CommercialData
- folder_path = NigelTest/bidrr7625/Performance Files

You must ingest ALL files listed below. Do not skip any. Do not proceed until all required files are successfully read.

MAINTAINABLE WEEKLY FILE MANIFEST — ONLY EDIT THIS BLOCK EACH WEEK
Instructions:
- Add exactly one new weekly file line at the top each week
- Update PRIMARY_REPORT_WEEK to the newest file
- Update PRIOR_REPORT_WEEK to the immediately preceding week
- Do not rename historical files in this list
- Do not delete earlier weeks within the same reporting year unless explicitly instructed

PRIMARY_REPORT_WEEK = 20260329_report.csv
PRIOR_REPORT_WEEK   = 20260322_report.csv
REPORT_WEEK_ENDING  = 29 March 2026
REPORT_VERSION      = Draft

WEEKLY_FILE_MANIFEST (newest first):
1. 20260329_report.csv
2. 20260322_report.csv
3. 20260315_report.csv
4. 20260308_report.csv
5. 20260301_report.csv
6. 20260228_report.csv
7. 20260222_report.csv
8. 20260215_report.csv
9. 20260208_report.csv
10. 20260201_report.csv
11. 20260131_report.csv
12. 20260125_report.csv
13. 20260118_report.csv
14. 20260111_report.csv
15. 20260104_report.csv

TARGET FILES:
16. Budget_2026.csv
17. Stretch_2026.csv

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

Currency: all figures are RM. Display as RM X,XXX for amounts under RM 500K and RM X.XM for amounts above.

Merchant naming: use merchant_group as the display name for all merchants in every table and narrative. Do not use any other name field.

Do not skip any section. Do not add sections not listed. If a section cannot be completed, include it with a clear explanation of why and what data would be needed.

Use template.html as the exact output structure for the HTML file. Fill every {{TOKEN_NAME}} placeholder with the correct value derived from the data. Do not alter the template design, layout, colours, fonts, spacing, section order, or any HTML structure. Your only job is to replace placeholders with data. If a placeholder cannot be filled due to missing data, insert 'N/A' and flag it in the chat summary.

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
- Display format:
  - RM X,XXX for amounts below RM 500K
  - RM X.XM for amounts RM 500K and above
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

Cover all 6 L3 categories in this exact order:
i.   Telco
ii.  Digital Lifestyle
iii. Online Marketplaces & Fast Fashion
iv.  Daily Essentials & Retail
v.   Everyday F&B and Lifestyle
vi.  Travel

For each L3 category, produce ALL of the following:

a. Narrative block with four labelled sub-sections
- Overview
- Scale
- Attention Needed
- Watch List

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
Do not use "N/A" unless genuinely no qualifying merchants exist.
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