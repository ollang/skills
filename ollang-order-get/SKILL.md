---
name: ollang-order-get
description: Retrieve details of a specific Ollang order by its ID, including status, languages, documents, and QC results. Use when the user wants to check the status or details of an order.
---

# Ollang Get Order by ID

Fetch full details of a single order including its current status, associated documents, and QC evaluation results.

## Authentication

All requests require the `X-Api-Key` header from https://lab.ollang.com.

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
  "status": "pending | ongoing | completed | revision | waitingForCC | waitingForSubtitle",
  "type": "cc | subtitle | document | aiDubbing | studioDubbing",
  "paymentAmount": "number",
  "orderDocs": [
    {
      "id": "string",
      "type": "string",
      "url": "string"
    }
  ],
  "qcEvaluation": { }
}
```

### Order Statuses
| Status | Description |
|--------|-------------|
| `pending` | Awaiting processing |
| `ongoing` | Currently being processed |
| `completed` | Finished successfully |
| `revision` | Under revision |
| `waitingForCC` | Waiting for closed captions |
| `waitingForSubtitle` | Waiting for subtitle file |

## Example (curl)
```bash
curl -X GET https://api-integration.ollang.com/integration/orders/ORDER_ID \
  -H "X-Api-Key: YOUR_API_KEY"
```

## Behavior

1. Ask the user for their API key if not provided
2. Ask for the `orderId` if not provided
3. Fetch and display the order details
4. Highlight the `status` field prominently
5. If `orderDocs` are present, list available documents and their download URLs
6. If `qcEvaluation` is present, summarize the QC scores

## Error Codes
- `401` - Invalid or missing API key
- `403` - Access denied (order belongs to another account)
- `404` - Order not found
