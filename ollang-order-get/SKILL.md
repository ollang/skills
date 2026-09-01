---
name: ollang-order-get
description: Retrieve details of a specific Ollang order by its ID, including status, languages, documents, and QC results. Use when the user wants to check the status or details of an order.
---

# Ollang Get Order by ID

Fetch full details of a single order including its current status, associated documents, and QC evaluation results.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal (get the key from https://lab.ollang.com). **Never ask the user to share the key in the conversation, and never print, echo, or log its value** — pass it only via shell expansion of `$OLLANG_API_KEY`.

## Endpoint

**GET** `https://api-integration.ollang.com/integration/orders/{orderId}`

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `orderId` | string | Yes | The unique order identifier |

## Response (200)

```json
{
  "id": "string",
  "name": "string",
  "createdAt": "ISO8601 timestamp",
  "sourceLanguage": "string",
  "targetLanguage": "string",
  "status": "pending | ongoing | completed | revision | delayed | waitingForCC | waitingForSubtitle | delivered | qualityCheck | readyToSent",
  "type": "cc | subtitle | document | aiDubbing | studioDubbing | proofreading | other | revision",
  "rate": 0.15,
  "projectId": "string",
  "level": 0,
  "folderId": "string",
  "vttUrl": "string",
  "orderDocs": [
    {
      "id": "string",
      "name": "string",
      "url": "string",
      "type": "string",
      "size": 0,
      "duration": 0,
      "wordCount": 0,
      "sourceLanguage": "string",
      "createdAt": "ISO8601",
      "updatedAt": "ISO8601"
    }
  ],
  "finance": {
    "paymentAmount": 0
  },
  "latestEvaluation": {
    "id": "string",
    "orderId": "string",
    "createdAt": "ISO8601",
    "textSummary": "string",
    "scores": [
      { "name": "Accuracy", "description": "string", "value": 0 }
    ],
    "segmentEvals": [
      { "id": "string", "explain": "string", "suggestedNewValue": "string" }
    ],
    "isLoading": false
  },
  "deliveryEvents": [
    {
      "type": "ai_delivered | translator_delivered | lsp_approved | order_delivered",
      "occurredAt": "ISO8601",
      "actorType": "ai | translator | lsp | system",
      "actorId": "string or null"
    }
  ]
}
```

### Order Statuses
| Status | Description |
|--------|-------------|
| `pending` | Awaiting processing |
| `ongoing` | Currently being processed |
| `completed` | Finished successfully |
| `revision` | Under revision |
| `delayed` | Processing is delayed |
| `waitingForCC` | Waiting for closed captions |
| `waitingForSubtitle` | Waiting for subtitle file |
| `delivered` | Content has been delivered |
| `qualityCheck` | Undergoing quality check |
| `readyToSent` | Ready to be sent/delivered |

### Delivery Events

`deliveryEvents` records the order's delivery milestones, oldest first — a status is only a snapshot, but this array shows how the order got there:

| Event Type | Meaning |
|------------|---------|
| `ai_delivered` | AI pipeline finished; AI results became available |
| `translator_delivered` | A translator delivered their work |
| `lsp_approved` | An LSP admin/PM approved a member's delivery |
| `order_delivered` | The order reached a delivered/completed state |

Empty array for orders that haven't reached a milestone yet, or orders created before delivery-event tracking existed (the log is not backfilled). The same event type can appear more than once (e.g., a re-delivery or rerun). `actorId` is null for `ai` and `system` events.

## Example (curl)
```bash
curl -X GET https://api-integration.ollang.com/integration/orders/ORDER_ID \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal — never ask them to share the key in the conversation
2. Ask for the `orderId` if not provided
3. Fetch and display the order details
4. Highlight the `status` field prominently
5. If `orderDocs` are present, list available documents and their download URLs
6. If `latestEvaluation` is present, summarize the QC scores
7. If the user asks about delivery history or timing, summarize `deliveryEvents` chronologically

## Error Codes
- `401` - Invalid or missing API key
- `403` - Access denied (order belongs to another account)
- `404` - Order not found
