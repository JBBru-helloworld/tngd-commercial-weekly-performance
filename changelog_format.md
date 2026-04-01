# Changelog Format
## Week-on-Week Report Log — Weekly Performance Snapshot

Every session must end with a Changelog summary delivered in the chat response only. Do not insert this into the HTML file. Do not produce a Markdown version. This creates an audit trail and helps the team track structural changes, data quality issues, and threshold shifts over time.

---

## Format

Include the following as a chat summary at the end of your response, after delivering the HTML file. Do not insert into the HTML file:

```
---

## 📋 Report Changelog

| Field | Detail |
|---|---|
| Report Generated | {{CHANGELOG_DATE}} |
| Week Covered | Week ending {{WEEK_ENDING}} |
| Report Version | {{REPORT_VERSION}} |

### Schema & Structure
| Item | Status |
|---|---|
| Schema changes vs last week | {{SCHEMA_CHANGE_NOTES}} |
| Noise filter threshold applied | RM {{NOISE_THRESHOLD_VALUE}} ({{NOISE_THRESHOLD_METHOD}}) |
| Threshold change vs last week | {{THRESHOLD_CHANGE}} |
| Sections fully generated | {{SECTIONS_COMPLETE}} |
| Sections partially generated | {{SECTIONS_PARTIAL}} |
| Sections skipped | {{SECTIONS_SKIPPED}} |

### Merchant Changes
| Category | Merchants |
|---|---|
| New entrants (first appearance ever) | {{NEW_MERCHANTS}} |
| Reactivated (returned after absence) | {{REACTIVATED_MERCHANTS_LOG}} |
| Newly inactive (absent this week) | {{NEWLY_INACTIVE}} |
| Excluded by noise filter | {{EXCLUDED_COUNT}} merchants excluded (RM {{NOISE_THRESHOLD_VALUE}} threshold) |

### Data Quality Flags
{{DATA_FLAGS}}

*(If no flags: "No data quality issues detected this week.")*

---
```

---

## Field Guidance

**`{{SCHEMA_CHANGE_NOTES}}`**
Describe any column renaming, additions, or removals detected vs the prior session's schema. If no changes: "No schema changes detected."

**`{{SECTIONS_COMPLETE}}`**
List section numbers fully generated. Example: "Sections 1, 2, 3, 4, 5"

**`{{SECTIONS_PARTIAL}}`**
List section numbers generated with caveats and briefly describe what was missing. Example: "Section 3 — momentum signals omitted (only 2 weeks of data available)"

**`{{SECTIONS_SKIPPED}}`**
List any section skipped entirely and why. Example: "None" or "Section 2 WoW row — no prior week data in file"

**`{{REACTIVATED_MERCHANTS_LOG}}`**
Distinct from the Section 3 "Reactivated" bucket — this logs all reactivated merchants regardless of L2 scope, for audit purposes.

**`{{NEWLY_INACTIVE}}`**
Merchants present in last week's data but absent this week (after noise filter). List names or "None."

**`{{DATA_FLAGS}}`**
Bulleted list of any data quality issues encountered during validation. Examples:
- "Merchant 'ABC Trading' appears under two name variants — treated as same entity pending user confirmation"
- "Week 11 data gap detected — momentum signals calculated excluding this week"
- "3 merchants had blank current-week revenue — excluded from analysis, not treated as zero"

---

## Changelog Log (maintained manually by team)

Keep a running log of changelogs outside the AI system for governance purposes. Recommended: a simple Excel or SharePoint table with one row per report week.

| Week Ending | Report Date | Threshold Applied | New Merchants | Data Flags | Sections Complete |
|---|---|---|---|---|---|
| DD/MM/YYYY | DD/MM/YYYY | RM X,XXX | X | Y / N | 1,2,3,4,5 |