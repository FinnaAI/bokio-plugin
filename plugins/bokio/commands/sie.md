---
description: Export SIE accounting files from Bokio
---

# Bokio SIE Export

Download SIE files for your accountant or Skatteverket.

## What is SIE?

SIE (Standard Import Export) is the Swedish standard format for accounting data. Used for:
- Sending to your accountant
- Tax filings (Skatteverket)
- Migrating between accounting systems
- Annual reports (årsredovisning)

## Export SIE File

```bash
# Get available fiscal years first
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/fiscal-years" \
  -H "Authorization: Bearer {TOKEN}"

# Download SIE file for a fiscal year
curl -X GET "https://api.bokio.se/v1/companies/{COMPANY_ID}/sie-files/{FISCAL_YEAR_ID}" \
  -H "Authorization: Bearer {TOKEN}" \
  -o ~/Downloads/bokio-sie-{YEAR}.se
```

## SIE File Types

Bokio typically exports SIE4 format which includes:
- Chart of accounts (kontoplan)
- All transactions (verifikationer)
- Opening/closing balances
- Company information

## Common Use Cases

### For Accountant
```
/bokio:sie 2024
```
Downloads the 2024 fiscal year SIE file to share with your accountant.

### For Tax Filing
```
/bokio:sie current
```
Downloads current year for tax preparation.

### For Backup
```
/bokio:sie all
```
Downloads SIE files for all fiscal years as backup.

## Tips

- Run this quarterly or before meeting your accountant
- SIE files are text-based, can be opened in any text editor
- Keep copies in a safe place (cloud backup recommended)
