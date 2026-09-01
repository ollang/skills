---
name: ollang-credits
description: Check the Ollang AI credit wallet balance and list credit consumption line items with filters. Use when the user asks about their credit balance, spend, billing usage, or consumption breakdown per provider, order, or requestor.
---

# Ollang Credits & Consumption

Retrieve the account credit wallet and the per-provider-run consumption breakdown.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal (get the key from https://lab.ollang.com). **Never ask the user to share the key in the conversation, and never print, echo, or log its value** — pass it only via shell expansion of `$OLLANG_API_KEY`.

> **Role requirement:** Both endpoints are restricted to **account owners and billing managers**. The API key must belong to a user with the `client_owner` or `client_accessBillings` role, otherwise the request is rejected with `403`.

---

## Get Credit Wallet

**GET** `https://api-integration.ollang.com/integration/credits`

Returns total, remaining, and used AI credits with USD equivalents at the account rate of **1000 credits = $1**.

### Response (200)
```json
{
  "totalCredits": 30000000,
  "remainingCredits": 26649461.48,
  "usedCredits": 3350538.52,
  "currency": "USD",
  "creditsPerUsd": 1000,
  "totalUsd": 30000,
  "remainingUsd": 26649.46,
  "usedUsd": 3350.54,
  "lastSyncDate": "ISO8601 or null"
}
```

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/credits \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## List Consumption

**GET** `https://api-integration.ollang.com/integration/consumption`

Paginated list of AI credit consumption line items — one row per provider run — equivalent to the dashboard **Consumption Breakdown** export.

### Query Parameters

#### Pagination & Sorting
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pageOptions[page]` | number | 1 | Page number |
| `pageOptions[take]` | number | 10 | Items per page (1–50) |
| `pageOptions[orderBy]` | string | `id` | Sort field: `occurredAt`, `creditsUsed`, `provider` |
| `pageOptions[orderDirection]` | string | `desc` | `asc` or `desc` |
| `pageOptions[search]` | string | — | Search across relevant fields |

#### Filters
| Parameter | Type | Description |
|-----------|------|-------------|
| `filter[search]` | string | Search by order name (case-insensitive contains) |
| `filter[from]` | ISO 8601 | Include consumption from this date onward (inclusive) |
| `filter[to]` | ISO 8601 | Include consumption up to this date (inclusive) |
| `filter[provider]` | string | Provider key, e.g. `openai`, `gemini`, `elevenlabs` (use the `providerKey` value) |
| `filter[orderType]` | string | `cc`, `subtitle`, `document`, `aiDubbing`, `studioDubbing`, `proofreading`, `other`, `revision` |
| `filter[createdBy]` | string | Requestor — accepts a user ID or an email address |
| `filter[orderId]` | string | A single order ID |
| `filter[tag]` | string | A single program tag value, e.g. `Energy Studio` |

### Response (200)
```json
{
  "data": [
    {
      "id": "string",
      "orderId": "string",
      "orderName": "string",
      "createdBy": { "id": "string", "name": "string", "email": "string" },
      "orderType": "document",
      "creditsUsed": 9923,
      "usdEquivalent": 9.92,
      "provider": "Elevenlabs",
      "providerKey": "elevenlabs",
      "service": "Text to Speech",
      "tags": ["Energy Studio"],
      "occurredAt": "ISO8601",
      "projectId": "string"
    }
  ],
  "meta": {
    "page": 1,
    "take": 10,
    "itemCount": 1,
    "pageCount": 1,
    "hasNextPage": false,
    "hasPreviousPage": false
  }
}
```

### Example (curl)

> **Important:** Always use the `-g` (`--globoff`) flag with curl when query params contain brackets `[]`.

```bash
# Recent consumption, newest first
curl -g "https://api-integration.ollang.com/integration/consumption?pageOptions[orderBy]=occurredAt&pageOptions[orderDirection]=desc" \
  -H "X-Api-Key: $OLLANG_API_KEY"

# Filter by date range and provider
curl -g "https://api-integration.ollang.com/integration/consumption?filter[from]=2026-01-01T00:00:00Z&filter[to]=2026-01-31T23:59:59Z&filter[provider]=elevenlabs" \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to run `export OLLANG_API_KEY=<your-api-key>` in their own terminal — never ask them to share the key in the conversation
2. Determine the action: **wallet balance** or **consumption breakdown**
3. For **wallet**: display credits and USD equivalents; highlight `remainingUsd`
4. For **consumption**: ask which filters to apply (date range, provider, order type, requestor, order ID, tag); **always include `-g`** in curl commands for bracket params
5. Display consumption in a table: Occurred At, Order, Type, Provider, Service, Credits, USD, Requestor
6. On `403`, explain the key's user needs the `client_owner` or `client_accessBillings` role

## Error Codes
- `400` - Invalid query parameters
- `401` - Invalid or missing API key
- `403` - API key user lacks the `client_owner` or `client_accessBillings` role
