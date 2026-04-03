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
    "large": "RM X.XM (above 500,000)",
    "small": "RM X,XXX (below 500,000)",
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
      }
    ],

    "section_1_data_rules": [
      {
        "token": "MERCHANT_IDENTIFIER_FIELD",
        "value": "merchant_group",
        "source": "excel",
        "required": true,
        "notes": "Always use merchant_group as the primary display name for merchants. Do not substitute with other name fields."
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
        "source": "excel",
        "format": "RM X.XM",
        "required": true
      },
      {
        "token": "YTD_BUDGET",
        "source": "excel",
        "format": "RM X.XM",
        "required": true
      },
      {
        "token": "YTD_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": true
      },
      {
        "token": "YTD_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": true
      },
      {
        "token": "YTD_STRETCH",
        "source": "excel",
        "format": "RM X.XM",
        "required": false,
        "notes": "YTD stretch target. If unavailable, omit YTD vs Stretch row and flag."
      },
      {
        "token": "YTD_STR_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": false,
        "notes": "YTD Actual minus YTD Stretch target."
      },
      {
        "token": "YTD_STR_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": false,
        "notes": "YTD variance vs Stretch as a percentage."
      },
      {
        "token": "MTD_ACTUAL",
        "source": "excel",
        "format": "RM X.XM",
        "required": true
      },
      {
        "token": "MTD_BUDGET",
        "source": "excel",
        "format": "RM X.XM",
        "required": true
      },
      {
        "token": "MTD_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": true
      },
      {
        "token": "MTD_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": true
      },
      {
        "token": "MTD_STRETCH",
        "source": "excel",
        "format": "RM X.XM",
        "required": false,
        "notes": "If YTD Stretch or MTD Stretch targets are unavailable, omit those rows and flag."
      },
      {
        "token": "MTD_STR_VAR_ABS",
        "source": "calculated",
        "format": "±RM X.XM",
        "required": false
      },
      {
        "token": "MTD_STR_VAR_PCT",
        "source": "calculated",
        "format": "±X.X%",
        "required": false
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
        "source": "excel + calculated"
      },
      "bucket_tables": {
        "note": "All buckets: Top 5 merchants. Token pattern: {PREFIX}_{BUCKET}{N}_{COL}",
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
      "required": true
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
        "token": "CONC_M1_CUM_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Cumulative % contribution of top 1 merchant to total commercial portfolio weekly revenue"
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
        "token": "CONC_M2_CUM_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Cumulative % contribution of top 2 merchants to total commercial portfolio weekly revenue"
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
        "token": "CONC_M3_CUM_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Cumulative % contribution of top 3 merchants to total commercial portfolio weekly revenue. Flag 🟡 Caution if >50%."
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
        "token": "CONC_M4_CUM_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Cumulative % contribution of top 4 merchants to total commercial portfolio weekly revenue"
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
        "token": "CONC_M5_CUM_PCT",
        "source": "calculated",
        "format": "X.X%",
        "required": true,
        "notes": "Cumulative % contribution of top 5 merchants to total commercial portfolio weekly revenue. Flag 🔴 High Risk if >70%."
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
    ]
  }
}
```
