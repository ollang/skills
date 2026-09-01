---
name: ollang-figma
description: Translate Figma design files with Ollang — import a Figma file and create translation orders, list orders for a file, and check Figma order status including review gates. Use when the user wants to translate or localize a Figma design.
---

# Ollang Figma Integration

Import a Figma design file and create AI translation orders in one step, then track them per file or per order.

## Prerequisites

**Figma OAuth connection required.** The user's Figma account must be connected to Ollang before these endpoints work: Ollang dashboard (https://lab.ollang.com) → Settings → Integrations → Figma. If a request returns `400` with "Figma account not connected", point the user there.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

---

## Create Figma Order

**POST** `https://api-integration.ollang.com/integration/orders/figma/create`

Handles the full pipeline: imports text layers from the Figma file, creates/reuses the project, and creates one AI translation order per target language. The file is always re-imported to capture the latest design state.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fileKey` | string | Yes | Figma file key — the segment of the URL between `/design/` and the file name (e.g., `UwNAtPYAu5JPIhFc03M71h`) |
| `fileUrl` | string | Yes | Full Figma file URL (must contain `figma.com`) |
| `sourceLanguage` | string | Yes | Source language code of the design content (e.g., `en`) |
| `targetLanguages` | string[] | Yes | Target language codes (at least one) |
| `folderId` | string | No | Folder to organize the project in (default folder if omitted) |

### Response (201)
```json
{
  "projectId": "string",
  "importId": "string",
  "orders": [
    { "orderId": "string", "targetLanguage": "fr", "status": "ongoing" }
  ]
}
```

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/orders/figma/create \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fileKey": "UwNAtPYAu5JPIhFc03M71h",
    "fileUrl": "https://www.figma.com/design/UwNAtPYAu5JPIhFc03M71h/My-Design",
    "sourceLanguage": "en",
    "targetLanguages": ["fr", "de", "tr"]
  }'
```

---

## Get Figma Orders (by file)

**GET** `https://api-integration.ollang.com/integration/orders/figma?fileKey={fileKey}`

Lists all translation orders for a Figma file, newest first, with review info when an order is at a review gate.

### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fileKey` | string | Yes | The Figma file key to look up orders for |

### Response (200)
Array of orders:
```json
[
  {
    "id": "string",
    "status": "ongoing | review | waitingForApprove | completed | delivered | pending | revision",
    "targetLanguage": "fr",
    "createdAt": "ISO8601",
    "orderName": "My Design → FR",
    "reviewInfo": {
      "teamTagName": "Medical Review",
      "teamTagColor": "#FF5733",
      "reviewType": "assignment | comment"
    }
  }
]
```
`reviewInfo` is `null` unless the order is at a review gate.

### Example
```bash
curl -X GET "https://api-integration.ollang.com/integration/orders/figma?fileKey=FILE_KEY" \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Get Figma Order Status

**GET** `https://api-integration.ollang.com/integration/orders/figma/{orderId}/status`

Status of a single Figma order, including `completedAt` (null until completed/delivered) and `reviewInfo` when at a review gate.

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/orders/figma/ORDER_ID/status \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Determine the action: **create orders**, **list orders for a file**, or **check one order's status**
3. For **create**: extract the `fileKey` from the Figma URL if the user pastes one; collect source and target languages
4. Handle `409` by explaining an active order for that language already exists in the project — wait for it or cancel it first
5. Handle `400` "Figma account not connected" by pointing to dashboard → Settings → Integrations → Figma
6. When an order is in `review`, surface the reviewing team from `reviewInfo`

## Error Codes
- `400` - Figma account not connected, or missing/invalid parameters
- `401` - Invalid or missing API key
- `404` - Order not found
- `409` - An active order for a target language already exists in this project
- `422` - Figma file import failed (Figma API error or access issue)
