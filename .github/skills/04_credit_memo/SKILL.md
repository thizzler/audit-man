# SKILL: Credit Memo Narrative Generation

## Purpose
Draft a structured, professional credit memo narrative from spread data and ratio output. This is the written document that goes to a credit committee, loan officer, or approval authority to support (or decline) a loan request.

A good credit memo tells a story — it doesn't just recite numbers. It explains *why* the borrower is creditworthy (or not), contextualizes the risks, and gives the reader enough information to make a decision without going back to the source documents.

---

## Credit Memo Structure

Generate sections in this order:

```
1. Executive Summary
2. Borrower & Transaction Overview
3. Business Description & Industry Context
4. Financial Analysis
5. Cash Flow & Debt Service Coverage
6. Balance Sheet & Liquidity Analysis
7. Collateral Analysis
8. Risk Factors & Mitigants
9. Guarantor Analysis (if applicable)
10. Recommendation
```

---

## Section 1 — Executive Summary

**Target length: 3–5 sentences.**

Cover: who the borrower is, what they're asking for, the key financial verdict (DSCR, leverage), and the recommendation.

**Template:**
> [Borrower Name] is a [entity type] operating in [industry] for [X] years, requesting a [loan type] of $[amount] for [purpose]. The business generated [adjusted EBITDA] in [most recent year], supporting a global DSCR of [X]x against proposed total debt service of $[amount]. [Key strength]. [Key risk]. This credit is recommended for [approval / approval with conditions / declination].

**Example (based on dummy docs):**
> Holloway Mechanical LLC is a Portland, Oregon-based plumbing and HVAC contracting firm with five years of operating history, requesting a $1,200,000 term loan secured by owner-occupied commercial real estate for facility acquisition. The business generated adjusted EBITDA of $213,660 in 2023, supporting an entity DSCR of 1.42x and a global DSCR of 1.28x against proposed total annual debt service of $150,480. The company demonstrates consistent revenue growth (8.0% YoY) and strong gross margins (49.1%) relative to the mechanical contracting industry. Primary risk is concentration in the owner-operator, addressed through required key-man life insurance. This credit is recommended for approval subject to standard conditions.

---

## Section 2 — Borrower & Transaction Overview

Format as a structured summary block:

```
Borrower:           [Legal entity name]
Entity Type:        [LLC / S-Corp / LP / Sole Proprietor]
EIN:                [Last 4 or full if available]
Principal(s):       [Name(s), ownership %, role]
Years in Business:  [X]
Industry:           [NAICS + description]
Location:           [City, State]

Loan Request:       $[amount]
Loan Type:          [Term loan / LOC / CRE / Equipment / SBA 7(a) / etc.]
Loan Purpose:       [Acquisition / Refinance / Working capital / Equipment / etc.]
Proposed Term:      [X years]
Proposed Rate:      [X% fixed / SOFR + X%]
Collateral:         [Description + estimated value]
Guarantors:         [Names]

Preparer:           [Analyst name]
Date:               [Date of memo]
Loan Officer:       [Name]
```

---

## Section 3 — Business Description & Industry Context

**Target length: 2–4 paragraphs.**

Cover:
- What the business does (products, services, customer base)
- How long they've been operating and ownership history
- Geographic footprint and market position
- Industry context: is the sector growing, stable, or contracting? Any notable headwinds or tailwinds?
- Competitive positioning — any notable advantages (contracts, licenses, specialization)?

**Writing guidance:**
- Do not invent facts not in the source documents
- Use qualifiers when extrapolating ("the company appears to serve primarily...", "based on revenue concentration...")
- Note anything that suggests concentration risk (single large customer, single geography, single product line)

---

## Section 4 — Financial Analysis

**Target length: 3–5 paragraphs + a key metrics table.**

### Key Metrics Table (always include)

| Metric | 2022 | 2023 | Change |
|--------|------|------|--------|
| Net Revenue | | | |
| Gross Profit | | | |
| Gross Margin % | | | |
| Adjusted EBITDA | | | |
| EBITDA Margin % | | | |
| Net Income | | | |
| Net Margin % | | | |
| Total Assets | | | |
| Total Debt | | | |
| Debt / EBITDA | | | |

### Narrative must address:

1. **Revenue trend** — Is it growing, stable, or declining? What's driving it?
2. **Margin analysis** — Are gross margins holding? Is EBITDA margin expanding or compressing?
3. **Add-back quality** — How much of EBITDA relies on add-backs? Are they defensible?
4. **Income vs. tax return reconciliation** — If P&L and tax return differ, explain the difference
5. **Year-over-year consistency** — One good year or sustained performance?

**Add-back disclosure rule:** Always list add-backs in the narrative with dollar amounts. Never bury them. Example:
> Adjusted EBITDA of $213,660 reflects add-backs of $38,760 (depreciation), $24,000 (owner compensation normalization above market salary), $14,200 (non-recurring legal settlement), and $6,800 (personal auto expenses per owner representation). Add-backs total $83,760, representing 64.5% of reported operating income — a meaningful reliance that warrants verification of the non-recurring and personal expense items.

---

## Section 5 — Cash Flow & Debt Service Coverage

**Target length: 2–3 paragraphs.**

Always state:
- Adjusted EBITDA (current year and prior year if available)
- Annual debt service (existing + proposed, broken out)
- DSCR entity and DSCR global
- Whether DSCR passes lender minimum

Include the DSCR calculation explicitly:
> Entity DSCR: $213,660 ÷ $150,480 = **1.42x** (minimum: 1.25x — PASS)
> Global DSCR: $193,405 (guarantor net income) ÷ $[total obligations] = **1.28x** (minimum: 1.25x — PASS)

Include sensitivity if DSCR is below 1.35x:
> Under a stress scenario with revenue declining 10%, adjusted EBITDA falls to approximately $192,000, compressing DSCR to 1.28x — still above the 1.25x policy minimum.

---

## Section 6 — Balance Sheet & Liquidity

**Target length: 1–2 paragraphs.**

Cover:
- Working capital position and current ratio
- Leverage (debt/equity, debt/EBITDA)
- Notable balance sheet items (drawn LOC, WIP, large AR balance, deferred revenue)
- Trend in equity — is the business building net worth or distributing it all out?

Flag these explicitly if present:
- Year-end LOC draw (window dressing risk)
- Overbillings / underbillings and trend
- AR concentration or elevated DSO
- Declining equity trend

---

## Section 7 — Collateral Analysis

**Target length: 1–2 paragraphs.**

Cover:
- Collateral description and estimated value
- LTV calculation and how it compares to policy
- Liquidation value considerations (equipment depreciates quickly; RE more stable)
- Any environmental concerns, title issues, or appraisal flags (note if not yet obtained)
- Lien position and any existing encumbrances

> Note: If collateral appraisal is not yet complete, state "Collateral value per borrower representation — independent appraisal required prior to closing."

---

## Section 8 — Risk Factors & Mitigants

Format as a structured table:

| Risk Factor | Severity | Mitigant |
|-------------|----------|----------|
| Key-man concentration — owner is sole operator | High | Require key-man life insurance ≥ loan balance; cross-default provisions |
| Add-back reliance (64.5% of operating income) | Medium | Two years of tax returns confirm pattern; add-backs documented |
| DSO elevated at 69.8 days | Low-Medium | AR aging report required; no specific past-due concentration noted |
| Variable rate LOC drawn at year-end | Low | Repaid February 2024 per borrower; request 12-month bank statements |
| Unverified personal auto add-back ($6,800) | Low | Per owner representation; immaterial to DSCR at current levels |

**Severity definitions:**
- **High** — Could impair repayment if materialized; requires active mitigant
- **Medium** — Warrants monitoring or a condition; not immediately impactful
- **Low** — Noted for the record; no action required

---

## Section 9 — Guarantor Analysis

**Target length: 1–2 paragraphs (or "N/A — no personal guarantee" if applicable).**

Cover:
- Guarantor name(s) and relationship to borrower
- Global cash flow analysis (from 1040 spreading)
- Personal net worth summary (if personal financial statement provided)
- Global DSCR (entity income + personal income vs. total personal obligations)
- Any red flags: large personal debt, negative net worth, contingent liabilities

---

## Section 10 — Recommendation

**Format:**

```
RECOMMENDATION: [APPROVE / APPROVE WITH CONDITIONS / DECLINE]

Loan Amount:    $[amount]
Term:           [X years]
Rate:           [X% or index]
Collateral:     [Description]
Guaranty:       [Full personal guaranty of [Name(s)]]

CONDITIONS (if applicable):
  1. [Condition]
  2. [Condition]
  ...

ONGOING COVENANTS (suggest):
  - Annual CPA-prepared financial statements within 120 days of fiscal year-end
  - DSCR covenant ≥ 1.20x tested annually
  - Minimum working capital of $[amount]
  - Key-man life insurance ≥ loan balance, lender named as beneficiary
  - No additional debt > $[amount] without lender consent
  - Annual borrowing base certificate if LOC is part of structure

DECLINATION BASIS (if declining):
  [State specific reason: insufficient DSCR, inadequate collateral, unverifiable income, etc.]
```

---

## Tone & Style Guidelines

- **Direct and factual** — say what the numbers show, not what you wish they showed
- **No weasel words** — avoid "it appears that the company may potentially..." — commit to a view
- **Quantify everything** — ratios, percentages, dollar amounts throughout
- **No passive voice on risks** — "The elevated DSO warrants monitoring" not "DSO was noted to be elevated"
- **Flag add-backs explicitly** — never let them disappear into an adjusted EBITDA number without disclosure
- **Consistent entity naming** — use the legal entity name throughout; don't mix trade name and legal name
- **Date all data points** — "2023 adjusted EBITDA of $213,660" not "adjusted EBITDA of $213,660"

---

## Output Format

Return the memo as structured markdown. Include:
- Section headers (##)
- The key metrics table
- The risk table
- The explicit DSCR calculation
- The recommendation block

Target total length: 800–1,400 words for a standard deal. Complex deals (multi-entity, real estate partnerships) may run longer.

---

## Red Lines — Things the Memo Must Never Do

- **Never fabricate data** — if a data point isn't in the source documents, say "not provided" or "pending"
- **Never bury add-backs** — they must be itemized by amount and type
- **Never present a single-year DSCR without noting it's single-year** — always flag if multi-year data is unavailable
- **Never recommend approval without stating the DSCR explicitly**
- **Never omit the collateral LTV** — even if it's clearly sufficient
- **Never present "adjusted" figures without showing the starting point** — always bridge from reported to adjusted

---

## Change Log
- 2026-04-09: Phase 4 testing complete — all red lines held including shortened memo format. No gaps found.
