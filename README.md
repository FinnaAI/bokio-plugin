# Bokio Plugin for Claude Code

Full [Bokio](https://www.bokio.se) Swedish accounting integration for Claude Code. Manage receipts, invoices, customers, and financial exports.

## Installation

In Claude Code, run:

```
/plugin marketplace add finnaai/bokio-plugin
/plugin install bokio@finnaai-bokio
```

Or test locally:

```bash
claude --plugin-dir ~/dev/bokio-plugin
```

## Setup

Run the setup command to configure your Bokio credentials:

```
/bokio:setup
```

You'll need:
- **Company ID** - Your Bokio company UUID (found in Settings > Integrations)
- **API Token** - Generate one in Settings > Integrations > API Access

## Commands

| Command | Description |
|---------|-------------|
| `/bokio:setup` | Configure API credentials |
| `/bokio:upload` | Upload a receipt or invoice |
| `/bokio:missing` | Find journal entries missing receipts |
| `/bokio:invoice` | Create and manage invoices |
| `/bokio:customers` | List and manage customers |
| `/bokio:unpaid` | Track unpaid invoices |
| `/bokio:sie` | Export SIE accounting files |

## Features

### Receipts & Documents

Upload files directly to Bokio:
```
/bokio:upload ~/Downloads/invoice.pdf
```
Supports PDF, JPEG, and PNG files.

Find missing receipts:
```
/bokio:missing
```

### Invoicing

Create invoices for customers:
```
/bokio:invoice create
```

Track unpaid invoices:
```
/bokio:unpaid
```

### Customers

List all customers:
```
/bokio:customers
```

### Exports

Download SIE files for your accountant:
```
/bokio:sie 2024
```

### Gmail Integration (Planned)

Future feature: automatically sync invoice emails from vendors like Vercel, Cloudflare, Google Workspace, etc.

## API Reference

Uses the [Bokio API](https://docs.bokio.se):

**Uploads:** `POST/GET /companies/{id}/uploads`
**Invoices:** `POST/GET /companies/{id}/invoices`, `/publish`, `/credit`
**Customers:** `POST/GET /companies/{id}/customers`
**Exports:** `GET /companies/{id}/sie-files/{fiscalYearId}`
**Journal Entries:** `GET /companies/{id}/journal-entries`

## License

MIT
