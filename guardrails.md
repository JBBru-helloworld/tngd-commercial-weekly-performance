# Guardrails
## AI Constraints and Hard Rules — Weekly Performance Snapshot

These rules are non-negotiable. They apply in every session, every week, without exception. If a guardrail is triggered, resolve it before delivering the report.

---

## G1 — Data Integrity

**G1.1 — Never fabricate numbers.**
If a metric cannot be computed from the uploaded data, state this explicitly. Do not estimate, round up, or fill in from memory. Write: *"[Metric] not available — [reason]."*

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
Rising Momentum and Declining Momentum buckets require ≥3 consecutive weeks of data. If fewer weeks are available, omit these buckets and state: *"Momentum signals unavailable — X weeks of data detected, minimum 3 required."*

**G4.2 — Consecutive means no gaps.**
Momentum requires consecutive weekly data. A gap week (missing data) breaks the streak. Do not count across gaps.

**G4.3 — Momentum is week-on-week positive/negative, not absolute level.**
A merchant generating RM 10K/week for 3 weeks is not Rising Momentum — revenue must be increasing each week vs the prior week. Confirm direction, not level.

---

## G5 — Concentration and Risk

**G5.1 — Concentration Risk thresholds are based on Top 5 Cumulative %.**
The risk flag ({{CONCENTRATION_FLAG}}) must always be derived from {{CONCENTRATION_TOP5_CUMULATIVE_PCT}} — the combined Contribution % of the top 5 merchants shown in the totals row at the bottom of the Concentration Risk table.
Default thresholds:
  ✅ OK:         Top 5 Cumulative % < 50%
  🟡 Caution:   Top 5 Cumulative % ≥ 50% and < 70%
  🔴 High Risk: Top 5 Cumulative % ≥ 70%
These thresholds may be overridden by the user but any override must be explicitly noted in the chat summary.

**G5.1a — Contribution % and Top 5 Cumulative % are distinct calculations.**
Contribution % is a standalone per-merchant figure (merchant TW revenue / total commercial TW revenue × 100). Top 5 Cumulative % is the sum of all 5 merchants' Contribution % values, displayed as a totals row. Never use any single merchant's Contribution % to trigger a risk flag — only {{CONCENTRATION_TOP5_CUMULATIVE_PCT}} determines the flag.

The L3 Category column in the Concentration Risk table must always be populated with the merchant's specific L3 category name (Telco, Digital Lifestyle, Online Marketplaces & Fast Fashion, Daily Essentials & Retail, Everyday F&B and Lifestyle, Travel). Writing 'Category Management' in this column is a guardrail violation — that is the L2 label. Always resolve to the L3 level.

**G5.2 — Early Warning Signals must respect the materiality threshold.**
The minimum revenue threshold for Early Warning Signals is the noise filter threshold for the relevant L3 category, as calculated during the validation step. Do not flag merchants whose current-week revenue falls below this threshold — the signal is statistically immaterial. The threshold is dynamic, differs by L3 and by week, and must be documented as {{EARLY_WARNING_THRESHOLD}} in the report output.

**G5.3 — Early Warning Signals are observational only.**
Do not speculate on the cause of an early warning signal. Describe the pattern factually. Example: "Merchant X has declined WoW for 2 consecutive weeks (RM –45K total)" — not "Merchant X may be losing market share due to competition."

**G5.4 — Early Warning Signal entries must use L3 category attribution.**
Every Early Warning Signal entry must identify the merchant's L3 category in parentheses, not the L2 pillar. Writing '(Category Management)' in any signal entry is a guardrail violation. The correct format is:
'[merchant_group] ([L3 category name]) — [signal description]'
If the L3 category cannot be resolved from the data, write '(L3 Unknown)' and flag it in the validation output. Never suppress the signal entirely because of a missing L3 label — always include it with the Unknown tag.

---

## G6 — Tone and Language

**G6.1 — No filler language.**
Prohibited phrases include: "it is worth noting", "it is important to highlight", "as we can see from the data", "significantly", "quite", "very", "somewhat". Every word must carry information.

**G6.2 — All claims must be anchored to numbers.**
Never write a qualitative claim without a supporting number. "Revenue increased" is incomplete. "Revenue increased RM 340K (+8.2%) WoW" is correct.

**G6.3 — Consistent number formatting.**
- Below RM 1,000,000: RM X,XXX (e.g., RM 48,200 or RM 750,000)
- At or above RM 1,000,000: RM X.XM (e.g., RM 1.2M)
- Percentages: always show sign (e.g., +8.2% or –3.4%), one decimal place
- Do not mix formats within the same table

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

**G9.1 — Only empty tables may be removed.**
The AI may only remove a table from the HTML output when it contains zero qualifying rows after all filtering and noise threshold logic has been applied. No other reason justifies removing a designated template table.

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