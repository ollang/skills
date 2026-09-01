---
name: ollang-memory
description: Manage Ollang translation memories — create, list, rename, delete memories and import translation units in bulk. Use when the user wants to set up a translation memory, add source/target text pairs, or apply stored translations to orders.
---

# Ollang Translation Memory

Create and manage translation memories (TMs). A memory stores translation units (source/target text per language pair) that are matched per target language at translation time. Pass memory IDs as `selectedMemories` when creating an order (see `ollang-order-create`).

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

> **Plan requirement:** Translation Memory is a plan-gated feature. If the account's plan doesn't include it, create requests return `403` with a message asking to upgrade.

---

## List Memories

**GET** `https://api-integration.ollang.com/integration/memories`

### Response (200)
Array of memory objects:
```json
[
  {
    "id": "507f1f77bcf86cd799439011",
    "title": "Product documentation EN→FR",
    "memoryItemsCount": 12480,
    "createdAt": "ISO8601",
    "updatedAt": "ISO8601"
  }
]
```

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/memories \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Create Memory

**POST** `https://api-integration.ollang.com/integration/memories`

Creates an empty memory. Fill it with the import endpoint below.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Human-readable name of the memory |

### Response (201)
Memory object with `memoryItemsCount: 0`.

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/memories \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "title": "Product documentation EN→FR" }'
```

---

## Get Memory by ID

**GET** `https://api-integration.ollang.com/integration/memories/{memoryId}`

Returns the memory object including its stored translation-unit count.

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/memories/MEMORY_ID \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Update Memory (rename)

**PATCH** `https://api-integration.ollang.com/integration/memories/{memoryId}`

Renames the memory — stored translation units are not affected.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | New name of the memory |

### Example
```bash
curl -X PATCH https://api-integration.ollang.com/integration/memories/MEMORY_ID \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "title": "Product documentation EN→FR (v2)" }'
```

---

## Delete Memory

**DELETE** `https://api-integration.ollang.com/integration/memories/{memoryId}`

Permanently deletes the memory and its translation units. Cannot be undone. Orders referencing it via `selectedMemories` will no longer receive matches from it.

### Response
- **204 No Content** — Memory deleted

### Example
```bash
curl -X DELETE https://api-integration.ollang.com/integration/memories/MEMORY_ID \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Import Memory Items

**POST** `https://api-integration.ollang.com/integration/memories/{memoryId}/items/import`

Queues translation units for **asynchronous** import (including vectorization for semantic matching). Returns a job to poll — not a synchronous result.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `items` | array | Yes | 1–1000 translation units per request. Send multiple requests for larger corpora. |

Each item:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sourceLanguage` | string | Yes | Language code of the source text (e.g., `en`) |
| `targetLanguage` | string | Yes | Language code of the target text (e.g., `fr`) |
| `sourceText` | string | Yes | Source-language text |
| `targetText` | string | Yes | Target-language text |

A memory can hold several language pairs at once — matches are resolved per target language at translation time.

### Response (201)
```json
{ "jobId": "66b8d6f1e1b9b1d8c6c0d8e1", "status": "pending" }
```

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/memories/MEMORY_ID/items/import \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [
      { "sourceLanguage": "en", "targetLanguage": "fr", "sourceText": "Getting started", "targetText": "Premiers pas" },
      { "sourceLanguage": "en", "targetLanguage": "fr", "sourceText": "Release notes", "targetText": "Notes de version" }
    ]
  }'
```

---

## Get Memory Import Job

**GET** `https://api-integration.ollang.com/integration/memories/import-jobs/{jobId}`

Track import progress. Statuses: `pending`, `processing`, `completed`, `failed`.

### Response (200)
```json
{
  "jobId": "string",
  "status": "pending | processing | completed | failed",
  "progress": 0,
  "itemsCount": null,
  "error": null,
  "completedAt": null,
  "vectorizationStatus": "pending | processing | completed | failed",
  "vectorizationProgress": 0,
  "vectorizationError": null,
  "vectorizationCompletedAt": null
}
```

- Exact-lookup matching is available once `status` is `completed`.
- Semantic matching is available once `vectorizationStatus` is also `completed`.

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/memories/import-jobs/JOB_ID \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Determine the action: **list**, **create**, **get**, **rename**, **delete**, **import items**, or **check import job**
3. For **create**: ask for a `title`; suggest importing items right after
4. For **import**: batch items in chunks of at most 1000; return the `jobId` and poll the job endpoint until `status` is `completed` (mention vectorization for semantic matching)
5. For **delete**: confirm before deleting (destructive, cannot be undone)
6. Suggest next steps: pass memory IDs as `selectedMemories` in `ollang-order-create` (max 20 per order; only document, subtitle/cc, and Adobe AE orders consume memories)

## Error Codes
- `400` - Invalid parameters (e.g., more than 1000 items per import request)
- `401` - Invalid or missing API key
- `403` - Translation Memory feature not available in the current plan
- `404` - Memory or job not found
