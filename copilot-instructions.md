## Triggering
When the user provides a financial document (tax return, P&L, or balance sheet),
automatically run the following pipeline in order:
1. Identify doc type and run 01_pdf_extraction
2. Pass output to 02_spreading
3. Pass output to 03_ratio_calculation
4. Generate a credit memo using 04_credit_memo

Do not ask the user which step to run - execute the full pipeline unless
the user explicitly requests only a specific step.

## Environment Rules
- Never install packages with pip unless explicitly asked
- All dependencies are declared in requirements.txt — reference it before importing anything
- If a required package is missing, STOP and tell the user rather than installing it
- Use the existing virtual environment at .venv/ — do not create a new one