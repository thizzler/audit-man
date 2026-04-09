# SKILL: Ratio Calculation & Underwriting Analysis

## Purpose
Compute standard commercial lending ratios from spread data (output of the Spreading skill) and flag results against typical lender thresholds. This skill produces the quantitative verdict that drives credit decisions.

---

## Primary Ratios

### 1. Debt Service Coverage Ratio (DSCR)

**The most important ratio in commercial lending.**

```
DSCR = Net Operating Income Available for Debt Service
       ─────────────────────────────────────────────────
       Total Annual Debt Service (including proposed loan)
```

**Input fields:**
- Numerator: `noi.available_for_debt_service` (from Spreading skill, Section 2)
- Denominator: `debt_service.total_annual_debt_service` (all obligations, including proposed)

**Calculation variants — compute all three:**

| Variant | Description |
|---------|-------------|
| `dscr_global` | Uses global cash flow (individual guarantor level, 1040-based) |
| `dscr_entity` | Uses entity-level adjusted EBITDA only |
| `dscr_consolidated` | Blended across multiple entities if applicable |

**Thresholds (typical — confirm against lender policy):**

| DSCR | Signal |
|------|--------|
| ≥ 1.35x | Strong — well within policy |
| 1.20x – 1.34x | Acceptable — within most lender minimums |
| 1.10x – 1.19x | Marginal — often requires additional support |
| 1.00x – 1.09x | Tight — break-even; policy exception territory |
| < 1.00x | Failing — loan not self-supporting |

```json
"dscr": {
  "entity": 1.42,
  "global": 1.28,
  "consolidated": 1.38,
  "threshold_minimum": 1.25,
  "pass": true,
  "notes": "Entity DSCR strong; global DSCR slightly compressed by personal debt obligations"
}
```

---

### 2. Loan-to-Value (LTV)

```
LTV = Proposed Loan Amount
      ─────────────────────
      Appraised Value of Collateral
```

**Thresholds:**

| LTV | Signal |
|-----|--------|
| ≤ 65% | Strong — significant equity cushion |
| 66%–75% | Standard — within most CRE/equipment loan policy |
| 76%–80% | Elevated — common max for owner-occupied RE |
| 81%–90% | High — SBA or specialty product territory |
| > 90% | Outside conventional policy |

```json
"ltv": {
  "loan_amount": 1200000,
  "collateral_value": 1800000,
  "ltv_pct": 66.7,
  "collateral_type": "Owner-occupied commercial real estate",
  "appraisal_date": "2023-11-15",
  "pass": true
}
```

---

### 3. Leverage Ratios

```
Debt-to-EBITDA = Total Debt
                 ──────────────────
                 Adjusted EBITDA

Debt-to-Equity = Total Liabilities
                 ──────────────────
                 Total Equity (Tangible Net Worth)

Senior Leverage = Senior Debt Only
                  ──────────────────
                  Adjusted EBITDA
```

**Thresholds:**

| Ratio | Conservative | Moderate | Aggressive |
|-------|-------------|----------|------------|
| Debt/EBITDA | < 3.0x | 3.0x–4.5x | 4.5x–6.0x |
| Debt/Equity | < 1.5x | 1.5x–2.5x | 2.5x–4.0x |

```json
"leverage": {
  "total_debt": 578600,
  "adjusted_ebitda": 213660,
  "debt_to_ebitda": 2.71,
  "total_equity": 362200,
  "debt_to_equity": 0.88,
  "signal": "conservative"
}
```

---

### 4. Liquidity Ratios

```
Current Ratio = Current Assets / Current Liabilities

Quick Ratio = (Cash + Accounts Receivable) / Current Liabilities

Days Sales Outstanding (DSO) = (AR Net / Annual Revenue) × 365

Days Payable Outstanding (DPO) = (AP / COGS) × 365

Working Capital = Current Assets − Current Liabilities
```

**Thresholds:**

| Ratio | Below Threshold | Acceptable | Strong |
|-------|----------------|------------|--------|
| Current Ratio | < 1.0x | 1.0x–1.5x | > 1.5x |
| Quick Ratio | < 0.8x | 0.8x–1.2x | > 1.2x |
| DSO | Industry-dependent — flag if > 60 days for services |
| DPO | Flag if > 60 days (potential AP strain) |

```json
"liquidity": {
  "current_assets": 402600,
  "current_liabilities": 173000,
  "current_ratio": 2.33,
  "cash": 144600,
  "ar_net": 174200,
  "quick_ratio": 1.73,
  "working_capital": 229600,
  "dso_days": 69.8,
  "dso_flag": true,
  "dso_note": "DSO slightly elevated for contracting — monitor AR aging"
}
```

---

### 5. Profitability Ratios

```
Gross Margin %    = Gross Profit / Net Revenue
Operating Margin% = Operating Income / Net Revenue
EBITDA Margin %   = Adjusted EBITDA / Net Revenue
Net Margin %      = Net Income / Net Revenue

Return on Assets (ROA) = Net Income / Total Assets
Return on Equity (ROE) = Net Income / Total Equity
```

```json
"profitability": {
  "gross_margin_pct": 49.1,
  "ebitda_margin_pct": 23.5,
  "net_margin_pct": 12.7,
  "roa": 17.0,
  "roe": 31.9,
  "industry_benchmark_gross_margin": "42–50%",
  "vs_benchmark": "in-line"
}
```

---

### 6. Trend Analysis (Multi-Year)

Compute year-over-year change for key metrics:

```
revenue_growth_yoy     = (current_yr - prior_yr) / prior_yr
ebitda_growth_yoy
net_income_growth_yoy
gross_margin_change_yoy  ← in percentage points
dscr_trend             ← improving | stable | declining
```

If 3 years available, compute CAGR:
```
revenue_cagr = (revenue_yr3 / revenue_yr1) ^ (1/2) − 1
```

Flag if:
- Revenue declining 2+ consecutive years
- EBITDA margin compressing more than 3 points year-over-year
- DSO trending up (collection slowdown)

---

### 7. Coverage & Capacity Ratios (Specialty)

#### For Real Estate / Rental Properties:
```
Net Operating Income (NOI) = Gross Rents − Operating Expenses (excl. debt service)
Cap Rate = NOI / Property Value
Debt Yield = NOI / Loan Amount     ← lender's return if they take the property back
```

#### For Construction / WIP Businesses:
```
Overbilling Ratio = Billings in Excess / Total Backlog
Underbilling Ratio = Costs in Excess / Total Backlog
WIP Net Position = Costs in Excess − Billings in Excess
```

#### For Partnerships (real estate):
```
Loan-to-Cost (LTC) = Total Debt / Total Project Cost
Interest Coverage = NOI / Annual Interest Expense
```

---

## Ratio Summary Scorecard

Generate a summary scorecard with pass/fail/flag per ratio:

```json
"scorecard": {
  "dscr_entity":       { "value": 1.42, "threshold": 1.25, "result": "PASS" },
  "dscr_global":       { "value": 1.28, "threshold": 1.25, "result": "PASS" },
  "ltv":               { "value": 0.667,"threshold": 0.75, "result": "PASS" },
  "debt_to_ebitda":    { "value": 2.71, "threshold": 4.5,  "result": "PASS" },
  "debt_to_equity":    { "value": 0.88, "threshold": 2.5,  "result": "PASS" },
  "current_ratio":     { "value": 2.33, "threshold": 1.25, "result": "PASS" },
  "quick_ratio":       { "value": 1.73, "threshold": 0.8,  "result": "PASS" },
  "dso_days":          { "value": 69.8, "threshold": 60,   "result": "FLAG" },
  "gross_margin_trend":{ "value": -0.5, "threshold": -3.0, "result": "PASS" },
  "overall":           "PASS — 1 flag (DSO) requiring monitoring"
}
```

---

## Sensitivity Analysis

For any DSCR between 1.10x and 1.35x, run sensitivity scenarios:

```
scenario_a: Revenue declines 10% — what is DSCR?
scenario_b: Interest rate increases 200bps (if variable rate debt) — what is DSCR?
scenario_c: Owner leaves / key-man scenario — what is sustainable EBITDA?
scenario_d: Proposed loan at max requested amount vs. proposed amount
```

```json
"sensitivity": {
  "base_dscr": 1.28,
  "revenue_down_10pct": { "adjusted_revenue": 819720, "dscr": 1.09, "result": "MARGINAL" },
  "rate_up_200bps":     { "additional_interest": 24000, "dscr": 1.18, "result": "ACCEPTABLE" },
  "key_man_departure":  { "adjusted_ebitda": 168000, "dscr": 0.98, "result": "FAIL — flag for key-man insurance requirement" }
}
```

---

## Normalization Notes

- **Inconsistent year-end dates**: If fiscal year ≠ calendar year, note and annualize partial periods
- **Non-recurring items already in EBITDA**: Don't double-count add-backs already captured in adjusted EBITDA
- **Pass-through entity taxes**: S-corp and partnership income is taxed at the individual level — do not deduct entity-level income taxes unless the entity makes tax distributions
- **Minority interest**: For partnerships, only spread the borrower's proportionate share of income and debt service obligation

---

## Output Format

```json
{
  "entity_name": "Holloway Mechanical LLC",
  "analysis_date": "2024-04-09",
  "years_analyzed": [2022, 2023],
  "ratios": {
    "dscr": { ... },
    "ltv": { ... },
    "leverage": { ... },
    "liquidity": { ... },
    "profitability": { ... },
    "trend": { ... }
  },
  "scorecard": { ... },
  "sensitivity": { ... },
  "ratio_flags": [ ... ],
  "analyst_notes": ""
}
```
