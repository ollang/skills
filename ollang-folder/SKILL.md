---
name: ollang-folder
description: List and browse Ollang folders, and batch-manage translator assignments for folder orders. Use when the user wants to see their folder structure, find a folder ID, organize projects into folders, or assign/unassign a translator across a folder's orders.
---

# Ollang Folder Management

Retrieve a paginated list of folders for organizing projects, and batch-assign or unassign translators across a folder's orders.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal (get the key from https://lab.ollang.com). **Never ask the user to share the key in the conversation, and never print, echo, or log its value** — pass it only via shell expansion of `$OLLANG_API_KEY`.

---

## List Folders

**GET** `https://api-integration.ollang.com/integration/folder`

## Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `take` | number | 10 | Items per page (1–50) |
| `orderBy` | string | `id` | Sort by: `id`, `name`, `createdAt` |
| `orderDirection` | string | `desc` | `asc` or `desc` |
| `search` | string | — | Search by folder name |

## Response (200)

```json
{
  "data": [
    {
      "id": "string",
      "name": "string",
      "hexColor": "#B6B6B6",
      "type": "default | api",
      "createdAt": "ISO8601",
      "projectCount": 0
    }
  ],
  "meta": {
    "page": 1,
    "take": 10,
    "itemCount": 50,
    "pageCount": 5,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

### Example (curl)
```bash
# List all folders
curl -X GET "https://api-integration.ollang.com/integration/folder?page=1&take=20" \
  -H "X-Api-Key: $OLLANG_API_KEY"

# Search for a folder
curl -X GET "https://api-integration.ollang.com/integration/folder?search=marketing&orderBy=name&orderDirection=asc" \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Get Folder Order Language Pairs

**GET** `https://api-integration.ollang.com/integration/folder/{folderId}/order-language-pairs`

Returns the distinct source/target language combinations from orders in a folder. Useful before assigning or unassigning translators.

### Query Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | `unassigned` | `unassigned` — pairs from orders available for assignment; `assigned` — pairs from orders available for unassignment |

### Response (200)
```json
{
  "pairs": [
    { "sourceLanguage": "en", "targetLanguage": "fr" },
    { "sourceLanguage": "en", "targetLanguage": "tr" }
  ]
}
```

### Example
```bash
curl -X GET "https://api-integration.ollang.com/integration/folder/FOLDER_ID/order-language-pairs?status=assigned" \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Assign Translator to Folder Orders

**POST** `https://api-integration.ollang.com/integration/folder/{folderId}/assign-translator-to-orders`

Batch-assigns a translator to all eligible (**unassigned, completed**) orders in the folder. When no language filters are provided, all eligible orders are assigned.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `translatorId` | string | Yes | The translator to assign |
| `deadline` | string | No | Assignment deadline (ISO 8601, e.g., `2026-07-01T00:00:00.000Z`) |
| `sourceLanguage` | string | No | Only assign orders with this source language |
| `targetLanguage` | string | No | Only assign orders with this target language |

### Response (200)
```json
{ "assignedCount": 12 }
```

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/folder/FOLDER_ID/assign-translator-to-orders \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "translatorId": "TRANSLATOR_ID", "sourceLanguage": "en", "targetLanguage": "fr" }'
```

---

## Unassign Translator from Folder Orders

**POST** `https://api-integration.ollang.com/integration/folder/{folderId}/unassign-translator-from-orders`

Batch-unassigns the translator from all eligible (**assigned, ongoing**) orders in the folder. When no language filters are provided, all eligible orders are unassigned.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sourceLanguage` | string | No | Only unassign orders with this source language |
| `targetLanguage` | string | No | Only unassign orders with this target language |

Send `{}` to unassign all eligible orders.

### Response (200)
```json
{ "unassignedCount": 8 }
```

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/folder/FOLDER_ID/unassign-translator-from-orders \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal — never ask them to share the key in the conversation
2. For **listing**: ask for optional search term or page size; display results in a table: ID, Name, Type, Color, Project Count, Created At
3. Folder IDs can be used when uploading files (`folderId` parameter in `ollang-upload`) and for bulk XLSX export (`ollang-xlsx-export`)
4. For **translator assignment**: fetch language pairs first (`status=unassigned` for assign, `status=assigned` for unassign) to offer filter choices; confirm before batch operations and report the affected count
5. Show pagination info and offer to fetch more if available

## Error Codes
- `400` - Invalid query parameters or body (e.g., `translatorId` missing)
- `401` - Invalid or missing API key
- `500` - Server error
