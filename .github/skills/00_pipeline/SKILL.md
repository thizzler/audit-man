# SKILL: Commercial Lending Analysis Pipeline (Orchestrator)

## Trigger
Activate this skill when the user:
- Says "run the pipeline" or "analyze this customer"
- Provides a folder name or customer name with financial documents
- Says "process these docs" or "spread these financials"
- Uploads or points to a set of financial documents for a single borrower

---

## Overview
This skill orchestrates the full commercial lending analysis pipeline.
It calls four downstream skills in order, passing output from each step
as input to the next. The user provides a customer folder — this skill
handles the rest.

Pipeline order:
```
[Customer Docs]
      ↓
  STEP 1: 01_pdf_extraction    → 01_extracted.json
      ↓
  STEP 2: 02_spreading         → 02_spread.json
      ↓
  STEP 3: 03_ratio_calculation → 03_ratios.json
      ↓
  STEP 4: 04_credit_memo       → 04_credit_memo.md
```

---

## Before You Start

### Step 1 — Confirm the customer folder
Ask the user to confirm:
> "Which customer folder should I run the pipeline on?"

Accept any of these as valid input:
- A folder name: `holloway_mechanical`
- A customer name: `Holloway Mechanical LLC`
- A direct file upload (treat all uploaded files as one customer)

### Step 2 — Inventory the documents
List every PDF found in the customer folder and identify its type:

| File | Detected Type | Status |
|------|--------------|--------|
| 1040_personal_return.pdf | 1040 | ✓ Ready |
| profit_loss_statement.pdf | PL | ✓ Ready |
| balance_sheet.pdf | BS | ✓ Ready |

Supported doc types: `1040`, `1120S`, `1065`, `PL`, `BS`

If a file cannot be identified, flag it:
> "⚠️ Could not identify doc type for `unknown_file.pdf` — skipping. Please confirm what this document is."

### Step 3 — Confirm before running
Show the inventory to the user and ask:
> "Found 3 documents for Holloway Mechanical LLC. Ready to run the full pipeline. Proceed?"

Wait for confirmation before continuing.

---

## Step 1 — PDF Extraction
**Skill: `01_pdf_extraction/SKILL.md`**

Run extraction on every document in the inventory.

Instructions:
- Process each document separately using the extraction rules for its doc type
- Output a single JSON object per document
- Merge all documents into one combined extraction object keyed by doc type:

```json
{
  "customer": "Holloway Mechanical LLC",
  "extraction_timestamp": "2024-04-09T14:32:00",
  "documents_processed": ["1040", "PL", "BS"],
  "data": {
    "1040": { ... },
    "PL":   { ... },
    "BS":   { ... }
  },
  "extraction_flags": [ ... ]
}
```

- Collect all `extraction_flags` from every document into a single top-level array
- If any document fails to extract cleanly, note it in `extraction_flags` and continue — do not stop the pipeline

**Output file:** `01_extracted.json`

**Progress checkpoint:**
After extraction, show the user a brief summary:
```
✓ Extraction complete
  Documents: 1040, PL, BS
  Flags: personal_expenses_in_business, drawn_loc, wip_present
  Proceeding to spreading...
```

---

## Step 2 — Spreading
**Skill: `02_spreading/SKILL.md`**

Input: `01_extracted.json`

Instructions:
- Determine entity type from the documents present:
  - 1040 present → Individual / Sole Proprietor or pass-through guarantor
  - 1120S present → S-Corporation
  - 1065 present → Partnership
  - PL + BS without tax return → use as supplemental; note tax return missing
- Apply all recasting rules from the spreading skill for the detected entity type
- Reconcile income figures across documents if both a tax return and P&L are present
- Default to the more conservative figure; document the reconciliation
- Build the full spread schema including:
  - Borrower profile
  - Business cash flow (all available years)
  - Add-backs (itemized)
  - Adjusted EBITDA
  - Balance sheet summary
  - Global cash flow (if 1040 present)

**Output file:** `02_spread.json`

**Progress checkpoint:**
```
✓ Spreading complete
  Entity type: LLC / Sole Proprietor
  Adjusted EBITDA (2023): $213,660
  Add-backs: 4 items totaling $83,760
  Spread flags: heavy_addback_reliance, unverified_addbacks
  Proceeding to ratio calculation...
```

---

## Step 3 — Ratio Calculation
**Skill: `03_ratio_calculation/SKILL.md`**

Input: `02_spread.json`

Instructions:
- Before calculating ratios, ask the user for the proposed loan details if not already provided:
  > "To calculate DSCR and LTV I need a few details:
  > 1. Proposed loan amount?
  > 2. Proposed loan term and rate? (to estimate annual debt service)
  > 3. Collateral type and estimated value?"
- If the user already provided these at the start, use those values
- Calculate all ratios defined in the ratio skill:
  - DSCR (entity, global if 1040 present, consolidated)
  - LTV
  - Leverage (debt/EBITDA, debt/equity)
  - Liquidity (current ratio, quick ratio, DSO, working capital)
  - Profitability (gross margin, EBITDA margin, net margin, ROA, ROE)
  - Trend analysis if 2+ years available
- Generate the scorecard with PASS / FAIL / FLAG per ratio
- Run sensitivity analysis automatically if any DSCR is between 1.10x and 1.35x

**Output file:** `03_ratios.json`

**Progress checkpoint:**
```
✓ Ratio calculation complete
  DSCR (entity):  1.42x  ✓ PASS
  DSCR (global):  1.28x  ✓ PASS
  LTV:            66.7%  ✓ PASS
  Current Ratio:  2.33x  ✓ PASS
  DSO:            69.8d  ⚠ FLAG
  Overall:        PASS — 1 flag
  Proceeding to credit memo...
```

---

## Step 4 — Credit Memo
**Skill: `04_credit_memo/SKILL.md`**

Input: `02_spread.json` + `03_ratios.json`

Instructions:
- Draft the full 10-section credit memo
- Pull borrower details, financials, ratios, and flags from prior step outputs
- Do not fabricate any data — if a field is missing, write "Not provided — pending"
- Always itemize add-backs by name and amount
- Always show DSCR calculation explicitly
- Always state LTV
- Include the risk table with at least one row per spread flag carried forward
- End with a clear recommendation block

**Output file:** `04_credit_memo.md`

---

## Final Output Summary

After all four steps complete, present the user with:

```
═══════════════════════════════════════════════
  PIPELINE COMPLETE — {Customer Name}
  Run date: {date}
═══════════════════════════════════════════════

  Documents processed:  {list}
  Steps completed:      4 / 4

  KEY RESULTS
  ───────────────────────────────────────────
  Adjusted EBITDA:      ${amount}
  DSCR (entity):        {x}x  {PASS/FAIL}
  DSCR (global):        {x}x  {PASS/FAIL}
  LTV:                  {%}   {PASS/FAIL}
  Overall scorecard:    {PASS / FAIL / CONDITIONAL}

  FLAGS REQUIRING ATTENTION
  ───────────────────────────────────────────
  {list each flag from any step}

  OUTPUT FILES
  ───────────────────────────────────────────
  01_extracted.json
  02_spread.json
  03_ratios.json
  04_credit_memo.md

═══════════════════════════════════════════════
```

---

## Partial Pipeline Runs

If the user only wants to run specific steps, accept these commands:

| User says | Run |
|-----------|-----|
| "just extract" / "extraction only" | Step 1 only |
| "extract and spread" | Steps 1–2 |
| "skip to ratios" | Steps 3–4 (requires existing 01 and 02 output) |
| "just the memo" | Step 4 only (requires existing 02 and 03 output) |
| "re-run ratios with new loan amount" | Step 3 only, prompt for new loan details |
| "re-run memo" | Step 4 only using existing spread and ratio files |

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Doc type cannot be identified | Flag and skip — continue with remaining docs |
| Required field missing from extraction | Note as "not provided" — do not halt pipeline |
| No tax return present (PL + BS only) | Proceed with note: "Tax return not provided — spreading based on financial statements only. Results should be treated as preliminary pending tax return review." |
| Conflicting figures across documents | Apply conservative figure — document reconciliation in spread output |
| User interrupts mid-pipeline | Save completed step outputs — allow resume from last completed step |
| Loan details not provided by user | Pause at Step 3 and ask — do not estimate or assume loan terms |

---

## Output Folder Structure

```
output/
  {customer_name}/
    01_extracted.json
    02_spread.json
    03_ratios.json
    04_credit_memo.md
    pipeline.log          ← timestamps, flags, and step summaries
```

All output files are written after each step completes — not held until the end.
This allows the user to inspect intermediate results if needed.

---

## Change Log
- 2024-04-09: Initial version