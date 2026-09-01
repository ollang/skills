---
name: ollang-xlsx-export
description: Export Ollang order segment data (timecodes, transcriptions, translations) as XLSX spreadsheets — a single order or all orders in multiple folders. Use when the user wants a spreadsheet/Excel export of subtitles or translations.
---

# Ollang XLSX Export

Export segment data (timecodes, source transcription, translated text) as XLSX spreadsheets, for a single order or in bulk for whole folders.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal (get the key from https://lab.ollang.com). **Never ask the user to share the key in the conversation, and never print, echo, or log its value** — pass it only via shell expansion of `$OLLANG_API_KEY`.

## Spreadsheet Format

Each sheet contains these columns:

| Column | Description |
|--------|-------------|
| SR NO | Sequential row number |
| TCR | Timecode (start timestamp) |
| SPEAKER | Speaker name |
| DIALOGUES | Source transcription text |
| *Target Language* | Translated text (column header is the target language code) |

---

## Export a Single Order

**GET** `https://api-integration.ollang.com/integration/orders/{orderId}/export-xlsx`

The target language is resolved automatically from the order. Response is a binary XLSX download (`Content-Disposition: attachment; filename="{OrderName}_{targetLanguage}.xlsx"`).

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `orderId` | string | Yes | The order to export |

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/orders/ORDER_ID/export-xlsx \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  --output order_export.xlsx
```

---

## Bulk Export Folders

**POST** `https://api-integration.ollang.com/integration/folder/export-xlsx`

Returns one multi-sheet XLSX with one sheet per order/language combination for all orders in the given folders. Sheet names follow `{OrderName} - {targetLanguage}` (max 31 characters). Response is a binary XLSX download (`filename="bulk_export_{date}.xlsx"`).

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `folderIds` | string[] | Yes | Folder IDs containing the orders to export |
| `targetLanguages` | string[] | Yes | Target language codes to include |

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/folder/export-xlsx \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "folderIds": ["FOLDER_ID"],
    "targetLanguages": ["tr", "en"]
  }' \
  --output bulk_export.xlsx
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal — never ask them to share the key in the conversation
2. Determine scope: **one order** (GET by orderId) or **folders in bulk** (POST with folderIds + targetLanguages)
3. Always pass `--output <file>.xlsx` to curl — the response is binary, never print it to the terminal
4. Find folder IDs via `ollang-folder` and order IDs via `ollang-orders-list` if needed
5. Confirm the saved file path and offer to open or summarize the export

## Error Codes
- `400` - Invalid parameters (e.g., empty `folderIds`)
- `401` - Invalid or missing API key
- `500` - No segment data found for the given order/folders and target languages
