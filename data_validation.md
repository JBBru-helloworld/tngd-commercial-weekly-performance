# Data Validation Checklist
## Pre-Analysis Checks — Weekly Performance Snapshot

The AI must run every check in this list before generating any section of the report. Surface all findings to the user before proceeding. Do not skip or assume.

---

## How to Report Validation Results

Present findings in this format before generating the report:

```
DATA VALIDATION SUMMARY
───────────────────────
Schema Mapping:       [PASS / WARNINGS]
Data Completeness:    [PASS / FAIL]
Date Coverage:        [X weeks detected]
Momentum Eligibility: [ELIGIBLE / INSUFFICIENT DATA]
Noise Filter:         [Threshold: RM X,XXX | Method: X]
Quality Flags:        [X issues found]

[Detail each warning or flag below]
```

Await user confirmation before proceeding.

---

## Checklist

### A — Schema Detection

- [ ] **Column mapping complete.** All required logical roles have been mapped to a column: merchant identifier, L2/L3/L4 hierarchy, current week revenue, prior week revenue, YTD actual, MTD actual, budget (YTD or MTD), stretch target.
- [ ] **No ambiguous columns.** Every column has been assigned a clear role. If a column's role is uncertain, flag it explicitly and ask the user.
- [ ] **merchant_group field confirmed as primary merchant identifier.** If absent or inconsistent across weeks, flag before proceeding. Do not use any other name field as the display name.
- [ ] **MID filter field confirmed.** Verify the data contains a column that can be filtered for MID = EP142731 to identify DDNQR merchants. If this column is absent or the value EP142731 returns zero rows, flag before generating the DDNQR tables and insert 'No DDNQR merchants found' in the relevant table placeholders.
- [ ] **Category hierarchy confirmed.** L2, L3, and L4 levels are distinguishable in the data (separate columns or encoded in a single field). Confirm which applies.
- [ ] **L2 scope confirmed.** The data contains rows classifiable as "04 Category Management." If the L2 filter yields no rows, stop and flag this before proceeding.
- [ ] **L3 category resolvable for all merchants flagged in Early Warning Signals and Concentration Risk table.** For every merchant that will appear in either the Early Warning Signals block or the Concentration Risk table, confirm their L3 category can be looked up from the data hierarchy. If any merchant's L3 category is unresolvable, flag the specific merchant name and the affected output section in the validation summary before generating the report.
- [ ] **Mojibake scan complete.** Scan all text columns — particularly merchant_group, category hierarchy fields, and any label columns — for mojibake sequences (e.g. Ã, â€, Å, Â followed by unexpected characters). These result from UTF-8 text being misread as Latin-1. If mojibake is detected: attempt automatic correction by treating the affected string as Latin-1 bytes and re-decoding as UTF-8; report the number of rows corrected and at least one before/after example in the validation summary; flag any rows where the correction attempt fails or produces still-garbled output.

---

### B — Data Completeness

- [ ] **Revenue column has no blanks.** If blanks exist in the current week revenue column, count them and flag. Do not treat blanks as zero without user confirmation.
- [ ] **Budget and stretch present.** If either is absent, note which sections will be partially generated.
- [ ] **YTD and MTD columns present.** If absent, note whether they can be computed from weekly data or must be flagged as unavailable.
- [ ] **No duplicate merchant rows.** Check for repeated merchant entries in the same week. If duplicates exist, flag them — do not silently sum or drop.
- [ ] **No negative revenue values** unless the data structure uses negatives intentionally (e.g., returns). Flag any negative values and ask the user to confirm treatment.
- [ ] **Merchant attribution coverage assessed.** Count rows where merchant_group is blank, null, or unresolvable. If any such rows exist, calculate their combined TW revenue and express as a % of total TW commercial revenue. If this value is greater than RM 0, include the standard disclosure in the validation summary: 'Merchant-ranked signals exclude unresolved rows worth RM X (Y% of TW commercial revenue).' Do not block report generation on this basis — proceed after disclosure.

---

### C — Historical Data Coverage

- [ ] **At least 1 prior week present.** Required for WoW calculations. If absent, WoW metrics cannot be generated — flag all affected sections.
- [ ] **At least 3 prior weeks present.** Required for momentum signals (Rising/Declining). If fewer than 3 weeks, state: *"Momentum signals unavailable — X weeks of data detected, minimum 3 required."*
- [ ] **Weeks are consecutive.** Check for gaps in the weekly data. A missing week breaks momentum signal calculations — flag and ask whether to interpolate or skip.
- [ ] **Week-ending dates are consistent.** All weekly data should share the same day-of-week cutoff (e.g., all Sunday or all Saturday). Flag inconsistencies.
- [ ] **MTD rollover week check.** Compare the week-start date (Monday of the reporting week) and the week-end date (TW_DATE).

  If month(week_start_date) ≠ month(TW_DATE):
    - Flag as rollover week in the validation summary
    - Identify boundary_date = first day of month(TW_DATE)
    - Confirm which transaction dates in the week fall before boundary_date (prior month — exclude from MTD) and which fall on or after boundary_date (current month — include in MTD)
    - List excluded dates explicitly: '[date]: excluded from MTD (prior month)'
    - Confirm MTD revenue at overall, L2, L3, and merchant levels has been recalculated using only current-month dates
    - Report: 'Rollover week: MTD includes [boundary_date] to [TW_DATE] only ([N days]). Excluded from MTD: [list of prior-month dates in the week].'

  If month(week_start_date) = month(TW_DATE):
    - No rollover. Standard MTD applies (1st of month to TW_DATE).
    - Report: 'No rollover week detected. Standard MTD logic applied.'

---

### D — Merchant Name Consistency

- [ ] **Merchant names are consistent across weeks.** Check for the same merchant appearing under different name variants (e.g., "7-Eleven" vs "7Eleven" vs "7-11"). Flag mismatches — do not auto-merge without user confirmation.
- [ ] **New merchant names detected.** List any merchant names in the current week not present in prior weeks. These are candidates for the "New Entrants" or "Reactivated" bucket — confirm which applies.
- [ ] **Merchants absent this week vs prior weeks.** List merchants that appeared in prior weeks but are absent this week. Flag whether they are newly inactive or were filtered by the noise threshold.

---

### E — Noise Filter Application

- [ ] **Per-L3 threshold computed.** For each of the 6 L3 categories independently, calculate: (a) 1% of that L3's total weekly revenue, and (b) the 25th percentile of active merchant weekly revenues within that L3. Apply the lower value as that L3's threshold. State each L3 threshold value separately in the validation summary.
- [ ] **Thresholds stated.** Report the exact RM value of the threshold applied for each of the 6 L3 categories and the method used.
- [ ] **Merchants excluded listed.** Provide a count of how many merchants were excluded per L3 threshold, and optionally list their names for the user's reference.
- [ ] **Thresholds are reasonable.** Sanity check: if any L3 threshold excludes >40% of merchants in that L3, flag this as potentially too aggressive and offer the user an override.

---

### F — Calculation Sanity Checks

- [ ] **Variances are directionally correct.** A positive variance vs budget means actual > budget. Confirm sign conventions are consistent throughout.
- [ ] **Percentages are based on correct denominators.** % contribution = merchant revenue / total L2 revenue (not total company revenue). Confirm scope of denominator.
- [ ] **WoW % uses prior week as base.** WoW % = (current – prior) / prior × 100. Confirm no division by zero for merchants with zero prior week revenue (these are Reactivated or New Entrants, not WoW% eligible).
- [ ] **Cumulative concentration % adds to ≤100%.** Sanity check that concentration table is internally consistent.

---

### G — Output Readiness

- [ ] **All five sections can be attempted.** Confirm which sections are fully generatable, partially generatable, or must be skipped.
- [ ] **Changelog data is ready.** Report date, week covered, new/removed merchants, and any threshold or schema changes are identified.
- [ ] **Empty table assessment complete.** For each bucket table and DDNQR table (both global Top 10 and all 6 per-L3 Top 5 tables), confirm whether qualifying rows exist after filtering and noise threshold application. List each table that will be empty in the validation summary using the format: 'Will be removed (empty): [TABLE NAME] — [L3 Category or Global]'. Confirm this assessment before generating the HTML output so removals are planned and do not create structural errors.
- [ ] **User confirmation received** before full report generation begins.