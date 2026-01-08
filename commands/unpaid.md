---
description: Track unpaid invoices and outstanding payments
---

# Unpaid Invoice Tracker

Show outstanding invoices and payments due to help with cash flow management.

**IMPORTANT:** Use the Task tool with `subagent_type=Bash` to fetch invoice data in a subagent.

## Fetch and Analyze

### 1. Get All Invoices

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/invoices?page=1&pageSize=100" \
  -H "Authorization: Bearer {TOKEN}" > /tmp/bokio-invoices.json
```

### 2. Filter Unpaid

Use jq to find unpaid invoices:

```bash
jq '[.items[] | select(.status == "sent" or .status == "overdue")] |
  sort_by(.dueDate) |
  map({
    invoiceNumber,
    customer: .customerName,
    amount: .totalAmount,
    currency,
    dueDate,
    daysOverdue: (now - (.dueDate | fromdateiso8601) | . / 86400 | floor)
  })' /tmp/bokio-invoices.json
```

### 3. Summary Report

Show:
- **Total outstanding** - Sum of unpaid invoices
- **Overdue** - Past due date
- **Due soon** - Within 7 days
- **By customer** - Who owes the most

## Output Format

```
=== Unpaid Invoices ===
Total outstanding: 125,000 SEK

OVERDUE (action needed):
| Invoice | Customer    | Amount     | Days Overdue |
|---------|-------------|------------|--------------|
| F-123   | Acme Corp   | 45,000 SEK | 15 days      |
| F-118   | Beta Ltd    | 30,000 SEK | 8 days       |

DUE SOON (next 7 days):
| Invoice | Customer    | Amount     | Due Date   |
|---------|-------------|------------|------------|
| F-125   | Gamma AB    | 50,000 SEK | 2025-01-15 |

=== By Customer ===
Acme Corp: 45,000 SEK (1 invoice, 15 days overdue avg)
Gamma AB:  50,000 SEK (1 invoice, due in 5 days)
Beta Ltd:  30,000 SEK (1 invoice, 8 days overdue)
```

## Actions

After showing unpaid invoices, offer:
1. **Send reminder** - Draft email to customer
2. **Record payment** - If payment received
3. **Create credit note** - If invoice disputed
