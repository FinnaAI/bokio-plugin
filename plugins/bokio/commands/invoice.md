---
description: Create and manage invoices in Bokio
---

# Bokio Invoice Management

Create, list, and manage invoices in Bokio.

## Actions

Based on user request, perform one of:

### 1. Create Invoice

Gather required info:
- **Customer** - Name or ID (list customers if needed)
- **Line items** - Description, quantity, unit price, VAT rate
- **Due date** - Payment terms
- **Currency** - SEK default, or EUR/USD for international

```bash
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/invoices" \
  -H "Authorization: Bearer {TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "uuid",
    "dueDate": "2025-02-15",
    "currency": "SEK",
    "lineItems": [
      {
        "description": "Consulting services",
        "quantity": 10,
        "unitPrice": 1500,
        "vatRate": 25
      }
    ]
  }'
```

### 2. List Invoices

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/invoices?page=1&pageSize=25" \
  -H "Authorization: Bearer {TOKEN}"
```

Show as table: Invoice #, Customer, Amount, Status, Due Date

### 3. Publish Invoice

After creation, publish to make it official:

```bash
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/invoices/{INVOICE_ID}/publish" \
  -H "Authorization: Bearer {TOKEN}"
```

### 4. Create Credit Note

For refunds or corrections:

```bash
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/invoices/{INVOICE_ID}/credit" \
  -H "Authorization: Bearer {TOKEN}"
```

## Tips

- Always confirm details before publishing (can't undo)
- Use `/bokio:customers` to find customer IDs
- Swedish invoices need F-skatt and org.nr (Bokio adds these)
