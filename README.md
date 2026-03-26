# Mariner Paperless → Receipts Space Migration

A Python script to migrate receipts from [Mariner Paperless] (also known as ReceiptWallet) to [Receipts Space](https://receipts-space.com).

## Background

Mariner Software went out of business, leaving Paperless users with a large library of receipts in a proprietary format with no migration path. This script reads the Paperless SQLite database and writes receipts directly to Receipts Space's open `.dat` transaction format, preserving vendors, categories, amounts, dates, notes, and tags.

Successfully migrated ~23,000 receipts across three libraries (EUR, USD, AUD).

## Features

- Migrates receipts, vendors, categories, amounts, dates, notes and payment dates
- Handles EU and US number formats (e.g. `1.234,56` and `1,234.56`)
- Normalises filenames (double extensions, multiple spaces, etc.)
- Maps Paperless tags to Receipts Space tags
- Handles `_deleted` records gracefully
- Supports multiple currency libraries (EUR, USD, AUD) via `--mode` flag

## Limitations

- **Account and Posted** are custom fields in Paperless that have no equivalent in Receipts Space. The script omits these fields. If you relied on them, you will need to handle them manually or extend the script.
- Receipts Space does not currently support custom fields.

## Requirements

- Python 3.9+
- [Receipts Space](https://receipts-space.com) for Mac (v2+)
- Mariner Paperless installed (to locate the library)

## Usage

```bash
python3 paperless_to_receipts.py --mode aud
```

Supported modes: `eur`, `usd`, `aud`

## Setup

Before running, edit the following variables at the top of the script to match your setup:

- `PAPERLESS_LIBRARY` — path to your Paperless library
- `RS_LIBRARY` — path to your Receipts Space library
- `RS_CLIENT_ID` — your Receipts Space client ID (found in the `transactions/` folder)
- `RS_WORKSPACE_ID` — your workspace ID (found in `info.json`)

## How It Works

Receipts Space uses an open, documented [CRDT-based file format](https://receipts-space.com/en/docs). Each change is stored as a `.dat` transaction file with a JSON header and JSON content line. This script writes new transaction files directly into the Receipts Space library, chaining them correctly using SHA256 hashes.

See [Receipts Space Technical Documentation](https://receipts-space.com/en/docs) for details on the file format.

## Caveats

- For large migrations, close Receipts Space before running the script. For small batches it can stay open.
- Receipts Space should pick up the changes automatically after the script completes. If entries don't appear updated, quit RS and delete the cache:
  `~/Library/Application Support/de.holtwick.mac.homebrew.Receipts2/data/<workspaceId>.*`
  Then reopen Receipts Space — it will rebuild from the transaction files.
- Duplicate detection in Receipts Space should be temporarily disabled during migration
- Test with a small batch first before migrating your full library

## License

GNU General Public License v3.0 — see [LICENSE](LICENSE) for details.
