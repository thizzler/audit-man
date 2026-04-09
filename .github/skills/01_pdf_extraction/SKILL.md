# SKILL: Financial Document PDF Extraction

## Purpose
Extract structured financial data from tax returns (1040, 1120S, 1065), P&L statements, and balance sheets into a consistent JSON schema for downstream spreading and analysis.

## Supported Document Types
| Code | Document | Key Data |
|------|----------|----------|
| `1040` | Personal Tax Return | AGI, Schedule C/E income, SE tax, add-backs |
| `1120S` | S-Corp Return | Ordinary income, K-1 allocations, officer comp, balance sheet |
| `1065` | Partnership Return | Ordinary income, guaranteed payments, K-1s, debt schedule |
| `PL` | Profit & Loss Statement | Revenue, COGS, EBITDA, add-backs, net income |
| `BS` | Balance Sheet | Assets, liabilities, equity, working capital components |

---

## Step 1 — Identify Document Type

Before extracting, identify the doc type from headers/form numbers. Look for:
- "Form 1040" → type: `1040`
- "Form 1120-S" → type: `1120S`
- "Form 1065" → type: `1065`
- "Profit & Loss" / "Income Statement" → type: `PL`
- "Balance Sheet" → type: `BS`

If multiple docs are present, extract each separately and tag with `doc_type`.

---

## Step 2 — Extraction Rules by Document Type

### 1040 — Personal Tax Return

Target lines (map to IRS line numbers where possible):

```
income.wages                    ← W-2 wages (Line 1a)
income.business_schedule_c      ← Schedule C net profit (Line 8)
income.rental_schedule_e        ← Schedule E net income (Line 5)
income.capital_gains            ← Schedule D gains (Line 7)
income.other                    ← Other income items
income.total_income             ← Line 9
adjustments.self_employed_health← Schedule 1, Line 17
adjustments.sep_ira             ← Schedule 1, Line 16
adjustments.half_se_tax         ← Schedule 1, Line 15
adjustments.total               ← Total adjustments
agi                             ← Adjusted Gross Income
deductions.standard_or_itemized ← Line 12
deductions.qbi                  ← Sec. 199A deduction
taxable_income                  ← Line 15
tax.federal_income_tax          ← Line 24
tax.self_employment_tax         ← Schedule SE
tax.credits                     ← Total credits (itemize)
tax.total_liability             ← Net tax owed
payments.withholding            ← W-2 withholding
payments.estimated              ← 1040-ES payments
refund_or_due                   ← Amount due or refund
```

**Schedule C — extract separately if present:**
```
schedule_c.business_name
schedule_c.ein
schedule_c.gross_receipts
schedule_c.returns_allowances
schedule_c.cogs
schedule_c.gross_profit
schedule_c.total_expenses       ← itemize all expense lines
schedule_c.depreciation         ← Line 13 (key add-back)
schedule_c.net_profit
```

**Schedule E — extract per property:**
```
schedule_e[n].address
schedule_e[n].rents_received
schedule_e[n].total_expenses    ← itemize: depreciation, mortgage interest, taxes, etc.
schedule_e[n].net_income_loss
schedule_e.suspended_losses     ← Note any prior-year suspended passive losses
schedule_e.total_net            ← Sum across all properties
```

---

### 1120S — S-Corporation Return

```
corp.name
corp.ein
corp.state_of_incorporation
corp.s_election_date
corp.business_activity
corp.accounting_method          ← cash or accrual
corp.tax_year

income.gross_receipts
income.returns_allowances
income.net_revenues
income.cogs
income.gross_profit
income.other_income
income.total_income

deductions.officer_compensation ← Schedule E — extract per officer
deductions.salaries_wages
deductions.depreciation         ← Key add-back line
deductions.rent
deductions.interest_expense
deductions.taxes_licenses
deductions.benefits
deductions.pension_contributions
deductions.other
deductions.total

ordinary_business_income        ← Page 1 bottom line

schedule_k.section_179          ← Separately stated — key add-back
schedule_k.charitable_contributions
schedule_k.investment_interest
schedule_k.tax_exempt_interest
schedule_k.other_separately_stated

shareholders[]                  ← Array per shareholder
  .name
  .ownership_pct
  .w2_wages
  .k1_ordinary_income
  .k1_section_179
  .k1_other_items

balance_sheet.total_assets
balance_sheet.total_current_assets
balance_sheet.total_liabilities
balance_sheet.total_current_liabilities
balance_sheet.long_term_debt
balance_sheet.shareholder_equity
```

---

### 1065 — Partnership Return

```
partnership.name
partnership.ein
partnership.type                ← LP, LLP, GP, LLC
partnership.business_activity
partnership.accounting_method
partnership.tax_year

income.gross_receipts
income.other_income
income.total_income

deductions.management_fees
deductions.salaries_wages
deductions.guaranteed_payments  ← Separately stated on Schedule K
deductions.repairs
deductions.bad_debts
deductions.rent
deductions.taxes
deductions.interest_expense     ← Mortgage interest — key line
deductions.depreciation         ← Key add-back
deductions.other
deductions.total

ordinary_business_income

schedule_k.guaranteed_payments
schedule_k.net_1231_gain
schedule_k.unrecaptured_1250_gain
schedule_k.charitable_contributions
schedule_k.investment_interest
schedule_k.tax_exempt_interest

partners[]                      ← Array per partner
  .name
  .type                         ← GP or LP
  .ownership_pct
  .ordinary_income_allocation
  .guaranteed_payments
  .other_allocations

debt_schedule[]                 ← Extract per loan if present
  .property_or_description
  .lender
  .original_principal
  .current_balance
  .interest_rate
  .rate_type                    ← fixed or variable (note index if variable)
  .maturity_date

balance_sheet.total_assets
balance_sheet.real_estate_net
balance_sheet.total_liabilities
balance_sheet.total_current_liabilities
balance_sheet.mortgage_debt_total
balance_sheet.partners_capital
```

---

### P&L Statement

```
entity.name
entity.period_start
entity.period_end
entity.basis                    ← cash or accrual
entity.preparer
entity.is_comparative           ← true/false (prior year present)

revenue[]                       ← Array per revenue line
  .description
  .current_year
  .prior_year                   ← if comparative

cogs[]
  .description
  .current_year
  .prior_year

gross_profit.current_year
gross_profit.prior_year
gross_margin_pct.current_year
gross_margin_pct.prior_year

operating_expenses[]
  .description
  .current_year
  .prior_year

operating_income.current_year   ← EBITDA proxy before add-backs
operating_income.prior_year

addbacks[]                      ← CRITICAL — extract every add-back line
  .description
  .amount
  .type                         ← depreciation | owner_comp_normalization | one_time | personal_expense | other

adjusted_ebitda.current_year
adjusted_ebitda.prior_year

interest_expense.current_year
income_taxes.current_year
net_income.current_year
net_income.prior_year
net_margin_pct.current_year
```

---

### Balance Sheet

```
entity.name
entity.as_of_date
entity.prior_date               ← if comparative
entity.basis
entity.is_audited               ← flag if "unaudited"

assets.current.cash_total
assets.current.accounts_receivable_gross
assets.current.allowance_for_doubtful_accounts
assets.current.accounts_receivable_net
assets.current.inventory
assets.current.prepaid_expenses
assets.current.wip_costs_in_excess   ← unbilled WIP
assets.current.other
assets.current.total

assets.ppe.gross
assets.ppe.accumulated_depreciation
assets.ppe.net

assets.other[]                  ← deposits, notes receivable, etc.
assets.total

liabilities.current.accounts_payable
liabilities.current.accrued_payroll
liabilities.current.line_of_credit  ← flag drawn LOC separately
liabilities.current.billings_in_excess  ← overbillings
liabilities.current.current_portion_ltd ← current portion of long-term debt
liabilities.current.other
liabilities.current.total

liabilities.long_term[]         ← Array per loan
  .description
  .balance
liabilities.long_term.total
liabilities.total

equity.member_or_shareholder_capital
equity.retained_earnings
equity.current_year_net_income
equity.total

working_capital                 ← current assets - current liabilities
current_ratio                   ← current assets / current liabilities
quick_ratio                     ← (cash + AR) / current liabilities
debt_to_equity                  ← total liabilities / total equity
```

---

## Step 3 — Flag Anomalies During Extraction

Always note these issues in an `extraction_flags[]` array:

| Flag | Trigger |
|------|---------|
| `unaudited_statements` | P&L or BS marked "unaudited" or "management use only" |
| `cash_basis` | Accounting method is cash (limits reliability of AR/AP) |
| `prior_year_suspended_losses` | Schedule E references suspended passive losses |
| `large_allowance_for_doubtful_accounts` | Allowance > 5% of gross AR |
| `drawn_loc` | Line of credit drawn at year-end (watch for window dressing) |
| `wip_present` | Costs/billings in excess present (percentage-of-completion) |
| `variable_rate_debt` | Any debt tied to SOFR, Prime, or other index |
| `guaranteed_payments` | Partnership has guaranteed payments (affects cash flow) |
| `owner_comp_may_be_below_market` | Officer/owner W-2 comp looks low vs. business size |
| `personal_expenses_in_business` | P&L or notes reference personal expenses run through entity |
| `one_time_items` | Any non-recurring income or expense items noted |
| `multi_entity` | Borrower has income/expenses across multiple entities |

---

## Output Format

Return a single JSON object:

```json
{
  "doc_type": "1040",
  "tax_year": 2023,
  "entity_name": "Marcus T. Holloway",
  "extraction_flags": ["personal_expenses_in_business", "multi_entity"],
  "data": { ... },
  "raw_notes": "Free text for anything that didn't fit the schema"
}
```

For multiple documents in one request, return an array of these objects.

---

## Common Pitfalls

- **Schedule C net profit ≠ total business income on 1040** — adjustments like SE tax deduction and home office reduce the Schedule C figure before it hits the 1040. Extract both.
- **Schedule E suspended losses** — the number flowing to the 1040 may be smaller than actual rental net income due to passive activity rules. Note the suspended loss balance.
- **K-1 income is not W-2 income** — for S-corps and partnerships, extract officer/partner W-2 wages separately from K-1 pass-through income. Both matter for individual cash flow analysis.
- **Guaranteed payments** — these appear as a deduction on the partnership return but are ordinary income to the recipient partner. Don't double-count.
- **Section 179** — expensed immediately on the tax return but treated as a multi-year depreciation add-back for cash flow spreading purposes.
- **Billings in excess of costs (overbillings)** — a liability on the balance sheet, not deferred revenue in the traditional sense. Flag for underwriter review.
- **LOC drawn at year-end** — note balance and whether payoff occurred post-period (check notes).

---

## Change Log
- 2026-04-09: Phase 1 testing complete — all 5 doc types (1040, 1120S, 1065, PL, BS) extracted successfully. No schema gaps found.
