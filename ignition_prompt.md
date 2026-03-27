# Ignition Prompt
## Weekly Performance Snapshot — Session Starter

---

## How to Use

Each week, after uploading your Excel file into the ChatGPT Project conversation:
1. Upload the Excel file first
2. Then paste the prompt below as your message
3. Do not modify the prompt — it is designed to enforce consistent output structure

---

## THE IGNITION PROMPT

```
I have uploaded this week's revenue Excel file. Please generate the Weekly Performance Snapshot report.

Before generating any output, run the full data validation checklist and report back what you found:
- Schema detected (list the column roles you have mapped and to which headers)
- Data quality flags (missing values, gaps, inconsistent merchant names, insufficient weeks for momentum)
- Noise filter threshold you will apply for Section 3, and why
- Date range covered by the data
- Which sections can be fully generated vs partially generated vs skipped due to data gaps

After I confirm, generate the full report in this order:
1. Main Header
2. Overall Revenue Performance (table + narrative)
3. Category Deep Dive — L2 "04 Category Management" only
4. Risky Business
5. Opportunities & Actions
6. Changelog block

Output format: produce both versions back to back — first the Markdown version (for Word), then the HTML version (for Outlook). Label each clearly.

Currency: all figures are RM. Display as RM X,XXX for amounts under RM 500K and RM X.XM for amounts above.

Do not skip any section. Do not add sections not listed. If a section cannot be completed, include it with a clear explanation of why and what data would be needed.
```

---

## Optional Additions (append to prompt if needed)

**If you want to specify the report week manually:**
```
The report covers the week ending [DD/MM/YYYY]. Please use this as the reference date in the header and changelog.
```

**If schema has changed since last week:**
```
Note: the Excel schema has changed this week. [Describe change briefly, e.g., "The budget column is now split into Budget_YTD and Budget_MTD as separate columns."] Please re-map accordingly.
```

**If you want to override the noise filter:**
```
Override the dynamic noise filter for Section 3. Use a fixed threshold of RM [amount] weekly revenue as the minimum. Apply this consistently across all L3 and L4 analysis.
```