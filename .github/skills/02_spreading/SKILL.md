# SKILL: Financial Spreading — Field Mapping & Normalization

## Purpose
Take extracted financial data (output of the PDF Extraction skill) and normalize it into a standardized spreading schema — the same fields a commercial underwriter would populate in software like Moody's CreditLens, Buker's, or a custom credit worksheet.

This skill handles the *judgment layer*: which numbers to use, how to recast owner compensation, how to treat add-backs, and how to reconcile across multiple years or entities.

---

## Core Spreading Schema

Every spread produces these top-level buckets:

```
BORROWER PROFILE
INCOME ANALYSIS (global cash flow)
BUSINESS CASH FLOW (entity level)
BALANCE SHEET SUMMARY
DEBT SERVICE SCHEDULE
ADJUSTED EBITDA BRIDGE
SPREAD FLAGS & NOTES
```

---

## Section 1 — Borrower Profile

```
borrower.name
borrower.entity_type            ← Individual, LLC, S-Corp, LP, C-Corp
borrower.ein_or_ssn_last4
borrower.industry               ← NAICS code + description
borrower.years_in_business
borrower.ownership_structure    ← summarize % ownership per principal
borrower.guarantors[]           ← names of personal guarantors
borrower.tax_years_analyzed[]   ← e.g. [2021, 2022, 2023]
borrower.accounting_basis       ← cash or accrual
borrower.preparer               ← CPA firm name (if present)
borrower.is_audited             ← true/false
```

---

## Section 2 — Business Cash Flow (Entity Level)

Populate for each tax year analyzed. Use **accrual basis** figures wherever available.

### Revenue

```
revenue.gross_receipts          ← Top-line before returns/allowances
revenue.returns_allowances      ← Deduct
revenue.net_revenue             ← Gross - returns
revenue.cogs                    ← Deduct
revenue.gross_profit
revenue.gross_margin_pct        ← gross_profit / net_revenue
```

### Operating Expenses (spread verbatim, then recast)

```
opex.officer_compensation       ← As reported
opex.salaries_wages
opex.depreciation               ← Will be added back below
opex.interest_expense           ← Will be handled in debt service
opex.rent
opex.insurance
opex.taxes_licenses
opex.benefits
opex.advertising
opex.other
opex.total
```

### EBITDA Calculation

```
ebitda.operating_income         ← Net income + interest + taxes + D&A as reported
ebitda.add_depreciation         ← From tax return or P&L — always add back
ebitda.add_amortization         ← If present
ebitda.reported_ebitda          ← Before discretionary add-backs
```

### Discretionary Add-Backs (require underwriter judgment)

Each add-back must be documented with a rationale:

| Add-Back Type | Rule |
|---------------|------|
| **Owner compensation normalization** | If owner W-2 + draws exceed market-rate salary, add back the excess. Benchmark: use BLS or industry data for equivalent role. |
| **Depreciation / Section 179** | Always add back. Section 179 immediate expensing should be spread over useful life (typically 5–7 years for equipment). |
| **One-time / non-recurring expenses** | Add back only if clearly non-recurring and documented (legal settlements, casualty losses, etc.). |
| **Personal expenses run through business** | Add back only if supported by owner representation or clear documentation. Flag for verification. |
| **Non-cash items** | Add back (stock comp, bad debt expense if non-recurring, etc.). |

```
addbacks[].description
addbacks[].amount
addbacks[].type
addbacks[].source               ← "P&L note", "owner representation", "tax return"
addbacks[].verified             ← true/false

adjusted_ebitda                 ← reported_ebitda + sum(addbacks)
```

### Net Operating Income (for debt service)

```
noi.adjusted_ebitda
noi.less_owner_market_comp      ← Deduct reasonable market salary for owner's role
noi.less_capex_reserve          ← Deduct maintenance capex estimate (industry-specific)
noi.less_income_taxes           ← Estimate taxes on pass-through income if applicable
noi.available_for_debt_service  ← The number used in DSCR
```

---

## Section 3 — Global Cash Flow (Individual Guarantor Level)

Used when a personal guarantor is borrowing or co-signing. Pull from 1040.

```
global_cf.guarantor_name
global_cf.w2_wages              ← All W-2 income (spouse included if MFJ)
global_cf.k1_income[]           ← Per entity: name, amount, ownership %
global_cf.schedule_c_income     ← After SE tax adjustment
global_cf.schedule_e_income     ← Net rental (after passive loss limitations)
global_cf.capital_gains         ← Note if recurring vs. one-time
global_cf.other_income
global_cf.total_gross_income

global_cf.less_personal_taxes   ← Estimate from 1040 actual liability
global_cf.less_personal_debt_service  ← From credit report / loan application
global_cf.less_living_expense   ← Policy-defined (e.g. $2,500/mo minimum)
global_cf.net_personal_cash_flow
```

---

## Section 4 — Balance Sheet Summary

Populate for most recent year-end. Note comparative if available.

```
bs.as_of_date
bs.cash_and_equivalents
bs.accounts_receivable_net
bs.inventory
bs.wip_net                      ← Costs in excess minus billings in excess
bs.other_current_assets
bs.total_current_assets

bs.ppe_net
bs.other_long_term_assets
bs.total_assets

bs.accounts_payable
bs.accrued_liabilities
bs.drawn_loc                    ← Line of credit balance (flag if year-end draw)
bs.current_portion_ltd
bs.other_current_liabilities
bs.total_current_liabilities

bs.long_term_debt               ← Itemize per loan if possible
bs.total_liabilities
bs.total_equity

bs.working_capital              ← current assets - current liabilities
bs.tangible_net_worth           ← equity minus intangibles
```

---

## Section 5 — Debt Service Schedule

List all existing debt obligations. Used to calculate DSCR.

```
debt_obligations[]
  .description                  ← lender + collateral description
  .original_balance
  .current_balance
  .monthly_payment
  .annual_payment               ← monthly × 12
  .interest_rate
  .rate_type                    ← fixed / variable
  .maturity_date
  .collateral
  .is_proposed                  ← true if this is the new loan being underwritten

total_annual_debt_service       ← sum of all annual_payment values
proposed_annual_debt_service    ← just the new loan
existing_annual_debt_service    ← all other loans
```

---

## Section 6 — Multi-Year Trend

If 2+ years of data are available, populate:

```
trend[]                         ← Array per year
  .year
  .net_revenue
  .gross_margin_pct
  .adjusted_ebitda
  .net_income
  .total_assets
  .total_debt
  .debt_to_equity

trend.revenue_cagr              ← Compound annual growth rate over period
trend.ebitda_cagr
trend.direction                 ← "improving" | "declining" | "stable"
trend.notes                     ← Narrative on any inflection points
```

---

## Recasting Rules by Entity Type

### Sole Proprietor / Schedule C
- Owner's total compensation = W-2 from other sources (if any) + Schedule C net profit + SE tax deduction add-back
- Do NOT double-count: if owner draw is already reflected in Schedule C expenses, don't add it back again
- SE tax deduction (half of SE tax) reduces AGI — add it back to get true economic income

### S-Corporation (1120S)
- Spread ordinary business income PLUS officer W-2 wages (both flow to shareholder)
- K-1 distributions are return of capital / tax preference — not additional income
- Section 179 add-back: spread over 5 years for equipment, 7 years for other property
- Watch for below-market officer compensation (IRS scrutiny issue AND cash flow understatement)

### Partnership (1065)
- Guaranteed payments to GP/managing partner are operating expenses on the return but cash income to the recipient — treat as compensation, not debt service
- Limited partner K-1 income is passive unless LP materially participates — confirm with borrower
- Depreciation on real estate is typically large — always add back

### Real Estate / Rental Income
- Use Schedule E net income after all expenses including depreciation
- Add back depreciation to get cash-basis income
- Suspended passive losses: do NOT add back to current income — they are not available unless property is sold
- Variable rate mortgages: note payment sensitivity to rate changes

---

## Handling Inconsistencies Across Documents

When tax return income differs from P&L income, document the reconciliation:

```
reconciliation.doc_a            ← e.g. "Schedule C net profit: $158,750"
reconciliation.doc_b            ← e.g. "P&L net income: $115,700"
reconciliation.difference       ← $43,050
reconciliation.explanation      ← e.g. "Difference attributable to SE tax deduction
                                   ($10,035), home office ($8,400), and timing
                                   differences in accrual vs. cash recognition ($24,615)"
reconciliation.use_for_spreading← "tax_return" | "pl" | "average" | "conservative"
```

**Default rule: use the lower (more conservative) figure unless there is a documented, underwriter-approved reason to use a higher number.**

---

## Output Format

```json
{
  "borrower": { ... },
  "years_spread": [2021, 2022, 2023],
  "business_cash_flow": {
    "2023": { ... },
    "2022": { ... }
  },
  "global_cash_flow": { ... },
  "balance_sheet": { ... },
  "debt_service": { ... },
  "trend": { ... },
  "spread_flags": [ ... ],
  "underwriter_notes": ""
}
```

---

## Spread Flags (inherit from extraction + add spreading-level flags)

| Flag | Meaning |
|------|---------|
| `revenue_declining` | Year-over-year revenue decrease >10% |
| `ebitda_declining` | EBITDA trend negative over analysis period |
| `heavy_addback_reliance` | Add-backs >25% of reported EBITDA |
| `below_market_officer_comp` | Officer W-2 significantly below market rate |
| `unverified_addbacks` | One or more add-backs based on owner representation only |
| `multi_entity_complexity` | Income streams across 3+ entities |
| `passive_loss_limitations` | Rental income restricted by passive activity rules |
| `concentrated_revenue` | Single customer or project likely >25% of revenue |
| `wip_volatility` | Large swings in costs/billings in excess year-over-year |
| `guarantor_negative_cash_flow` | Personal global cash flow is negative |
| `debt_heavy` | Total debt > 3x adjusted EBITDA |
| `single_year_data` | Only one tax year provided — multi-year trend analysis not possible |
| `leverage_elevated_proposed` | Debt/EBITDA on proposed basis (existing + new loan) exceeds 4.5x |

---

## Leverage Calculation Rule — Existing vs. Proposed Basis

Always compute leverage ratios on **both** bases and label clearly:
- **Current basis:** existing debt only (pre-loan)
- **Proposed basis:** existing debt + proposed loan amount

If proposed-basis leverage exceeds thresholds but the loan is collateral-supported (e.g., CRE acquisition where the asset will add to the balance sheet), note this context rather than auto-failing. The scorecard should show FLAG with explanation, not FAIL.

---

## Change Log
- 2026-04-09: Added `single_year_data` and `leverage_elevated_proposed` flags to spread flags table after Test 2B/2C revealed missing triggers
- 2026-04-09: Added "Leverage Calculation Rule" section requiring both current-basis and proposed-basis computation with clear labeling
