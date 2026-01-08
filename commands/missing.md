---
description: Find journal entries that are missing receipts
---

# Find Missing Receipts

Cross-reference Bokio journal entries against uploaded receipts to find entries that need documentation.

**IMPORTANT:** Use the Task tool with `subagent_type=Bash` to run the data fetching and cross-reference operations. This avoids loading large JSON responses into the main conversation context.

Use the bokio-upload skill instructions (Step 3d) to:
1. Load credentials from `~/dev/finna/bokio-finna/.env`
2. Fetch all journal entries (paginated)
3. Fetch all uploads
4. Cross-reference to find entries without receipts
5. Filter out entries that don't need receipts (salary, interest, tax, etc.)
6. Categorize and display entries that DO need receipts
7. Export results to `~/dev/finna/bokio-finna/exports/`

Show a summary table with:
- Total entries vs entries with/without receipts
- Breakdown by category (software licenses, purchases, etc.)
- Top vendors missing receipts with amounts
