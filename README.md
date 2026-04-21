# Weekly Performance Snapshot System
### AI-Powered Commercial Revenue Commentary

---

## What This Is

This system enables an AI assistant to ingest a weekly revenue data file and generate a structured, executive-ready performance snapshot. It replaces manual commentary with consistent, data-driven narrative across four report sections: executive header, overall revenue performance, risky business (concentration risk, early warning signals, DDNQR), and category deep dive across 7 L3 categories.

---

## File Inventory

| File | Description |
|---|---|
| `README.md` | This file. Project overview, setup, and usage guide. |
| `instructions_full.md` | Complete AI knowledge base. Upload to AI Project knowledge. |
| `instructions_condensed.md` | Short instruction set. Paste into AI Project "Instructions" field. |
| `ignition_prompt.md` | Opening prompt the user pastes at the start of each weekly session. |
| `template.html` | Outlook-safe HTML report template with `{{PLACEHOLDERS}}`. |
| `template_schema.md` | Defines every `{{PLACEHOLDER}}` token, its source, and expected format. |
| `data_validation.md` | Pre-analysis checklist the AI must run before generating any output. |
| `changelog_format.md` | Standard format for the changelog block appended to every report. |
| `guardrails.md` | Hard rules, constraints, and failure modes the AI must always respect. |

---

## Report Structure

The generated report covers four sections:

1. **Main Header** — executive summary: overall performance vs budget/stretch, primary L2 driver, primary risk, one forward signal.
2. **Overall Revenue Performance** — Table 1A (vs targets) and Table 1B (WoW by L2 pillar: SME, Mobility, Government Services, Category Management, Crossborder excl. eSIM, eSIM, Foreign Worker Segment).
3. **Risky Business** — Concentration Risk (top 5 merchant table), Early Warning Signals (up to 10 merchants), Global DDNQR Top 10 table + DDNQR Penetration Tracker.
4. **Category Deep Dive** — L2 "04 Category Management" only. Seven L3 categories in fixed order: Telco Prepaid, Mobile Internet, Digital Lifestyle, Online Marketplaces & Fast Fashion, Daily Essentials & Retail, Everyday F&B and Lifestyle, Travel. Each L3 includes: narrative, Top Merchants table, and up to 7 bucket tables (Winners, Losers, Rising Momentum, Declining Momentum, Reactivated, New Entrants, DDNQR Top 5).

---

## Key System Features

- **DDNQR isolated calculation path:** Three discrete computation objects (ddnqr_global_revenue, ddnqr_l3_revenue, ddnqr_penetration_tpv) with a mandatory audit block before HTML generation.
- **Petrol merchant exclusion:** Petronas, Shell, Caltex, Petron, BHPetrol are excluded from all Daily Essentials ranked tables; revenue retained in totals.
- **eSIM extraction:** eSIM shown as a standalone row in Table 1B, extracted from Crossborder; Total Commercial unchanged.
- **Noise filter:** Fixed RM 12,500 for Telco Prepaid and Mobile Internet. Dynamic (lower of 1% L3 weekly revenue or 25th percentile) for the other 5 L3 categories.
- **Mojibake detection:** 4-pass decoding pipeline (Latin-1, Windows-1252, double-encoding, partial decode) with Tier 1 and Tier 2 pattern detection.
- **MTD rollover handling:** Automatic month-boundary detection with prior-month days excluded from MTD but included in YTD.

---

## How to Use

### Step 1 — Set up the AI Project
1. Create a new AI Project in your preferred AI assistant platform
2. Under **Instructions**, paste the contents of `instructions_condensed.md`
3. Under **Knowledge**, upload all of the following:
   - `instructions_full.md`
   - `template.html`
   - `template_schema.md`
   - `data_validation.md`
   - `guardrails.md`
   - `changelog_format.md`

### Step 2 — Each week: upload your data
1. Prepare your weekly data file (Excel or CSV)
2. Upload it directly into the AI Project conversation
3. Do not rename or reformat the file — let the AI infer the schema

### Step 3 — Run the ignition prompt
1. Open `ignition_prompt.md`
2. Copy the full prompt
3. Paste it as your first message in the session (after uploading the file)
4. The AI will run the data validation checklist, confirm the detected schema, then generate the full report in sequential chunks

### Step 4 — Review and export
1. Review the output against your knowledge of the week
2. If anything looks off, ask the AI to revise a specific section
3. For Outlook: download the HTML file and paste into an Outlook email in HTML compose mode

---

## AI Project Setup — Where to Paste What

```
PROJECT INSTRUCTIONS FIELD  →  instructions_condensed.md (full text)
PROJECT KNOWLEDGE UPLOADS   →  instructions_full.md
                               template.html
                               template_schema.md
                               data_validation.md
                               guardrails.md
                               changelog_format.md
EACH SESSION (user pastes)  →  ignition_prompt.md + weekly data file upload
```

---

## Processing Sequence (Chunks)

The AI processes data in 7 sequential chunks — never all at once:

| Chunk | Description |
|---|---|
| 1 | Schema detection, validation, mojibake scan, rollover/YTD checks |
| 2 | Overall L2 revenue aggregation (Table 1B + eSIM extraction) |
| 3 | YTD and MTD computation (separate passes) |
| 4 | Category Management L3 filtering and noise threshold application |
| 5 | Per-L3 merchant analysis (all 7 L3s, one at a time) |
| 6 | Global DDNQR isolated calculation (3 objects + audit block) + Risky Business |
| 7 | HTML template population (only after all chunks confirmed + audit passed) |

---

## Updating Instructions When Schema Changes

If the data file schema changes (new columns, renamed headers):
1. Open `instructions_full.md` → go to the Schema Detection section
2. Add a note describing the new column and its role
3. Re-upload `instructions_full.md` to Project Knowledge (delete old version first)
4. Update `template_schema.md` if new tokens are needed

The AI is schema-adaptive — it infers column roles from headers. Schema notes handle edge cases.

---

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| AI cannot find budget/stretch data | Column headers are ambiguous | Rename columns clearly or add a note in `instructions_full.md` |
| Momentum signals missing | Fewer than 3 weeks of data uploaded | Upload a file with at least 3 prior weeks included |
| Merchant names inconsistent | Source data has naming variations | Clean merchant names before uploading; ensure `merchant_group` column exists |
| DDNQR audit fails on TPV vs revenue check | Gross TPV and revenue field mapped to same column | Verify the Gross TPV field is distinct from the revenue field in the source data |
| DDNQR Migration % denominator wrong | AI used all-commercial TPV instead of Category Management TPV | Check guardrail G5.6 and G16 are in the uploaded guardrails.md |
| Mobile Internet merchants missing | commercial_l4 field absent or values don't match expected pattern | Confirm field exists and contains values like '05. Mobile Internet' |
| HTML template looks broken in Outlook | External CSS used | Ensure all CSS in `template.html` is inline only |
| Output structure varies week to week | Ignition prompt not used | Always start with the ignition prompt — do not freestyle |
| Petrol merchants appear in Daily Essentials tables | Petrol exclusion logic not applied | Verify guardrail G14 is in the uploaded guardrails.md |
