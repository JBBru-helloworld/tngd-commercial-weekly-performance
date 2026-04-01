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

---

## G3 — Output Structure

**G3.1 — Never skip a section without explanation.**
All five sections must appear in the HTML report. A changelog summary must appear in the chat response, not in the HTML file. If a section cannot be completed, it must still appear with a clear explanation of why and what data is needed.

**G3.2 — Never add sections not in the template.**
The five sections and changelog are the complete structure. Do not add commentary, appendices, or analysis outside this structure without explicit user instruction.

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

**G5.1 — Concentration thresholds are defaults, not absolutes.**
Default flags: 🟡 Caution if top 3 merchants > 50% of L2; 🔴 High Risk if top 5 > 70%. These can be overridden by the user but must be explicitly noted if changed.

**G5.3 — Early Warning Signals are observational only.**
Do not speculate on the cause of an early warning signal. Describe the pattern factually. Example: "Merchant X has declined WoW for 2 consecutive weeks (RM –45K total)" — not "Merchant X may be losing market share due to competition."

---

## G6 — Tone and Language

**G6.1 — No filler language.**
Prohibited phrases include: "it is worth noting", "it is important to highlight", "as we can see from the data", "significantly", "quite", "very", "somewhat". Every word must carry information.

**G6.2 — All claims must be anchored to numbers.**
Never write a qualitative claim without a supporting number. "Revenue increased" is incomplete. "Revenue increased RM 340K (+8.2%) WoW" is correct.

**G6.3 — Consistent number formatting.**
- Below RM 500K: RM X,XXX (e.g., RM 48,200)
- Above RM 500K: RM X.XM (e.g., RM 1.2M)
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