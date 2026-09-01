---
name: ollang-content
description: Export and import Ollang content translations (i18n key-value pairs) as JSON. Use when the user wants to pull translated strings into their app, sync locale files, or bulk-load translations from an external system or TMS export.
---

# Ollang Content Export & Import

Move translation units in and out of the account's content database. Useful for website/app localization workflows: export translated strings as JSON locale maps, or bulk-import existing translations.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

---

## Export Content

**GET** `https://api-integration.ollang.com/integration/content/export`

Exports content translations, filtered by target language(s), tag(s), and order ID(s).

### Query Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `targetLanguage` | string | Single target language code (e.g., `tr`). Do **not** combine with `targetLanguages` — pick one. |
| `targetLanguages` | string[] | One or more language codes. Both `?targetLanguages[]=tr&targetLanguages[]=en` and `?targetLanguages=tr&targetLanguages=en` are accepted. |
| `tag` | string | Filter by a single tag |
| `tags` | string[] | Content terms with at least one of these tags |
| `orderIds` | string[] | Only content terms associated with these orders |

### Response (200)

**Single language** — flat map of source text → translation:
```json
{
  "Good morning": "Günaydın",
  "Sign in": "Giriş Yap"
}
```

**Multiple languages** — grouped by language code:
```json
{
  "tr": { "Good morning": "Günaydın" },
  "es": { "Good morning": "Buenos días" }
}
```

Returns `{}` when no content matches.

### Example (curl)

> **Important:** Use curl's `-g` (`--globoff`) flag when using the bracket syntax (`targetLanguages[]=`).

```bash
# Single language
curl "https://api-integration.ollang.com/integration/content/export?targetLanguage=tr" \
  -H "X-Api-Key: $OLLANG_API_KEY"

# Multiple languages with tag filter
curl -g "https://api-integration.ollang.com/integration/content/export?targetLanguages[]=tr&targetLanguages[]=en&tags[]=ui" \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Import Content

**POST** `https://api-integration.ollang.com/integration/content/import`

Imports translation units (source/target text pairs) into the content database. Imported terms become immediately available for export, tagging, and editing.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `targetLanguage` | string | Yes | Target language code for the whole batch (e.g., `de`) |
| `translations` | array | Yes | Translation units to import |

Each translation unit:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sourceText` | string | Yes | Original source text (used as the key for matching/deduplication across languages) |
| `targetText` | string | Yes | Translated text in the target language |
| `elementId` | string | No | Origin reference (DOM element ID, JSON key path, custom ref) |
| `type` | string | No | Content type: `text` (default), `ui`, `subtitle`, `navigation` |

Import is **idempotent** per `sourceText` + `targetLanguage` — re-importing updates the target text; new source texts create new content terms.

### Response (201)
```json
{ "success": true, "imported": 3 }
```

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/content/import \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "targetLanguage": "de",
    "translations": [
      { "sourceText": "Good morning", "targetText": "Guten Morgen" },
      { "sourceText": "Sign in", "targetText": "Anmelden", "type": "ui" }
    ]
  }'
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Determine the action: **export** or **import**
3. For **export**: ask which language(s), and optional tag/order filters; offer to save the JSON to a locale file (e.g., `locales/tr.json`)
4. For **import**: collect `targetLanguage` and the source/target pairs (offer to read them from an existing locale/JSON file); one batch per target language
5. Report counts (`imported`) and suggest exporting to verify

## Error Codes
- `400` - Invalid parameters (e.g., empty `translations`, non-string language values)
- `401` - Invalid or missing API key
