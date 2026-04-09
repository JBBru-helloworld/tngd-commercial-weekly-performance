# Template Schema

## Token Reference — Weekly Performance Snapshot

All placeholders in `template.html` use `{{TOKEN_NAME}}` format.
This file defines every token in JSON for machine-readability, with a legend below.

---

## Legend

| Field      | Description                                                                 |
| ---------- | --------------------------------------------------------------------------- |
| `token`    | Exact placeholder string (without the `{{` `}}`)                            |
| `source`   | Where the value comes from: `excel`, `calculated`, `ai_generated`, `system` |
| `format`   | Expected display format                                                     |
| `section`  | Which report section uses this token                                        |
| `required` | Whether the report cannot be generated without it                           |

---

## Schema JSON

```json
{
  "schema_version": "2.0",
  "last_updated": "2026-04-01",
  "currency": "RM",
  "number_formats": {
    "large": "RM X.XM (at or above 1,000,000)",
    "small": "RM X,XXX (below 1,000,000)",
    "variance_positive": "+RM X.XM or +X.X%",
    "variance_negative": "-RM X.XM or -X.X%",
    "percentage": "±X.X% (always show sign, one decimal)"
  },

  "tokens": {
    "header": [
      {
        "token": "WEEK_ENDING",
        "source": "excel",
        "format": "DD Month YYYY",
        "required": true,
        "notes": "Last day of the reporting week"
      },
      {
        "token": "REPORT_DATE",
        "source": "system",
        "format": "DD Month YYYY",
        "required": true,
        "notes": "Date the report was generated"
      },
      {
        "token": "REPORT_VERSION",
        "source": "user_input",
        "format": "Draft / Final",
        "required": true,
        "notes": "Specified by user at session start"
      },
      {
        "token": "TW_DATE",
        "source": "excel",
        "format": "DD MMM YYYY",
        "required": true,
        "notes": "Week-ending date for the current reporting week (PRIMARY_REPORT_WEEK). Displayed in all TW column headers across Table 1B, all L3 Top Merchants tables, all bucket tables, and all DDNQR tables as 'TW — DD MMM YYYY'."
      },
      {
        "token": "LW_DATE",
        "source": "excel",
        "format": "DD MMM YYYY",
        "required": true,
        "notes": "Week-ending date for the prior reporting week (PRIOR_REPORT_WEEK). Displayed in all LW column headers across Table 1B, all L3 Top Merchants tables, all bucket tables, and all DDNQR tables as 'LW — DD MMM YYYY'."
      }
    ],

    "section_1_data_rules": [
      {
        "token": "MERCHANT_IDENTIFIER_FIELD",
        "value": "merchant_group",
        "source": "excel",
        "required": true,
        "notes": "Always use merchant_group as the primary display name for merchants. Do not substitute with other name fields."
      },
      {
        "token": "MOJIBAKE_DECODING",
        "value": "detect_and_correct",
        "source": "system",
        "required": true,
        "notes": "On every ingestion, scan all text columns for mojibake patterns (Ã, â€, Å, Â sequences indicating UTF-8 text misread as Latin-1). If detected, attempt auto-correction by re-decoding from Latin-1 bytes as UTF-8. Report all corrections in the validation summary with at least one before/after example. Escalate unresolvable cases to the user — do not substitute or silently skip."
      },
      {
        "rule": "YTD_ACCUMULATION",
        "window": "1 January of current year to TW_DATE inclusive",
        "method": "Sum across all available weekly data files — AI computed",
        "budget_stretch_source": "Pre-aggregated from separate budget/stretch file — read as provided, do not compute",
        "isolation_from_mtd": "Must be computed as a completely separate pass from the MTD month-boundary-filtered dataset",
        "rollover_week_behaviour": "Include full 7-day current week in YTD regardless of month boundary. Prior-month days excluded from MTD are still included in YTD.",
        "overlap_handling": "If two files cover overlapping dates, use more recent file for overlapping period. Flag in validation summary.",
        "gap_handling": "If a week between 1 Jan and TW_DATE has no file coverage, flag understatement risk. Do not estimate or interpolate.",
        "required": true
      },
      {
        "rule": "MTD_ROLLOVER_BOUNDARY",
        "trigger": "month(week_start_date) ≠ month(TW_DATE)",
        "boundary_date": "First day of month(TW_DATE)",
        "include_in_MTD": "Transaction dates >= boundary_date within the reporting week",
        "exclude_from_MTD": "Transaction dates < boundary_date within the reporting week (prior month days)",
        "applies_to": "All MTD tokens at all levels: overall, L2, L3, merchant",
        "does_not_apply_to": "WoW, LW, TW, YTD calculations",
        "disclosure_required": true,
        "disclosure_format": "Rollover week detected: week spans [start] to [TW_DATE]. MTD includes [boundary_date] to [TW_DATE] only ([N days]). Prior month dates excluded: [list].",
        "required": true
      }
    ],

    "section_1_headline": [
      {
        "token": "HEADLINE_SUMMARY",
        "source": "ai_generated",
        "format": "plain text, max 2 sentences",
        "required": true,
        "status": "chat_only",
        "notes": "Ultra-compressed exec summary covering budget, stretch, L2 driver, L2 risk — chat summary only, no longer an HTML placeholder."
      },
      {
        "token": "FORWARD_SIGNAL",
        "source": "ai_generated",
        "format": "plain text, 1 sentence",
        "required": true,
        "status": "chat_only",
        "notes": "One forward-looking observation — chat summary only, no longer an HTML placeholder."
      }
    ],

    "section_1a_targets": [
      {
        "token": "YTD_ACTUAL",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Computed by AI: sum of revenue across all weekly files from 1 Jan to TW_DATE inclusive. Must be computed as a separate pass from MTD. Full current week included regardless of rollover week status. Check for file overlaps and gaps before computing."
      },
      {
        "token": "YTD_BUDGET",
        "source": "excel",
        "format": "RM X.XM",
        "required": true,
        "notes": "Pre-aggregated: read directly from budget/stretch file as provided. Do not recompute or adjust."
      },
      {
        "token": "YTD_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": true,
        "notes": "Computed by AI: sum of revenue across all weekly files from 1 Jan to TW_DATE inclusive. Must be computed as a separate pass from MTD. Full current week included regardless of rollover week status. Check for file overlaps and gaps before computing."
      },
      {
        "token": "YTD_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": true,
        "notes": "Computed by AI: sum of revenue across all weekly files from 1 Jan to TW_DATE inclusive. Must be computed as a separate pass from MTD. Full current week included regardless of rollover week status. Check for file overlaps and gaps before computing."
      },
      {
        "token": "YTD_STRETCH",
        "source": "excel",
        "format": "RM X.XM",
        "required": false,
        "notes": "YTD stretch target. If unavailable, omit YTD vs Stretch row and flag. Pre-aggregated: read directly from budget/stretch file as provided. Do not recompute or adjust."
      },
      {
        "token": "YTD_STR_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": false,
        "notes": "YTD Actual minus YTD Stretch target. Computed by AI: sum of revenue across all weekly files from 1 Jan to TW_DATE inclusive. Must be computed as a separate pass from MTD. Full current week included regardless of rollover week status. Check for file overlaps and gaps before computing."
      },
      {
        "token": "YTD_STR_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": false,
        "notes": "YTD variance vs Stretch as a percentage. Computed by AI: sum of revenue across all weekly files from 1 Jan to TW_DATE inclusive. Must be computed as a separate pass from MTD. Full current week included regardless of rollover week status. Check for file overlaps and gaps before computing."
      },
      {
        "token": "MTD_ACTUAL",
        "source": "excel",
        "format": "RM X.XM",
        "required": true,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_BUDGET",
        "source": "excel",
        "format": "RM X.XM",
        "required": true,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": true,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": true,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_STRETCH",
        "source": "excel",
        "format": "RM X.XM",
        "required": false,
        "notes": "If YTD Stretch or MTD Stretch targets are unavailable, omit those rows and flag. MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_STR_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": false,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      },
      {
        "token": "MTD_STR_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": false,
        "notes": "MTD rollover rule applies: in weeks spanning two calendar months, MTD actual includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded. See MTD Month Boundary Logic in instructions_full.md."
      }
    ],

    "section_1b_wow_by_pillar": {
      "note": "One set of 4 tokens per L2 pillar. Pillars: SME, MOBILITY, GOVNT, CATMGMT, CROSSBORDER, FWS, TOTAL",
      "token_pattern": "{PILLAR}_LW / {PILLAR}_TW / {PILLAR}_VAR_ABS / {PILLAR}_VAR_PCT",
      "pillars": [
        "SME",
        "MOBILITY",
        "GOVNT",
        "CATMGMT",
        "CROSSBORDER",
        "FWS",
        "TOTAL"
      ],
      "format": "RM X.XM for LW/TW; ±RM X.XM for VAR_ABS; ±X.X% for VAR_PCT",
      "prefix": "WOW_",
      "source": "excel + calculated",
      "required": true,
      "notes": "TOTAL row uses light blue background (#dbeafe) with dark navy text (#1e3a5f)"
    },

    "section_1_narrative": [
      {
        "token": "REVENUE_NARRATIVE",
        "source": "ai_generated",
        "format": "3-5 sentences",
        "required": false,
        "status": "removed",
        "notes": "Removed — Revenue Narrative block removed from template.html as of April 2026."
      }
    ],

    "section_2_noise_filter": [
      {
        "token": "NOISE_THRESHOLD_VALUE",
        "source": "calculated",
        "format": "X,XXX",
        "required": false,
        "status": "deprecated",
        "notes": "Replaced by per-L3 threshold tokens. Do not use."
      },
      {
        "token": "NOISE_THRESHOLD_METHOD",
        "source": "ai_generated",
        "format": "plain text",
        "required": false,
        "status": "deprecated",
        "notes": "Replaced by per-L3 threshold tokens. Do not use."
      }
    ],

    "section_3_l3_noise_thresholds": [
      {
        "token": "TELCO_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Telco L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      },
      {
        "token": "DL_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Digital Lifestyle L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      },
      {
        "token": "MKTPL_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Online Marketplaces & Fast Fashion L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      },
      {
        "token": "DAILY_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Daily Essentials & Retail L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      },
      {
        "token": "FNB_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Everyday F&B and Lifestyle L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      },
      {
        "token": "TRAVEL_NOISE_THRESHOLD",
        "source": "calculated",
        "format": "RM X,XXX",
        "required": true,
        "notes": "Dynamic noise filter threshold for Travel L3 category. Calculated as the lower of: 1% of total L3 weekly revenue or the 25th percentile of active merchant weekly revenues within this L3."
      }
    ],

    "section_2_l3_blocks": {
      "note": "Identical token structure for all 6 L3 categories. Replace PREFIX with: TELCO, DL, MKTPL, DAILY, FNB, TRAVEL",
      "l3_categories": [
        { "prefix": "TELCO", "full_name": "Telco", "anchor": "l3-telco" },
        {
          "prefix": "DL",
          "full_name": "Digital Lifestyle",
          "anchor": "l3-digital-lifestyle"
        },
        {
          "prefix": "MKTPL",
          "full_name": "Online Marketplaces & Fast Fashion",
          "anchor": "l3-marketplaces"
        },
        {
          "prefix": "DAILY",
          "full_name": "Daily Essentials & Retail",
          "anchor": "l3-daily-essentials"
        },
        {
          "prefix": "FNB",
          "full_name": "Everyday F&B and Lifestyle",
          "anchor": "l3-fnb"
        },
        { "prefix": "TRAVEL", "full_name": "Travel", "anchor": "l3-travel" }
      ],
      "narrative_tokens": [
        {
          "token": "{PREFIX}_NARRATIVE",
          "source": "ai_generated",
          "format": "2-3 sentences, no merchant names",
          "required": true
        },
        {
          "token": "{PREFIX}_SCALE",
          "source": "ai_generated",
          "format": "1 sentence — rising momentum narrative",
          "required": false,
          "status": "removed",
          "notes": "Removed — narrative condensed to Overview only."
        },
        {
          "token": "{PREFIX}_ATTENTION",
          "source": "ai_generated",
          "format": "1 sentence — biggest losers / declining momentum",
          "required": false,
          "status": "removed",
          "notes": "Removed — narrative condensed to Overview only."
        },
        {
          "token": "{PREFIX}_WATCHLIST",
          "source": "ai_generated",
          "format": "1 sentence — reactivated and new entrants",
          "required": false,
          "status": "removed",
          "notes": "Removed — narrative condensed to Overview only."
        }
      ],
      "top_merchants_table": {
        "sort": "YTD descending (primary), rows 1-5 plus Total L3 row",
        "token_pattern": "{PREFIX}_M{N}_{COL} where N=1-5, COL = NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
        "total_row_pattern": "{PREFIX}_TOTAL_{COL} where COL = YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
        "source": "excel + calculated",
        "mtd_note": "MTD rollover rule applies to all MTD values in this table: in weeks spanning two calendar months, MTD includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded at merchant and L3 total level.",
        "ytd_note": "YTD values in this table are AI-computed sums across all weekly files from 1 Jan to TW_DATE inclusive, computed as a separate pass from MTD. Full current week always included."
      },
      "bucket_tables": {
        "note": "All buckets: Top 5 merchants. Token pattern: {PREFIX}_{BUCKET}{N}_{COL}",
        "mtd_note": "MTD rollover rule applies to all MTD values in bucket tables: in weeks spanning two calendar months, MTD includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded at merchant level.",
        "ytd_note": "YTD values in bucket tables are AI-computed sums across all weekly files from 1 Jan to TW_DATE inclusive, computed as a separate pass from MTD. Full current week always included.",
        "buckets": [
          {
            "code": "WIN",
            "label": "Biggest Winners",
            "sort": "primary: WoW RM descending | secondary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT"
          },
          {
            "code": "LOSE",
            "label": "Biggest Losers",
            "sort": "primary: WoW RM ascending | secondary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT"
          },
          {
            "code": "RISE",
            "label": "Rising Momentum",
            "eligibility": "3+ consecutive weeks of WoW growth (week N > week N-1 for 3+ consecutive weeks)",
            "sort": "primary: consecutive week count descending | secondary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
            "unavailable_if": "fewer than 3 weeks of data"
          },
          {
            "code": "DEC",
            "label": "Declining Momentum",
            "eligibility": "3+ consecutive weeks of WoW decline (week N < week N-1 for 3+ consecutive weeks)",
            "sort": "primary: consecutive week count descending | secondary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
            "unavailable_if": "fewer than 3 weeks of data"
          },
          {
            "code": "REACT",
            "label": "Reactivated",
            "eligibility": "transaction in current week BUT was inactive (zero revenue) in prior week(s)",
            "sort": "primary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT"
          },
          {
            "code": "NEW",
            "label": "New Entrants",
            "eligibility": "lifetime transaction count = 0 before this week, but this week > 0",
            "sort": "primary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT"
          },
          {
            "code": "DDNQR",
            "label": "DDNQR Top 5",
            "eligibility": "Merchants filtered by MID = EP142731 within the L3 category scope",
            "sort": "primary: YTD descending",
            "columns": "NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
            "token_pattern": "{PREFIX}_DDNQR{N}_{COL} where N=1-5",
            "note": "Apply MID filter EP142731 first, then filter to L3 category, then sort by YTD descending. Use merchant_group as the display name."
          }
        ]
      }
    },

    "section_2_ddnqr_global": {
      "note": "Top 10 DDNQR merchants across all Commercial L2 pillars. Filter: MID = EP142731. Sort: YTD descending.",
      "token_pattern": "DDNQR_GLOBAL_R{N}_{COL} where N=1-10, COL = NAME|YTD|MTD|LW|TW|WOW_RM|WOW_PCT",
      "source": "excel + calculated",
      "required": true,
      "mtd_note": "MTD rollover rule applies to all MTD values in this table: in weeks spanning two calendar months, MTD includes only current-month days (from 1st of TW month to TW_DATE). Prior-month days within the reporting week are excluded at merchant level.",
      "ytd_note": "YTD values in this table are AI-computed sums across all weekly files from 1 Jan to TW_DATE inclusive, computed as a separate pass from MTD. Full current week always included."
    },

    "section_3_risky_business": [
      {
        "token": "VOLATILE_MOVERS",
        "source": "ai_generated",
        "format": "narrative list",
        "required": false,
        "status": "removed",
        "notes": "Removed — Volatile Movers section removed from template.html as of April 2026."
      },
      {
        "token": "CONCENTRATION_FLAG",
        "source": "ai_generated",
        "format": "emoji + label",
        "required": true,
        "notes": "✅ OK | 🟡 Caution (top 3 > 50%) | 🔴 High Risk (top 5 > 70%)"
      },
      {
        "token": "CONC_M1_NAME",
        "source": "calculated",
        "format": "plain text",
        "required": true,
        "notes": "Rank 1 merchant by weekly revenue across all commercial L2 pillars"
      },
      {
        "token": "CONC_M1_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Rank 1 merchant current week revenue"
      },
      {
        "token": "CONCENTRATION_R1_CONTRIBUTION_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Standalone contribution of rank 1 merchant to total commercial TW revenue. Formula: merchant TW revenue / total commercial TW revenue × 100. Independent per-merchant figure — does not accumulate."
      },
      {
        "token": "CONC_M2_NAME",
        "source": "calculated",
        "format": "plain text",
        "required": true,
        "notes": "Rank 2 merchant by weekly revenue across all commercial L2 pillars"
      },
      {
        "token": "CONC_M2_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Rank 2 merchant current week revenue"
      },
      {
        "token": "CONCENTRATION_R2_CONTRIBUTION_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Standalone contribution of rank 2 merchant to total commercial TW revenue. Formula: merchant TW revenue / total commercial TW revenue × 100. Independent per-merchant figure — does not accumulate."
      },
      {
        "token": "CONC_M3_NAME",
        "source": "calculated",
        "format": "plain text",
        "required": true,
        "notes": "Rank 3 merchant by weekly revenue across all commercial L2 pillars"
      },
      {
        "token": "CONC_M3_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Rank 3 merchant current week revenue"
      },
      {
        "token": "CONCENTRATION_R3_CONTRIBUTION_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Standalone contribution of rank 3 merchant to total commercial TW revenue. Formula: merchant TW revenue / total commercial TW revenue × 100. Independent per-merchant figure — does not accumulate."
      },
      {
        "token": "CONC_M4_NAME",
        "source": "calculated",
        "format": "plain text",
        "required": true,
        "notes": "Rank 4 merchant by weekly revenue across all commercial L2 pillars"
      },
      {
        "token": "CONC_M4_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Rank 4 merchant current week revenue"
      },
      {
        "token": "CONCENTRATION_R4_CONTRIBUTION_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Standalone contribution of rank 4 merchant to total commercial TW revenue. Formula: merchant TW revenue / total commercial TW revenue × 100. Independent per-merchant figure — does not accumulate."
      },
      {
        "token": "CONC_M5_NAME",
        "source": "calculated",
        "format": "plain text",
        "required": true,
        "notes": "Rank 5 merchant by weekly revenue across all commercial L2 pillars"
      },
      {
        "token": "CONC_M5_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Rank 5 merchant current week revenue"
      },
      {
        "token": "CONCENTRATION_R5_CONTRIBUTION_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Standalone contribution of rank 5 merchant to total commercial TW revenue. Formula: merchant TW revenue / total commercial TW revenue × 100. Independent per-merchant figure — does not accumulate."
      },
      {
        "token": "CONCENTRATION_TOP5_TOTAL_REV",
        "source": "calculated",
        "format": "RM X.XM",
        "required": true,
        "notes": "Combined weekly revenue of the top 5 merchants. Displayed in the Weekly Rev (RM) column of the totals row. Formula: sum of CONC_M1_REV through CONC_M5_REV."
      },
      {
        "token": "CONCENTRATION_TOP5_CUMULATIVE_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Total Contribution % of the top 5 merchants combined. Displayed as a totals row at the bottom of the Concentration Risk table. This is the primary concentration risk signal: Flag 🟡 Caution if ≥50%, 🔴 High Risk if ≥70%. Formula: sum of CONCENTRATION_R1 through R5 CONTRIBUTION_PCT."
      },
      {
        "token_pattern": "CONCENTRATION_R{N}_L3",
        "source": "excel",
        "format": "L3 category name",
        "required": true,
        "notes": "The specific L3 category the merchant belongs to (e.g. Telco, Digital Lifestyle, Online Marketplaces & Fast Fashion, Daily Essentials & Retail, Everyday F&B and Lifestyle, Travel). Do not use the L2 pillar name 'Category Management' here. N = 1–5 matching the row rank."
      },
      {
        "token": "EARLY_WARNINGS",
        "source": "ai_generated",
        "format": "narrative list",
        "required": true,
        "notes": "2 consecutive declines, >20% single-week drop, re-entry after 3+ weeks absent"
      },
      {
        "token": "EARLY_WARNING_THRESHOLD",
        "source": "calculated",
        "format": "X,XXX",
        "required": true,
        "notes": "Minimum weekly revenue threshold below which merchants are excluded from Early Warning Signals. Displayed as subtitle in HTML template."
      }
    ],

    "section_4_actions": [
      {
        "token": "SCALE_ACTIONS",
        "source": "ai_generated",
        "format": "narrative with named merchants + RM upside",
        "required": false,
        "status": "removed",
        "notes": "Section 5 Opportunities and Actions removed from report."
      },
      {
        "token": "FIX_ACTIONS",
        "source": "ai_generated",
        "format": "narrative with named merchants + magnitude",
        "required": false,
        "status": "removed",
        "notes": "Section 5 Opportunities and Actions removed from report."
      },
      {
        "token": "MONITOR_WATCHLIST",
        "source": "ai_generated",
        "format": "narrative with named merchants + signal",
        "required": false,
        "status": "removed",
        "notes": "Section 5 Opportunities and Actions removed from report."
      }
    ],

    "changelog": [
      {
        "token": "CHANGELOG_DATE",
        "source": "system",
        "format": "DD Month YYYY",
        "required": true,
        "status": "chat_only",
        "notes": " — changelog is chat-only, not an HTML placeholder."
      },
      {
        "token": "NEW_MERCHANTS",
        "source": "ai_detected",
        "format": "list or 'None'",
        "required": true,
        "status": "chat_only",
        "notes": " — changelog is chat-only, not an HTML placeholder."
      },
      {
        "token": "DATA_FLAGS",
        "source": "ai_detected",
        "format": "list or 'None'",
        "required": true,
        "status": "chat_only",
        "notes": " — changelog is chat-only, not an HTML placeholder."
      },
      {
        "token": "MERCHANT_ATTRIBUTION_NOTE",
        "source": "ai_generated",
        "format": "Standard disclosure wording or 'Full attribution — no unresolved rows detected.'",
        "required": true,
        "status": "chat_only",
        "notes": "Chat changelog only — NEVER in the HTML file. If unresolved merchant-attributable TW revenue > RM 0, insert: 'Merchant-ranked signals exclude unresolved rows worth RM X (Y% of TW commercial revenue).' Otherwise insert: 'Full attribution — no unresolved rows detected.'"
      }
    ],

    "quick_links": [
      {
        "token": "DASHBOARD_LINK_COMMERCIAL",
        "source": "user_input",
        "format": "URL",
        "required": true,
        "notes": "URL for the Commercial Dashboard. Appears in Quick Links section above Footer."
      },
      {
        "token": "DASHBOARD_LINK_MERCHANT",
        "source": "user_input",
        "format": "URL",
        "required": true,
        "notes": "URL for the Merchant Search Dashboard. Appears in Quick Links section above Footer."
      }
    ],

    "legend": [
      {
        "section": "Legend",
        "static": true,
        "notes": "The Legend block is located between the Quick Links section and the Footer. All definition rows are static and must never be modified, reordered, or removed. The two dynamic tokens TW_DATE and LW_DATE within the legend must be populated with the correct week-ending dates for the current and prior report weeks.",
        "definitions": [
          "TW — This Week: revenue for the current reporting week ending {{TW_DATE}}",
          "LW — Last Week: revenue for the prior reporting week ending {{LW_DATE}}",
          "WoW — Week-on-Week: change from LW to TW (RM and %)",
          "YTD — Year to Date: cumulative revenue from 1 January through the report week-ending date",
          "MTD — Month to Date: cumulative revenue from the 1st of the report month through the report week-ending date",
          "Contribution % — this merchant's TW revenue as a percentage of total commercial TW revenue",
          "Top 5 Cumulative % — sum of Contribution % for the top 5 merchants in the Concentration Risk table; the primary concentration risk signal",
          "Noise Filter — minimum weekly revenue threshold applied per L3 category to exclude immaterial merchants from bucket tables",
          "DDNQR — merchants using Dynamic Duitnow QR (MID = EP142731); shown in a dedicated table per L3 and globally",
          "Rising Momentum — merchants with 3 or more consecutive weeks of WoW revenue growth",
          "Declining Momentum — merchants with 3 or more consecutive weeks of WoW revenue decline",
          "Reactivated — merchants with TW revenue > 0 and LW revenue = 0 exactly",
          "New Entrants — merchants with no revenue recorded in any prior week across the full combined dataset before TW"
        ]
      }
    ]
  }
}
```
