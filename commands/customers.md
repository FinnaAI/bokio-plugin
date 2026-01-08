---
description: List and manage customers in Bokio
---

# Bokio Customer Management

List, create, and manage customers for invoicing.

## Actions

### 1. List Customers

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/customers?page=1&pageSize=50" \
  -H "Authorization: Bearer {TOKEN}"
```

Show as table: Name, Email, Country, Customer ID

### 2. Create Customer

Gather info:
- **Name** (required) - Company or person name
- **Email** - For sending invoices
- **Country** - SE, GB, US, etc.
- **VAT number** - For EU B2B (reverse charge)
- **Address** - Street, city, postal code

```bash
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/customers" \
  -H "Authorization: Bearer {TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "email": "billing@acme.com",
    "country": "SE",
    "vatNumber": "SE123456789001",
    "address": {
      "street": "Storgatan 1",
      "city": "Stockholm",
      "postalCode": "111 23"
    }
  }'
```

### 3. Get Customer Details

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/customers/{CUSTOMER_ID}" \
  -H "Authorization: Bearer {TOKEN}"
```

### 4. Search Customer by Name

Filter the list to find a specific customer:

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/customers?query=name~Acme" \
  -H "Authorization: Bearer {TOKEN}"
```

## Tips

- Customer IDs are UUIDs - save them for quick invoice creation
- For EU customers, VAT number enables reverse charge (0% VAT)
- Email is required for Bokio to send invoice PDFs automatically
