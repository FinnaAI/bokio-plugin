---
name: bokio
description: Full Bokio accounting integration - upload receipts, create invoices, track unpaid, export SIE files, find missing receipts. Swedish accounting platform API.
---

# Bokio Accounting Skill

Complete integration with Bokio's Swedish accounting platform. Manage receipts, invoices, customers, and financial exports.

## When to use this skill

**Receipts & Documents:**
- User says "upload to bokio" or mentions receipts
- User asks about "missing receipts" or "entries without receipts"
- User has email attachments to send to Bokio

**Invoices & Billing:**
- User wants to create, list, or manage invoices
- User asks about unpaid or outstanding invoices
- User wants to track customer payments

**Customers:**
- User wants to add or list customers
- User needs a customer ID for invoicing

**Exports & Reports:**
- User asks for SIE export or accounting files
- User mentions accountant or Skatteverket
- User wants fiscal year data

## Configuration

Credentials can be stored in either location (checked in order):

1. **Plugin config** (recommended): `~/.claude/bokio.local.md`
2. **Env file** (legacy): `~/dev/finna/bokio-finna/.env`

Run `/bokio:setup` to configure credentials interactively.

### Config file format (`~/.claude/bokio.local.md`)

```yaml
---
bokio_company_id: your-company-uuid
bokio_api_token: your-api-token
---
```

### Env file format (legacy)

```
BOKIO_COMPANY_ID=your-company-uuid
BOKIO_API_TOKEN="your-api-token"
```

## API Reference

**Base URL:** `https://api.bokio.se/v1`
**Auth:** Bearer token in Authorization header

### Endpoints

**Uploads (Receipts):**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies/{id}/uploads` | POST | Upload a file |
| `/companies/{id}/uploads` | GET | List uploads (paginated) |
| `/companies/{id}/uploads/{uploadId}` | GET | Get upload details |

**Journal Entries:**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies/{id}/journal-entries` | GET | List entries (paginated) |
| `/companies/{id}/journal-entries/{entryId}` | GET | Get entry details |

**Invoices:**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies/{id}/invoices` | POST | Create invoice |
| `/companies/{id}/invoices` | GET | List invoices |
| `/companies/{id}/invoices/{invoiceId}` | GET | Get invoice |
| `/companies/{id}/invoices/{invoiceId}/publish` | POST | Publish invoice |
| `/companies/{id}/invoices/{invoiceId}/credit` | POST | Create credit note |

**Customers:**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies/{id}/customers` | POST | Create customer |
| `/companies/{id}/customers` | GET | List customers |
| `/companies/{id}/customers/{customerId}` | GET | Get customer |

**Exports:**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies/{id}/fiscal-years` | GET | List fiscal years |
| `/companies/{id}/sie-files/{fiscalYearId}` | GET | Download SIE file |

## Instructions

### Step 1: Load Credentials

**Try plugin config first:**

```bash
# Check if plugin config exists
if [ -f ~/.claude/bokio.local.md ]; then
  # Extract from YAML frontmatter
  COMPANY_ID=$(grep "bokio_company_id:" ~/.claude/bokio.local.md | cut -d: -f2 | tr -d ' ')
  TOKEN=$(grep "bokio_api_token:" ~/.claude/bokio.local.md | cut -d: -f2 | tr -d ' ')
fi
```

**Fallback to env file:**

```bash
# If no plugin config, try legacy env file
if [ -z "$COMPANY_ID" ] && [ -f ~/dev/finna/bokio-finna/.env ]; then
  COMPANY_ID=$(grep "^BOKIO_COMPANY_ID=" ~/dev/finna/bokio-finna/.env | cut -d= -f2)
  TOKEN=$(grep "^BOKIO_API_TOKEN=" ~/dev/finna/bokio-finna/.env | cut -d= -f2 | tr -d '"')
fi
```

**If no credentials found**, prompt user to run `/bokio:setup`.

### Step 2: Determine Action

Ask the user what they want to do:

1. **Upload file** - From local path or base64 data
2. **List uploads** - Show recent uploads with filtering
3. **Get upload** - Retrieve specific upload details
4. **Find missing receipts** - Cross-reference entries vs uploads

### Step 3a: Upload File

**Supported formats:** PDF, JPEG, PNG only

**From local file path:**

```bash
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/uploads" \
  -H "Authorization: Bearer {API_TOKEN}" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/file.pdf" \
  -F "description=Invoice from Supplier XYZ"
```

**From base64 data (for email attachments):**

First decode and save to temp file, then upload:

```bash
# Decode base64 to temp file
echo "{base64_data}" | base64 -d > /tmp/receipt.pdf

# Upload
curl -X POST "https://api.bokio.se/v1/companies/{COMPANY_ID}/uploads" \
  -H "Authorization: Bearer {API_TOKEN}" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/tmp/receipt.pdf" \
  -F "description=Email attachment - Invoice"

# Clean up
rm /tmp/receipt.pdf
```

**Optional parameters:**
- `description` - Defaults to filename if omitted
- `journalEntryId` - UUID to link upload to a journal entry

**Response (200 OK):**
```json
{
  "id": "a419cf69-db6f-4de9-992c-b1a60942a443",
  "description": "example.pdf",
  "contentType": "application/pdf",
  "journalEntryId": null
}
```

### Step 3b: List Uploads

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/uploads?page=1&pageSize=25" \
  -H "Authorization: Bearer {API_TOKEN}" \
  -H "Accept: application/json"
```

**Query parameters:**
- `page` - Page number (default: 1)
- `pageSize` - Items per page (default: 25, max: 100)
- `query` - Filter by description (e.g., `query=description~invoice`)

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "description": "receipt.pdf",
      "contentType": "application/pdf",
      "journalEntryId": null
    }
  ],
  "page": 1,
  "pageSize": 25,
  "totalPages": 3,
  "totalItems": 72
}
```

### Step 3c: Get Upload Details

```bash
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/uploads/{UPLOAD_ID}" \
  -H "Authorization: Bearer {API_TOKEN}" \
  -H "Accept: application/json"
```

### Step 3d: Find Entries Missing Receipts

**IMPORTANT:** This operation fetches large amounts of data. Use the Task tool with `subagent_type=Bash` to run the fetch and cross-reference in a subagent, then return only the summary to the main conversation.

The Bokio API doesn't have a direct endpoint for this. Cross-reference locally:

**1. Fetch all journal entries (paginate if >100):**

```bash
# Write to script file to handle token properly
cat > /tmp/bokio-fetch.sh << 'SCRIPT'
#!/bin/bash
COMPANY_ID="{COMPANY_ID}"
TOKEN="{API_TOKEN}"

for page in 1 2 3; do
  curl -s -X GET "https://api.bokio.se/v1/companies/${COMPANY_ID}/journal-entries?page=${page}&pageSize=100" \
    -H "Authorization: Bearer ${TOKEN}" \
    -H "Accept: application/json" | jq '.items' > /tmp/entries_page${page}.json
done
jq -s 'add' /tmp/entries_page*.json > /tmp/all_entries.json

for page in 1 2; do
  curl -s -X GET "https://api.bokio.se/v1/companies/${COMPANY_ID}/uploads?page=${page}&pageSize=100" \
    -H "Authorization: Bearer ${TOKEN}" \
    -H "Accept: application/json" | jq '.items' > /tmp/uploads_page${page}.json
done
jq -s 'add' /tmp/uploads_page*.json > /tmp/all_uploads.json
SCRIPT
chmod +x /tmp/bokio-fetch.sh
/tmp/bokio-fetch.sh
```

**2. Cross-reference to find entries without uploads:**

```bash
jq --slurpfile linked <(jq -r '[.[] | select(.journalEntryId != null) | .journalEntryId] | unique' /tmp/all_uploads.json) '
  ($linked[0] // []) as $linkedIds |
  [.[] | select(.id as $id | ($linkedIds | index($id)) == null)]
' /tmp/all_entries.json > /tmp/entries_without_receipts.json
```

**3. Filter to entries that actually need receipts:**

Entries that DON'T need receipts (exclude these):
- Salary payments (Lön, Salary)
- Interest (ränta, Ränta)
- Tax entries (Skatt, arbetsgivar)
- Payment confirmations (betalad av)
- Annulments (Annul, Regler, valutavinst)

Entries that DO need receipts:
- Software licenses (Mjukvara, licens)
- Supplier invoices (Leverantör, Inköp)
- Purchases with vendor names (VERCEL, CLOUDFLARE, etc.)

**4. Export results:**

```bash
mkdir -p ~/dev/finna/bokio-finna/exports
jq '[.[] | {
  id, date, journalEntryNumber, title,
  amount: (.items | map(select(.debit > 0) | .debit) | add)
}] | sort_by(.date) | reverse' /tmp/entries_without_receipts.json \
  > ~/dev/finna/bokio-finna/exports/missing-receipts.json
```

**Exports location:** `~/dev/finna/bokio-finna/exports/`
- `missing-receipts.json` - Full list with details
- `missing-receipts.csv` - Spreadsheet format
- `missing-receipts-by-vendor.json` - Grouped by vendor

## Gmail Integration Workflow

### Current: Manual Upload from Email

When syncing email attachments to Bokio manually:

1. **Extract attachment** from email (base64 encoded)
2. **Determine file type** from filename or MIME type
3. **Validate format** - Only PDF, JPEG, PNG supported
4. **Decode and upload** using the base64 workflow above
5. **Use email subject** as description for context

**Example description format:**
`Email: {subject} - {sender} - {date}`

### Future: Automated Gmail Sync

**Status:** Planned - credentials not yet configured

**Setup required:**
1. Configure Gmail OAuth credentials in `~/dev/finna/bokio-finna/.env`:
   - `GMAIL_CLIENT_ID`
   - `GMAIL_CLIENT_SECRET`
2. Implement OAuth flow for Gmail API access
3. Set up filters for invoice-related emails

**Target senders to monitor:**
Based on missing receipts analysis, prioritize emails from:
- Vercel (receipts@vercel.com)
- Google Workspace (payments-noreply@google.com)
- Cloudflare, Fly.io, Neon.tech
- HubSpot, Anthropic, Cursor
- Apple, Namecheap

**Matching strategy:**
1. Search Gmail for invoices from vendors in `missing-receipts-by-vendor.json`
2. Match by vendor name + approximate date + amount if visible
3. Upload with description linking to journal entry
4. Optionally link upload to journal entry via `journalEntryId` parameter

## Error Handling

| HTTP Code | Error | Action |
|-----------|-------|--------|
| 400 | Invalid file data | Check file format (PDF/JPEG/PNG only) |
| 401 | Unauthorized | Check API token is valid |
| 404 | Not found | Verify company ID and upload ID |
| 429 | Rate limited | Wait for Retry-After seconds |

**Error response format:**
```json
{
  "code": "validation-error",
  "message": "Validation failed",
  "bokioErrorId": "uuid",
  "errors": [
    { "field": "#/file", "message": "Unsupported file type" }
  ]
}
```

## Example Conversations

### Quick Upload
```
User: /bokio-upload
Assistant: What would you like to upload to Bokio?
User: ~/Downloads/faktura-2024-01.pdf
Assistant: [Uploads file, returns upload ID and confirmation]
```

### List Recent
```
User: /bokio-upload list
Assistant: [Shows paginated list of recent uploads with IDs and descriptions]
```

### Gmail Attachment
```
User: Upload this invoice attachment to Bokio
[Provides base64 data or file]
Assistant: [Decodes, validates format, uploads, confirms]
```

### Find Missing Receipts
```
User: /bokio-upload missing
User: What entries need receipts?
User: Find transactions without invoices
Assistant: [Fetches all entries and uploads, cross-references, shows categorized list]
```

## Security Notes

- Never log or display the full API token
- Clean up temp files after upload
- Validate file types before uploading
- Use HTTPS only (API enforces this)

## Rate Limits

The Bokio API has rate limiting. If you hit a 429 response:
- Check the `Retry-After` header
- Wait the specified seconds before retrying
- Default wait time is 60 seconds if header is missing
