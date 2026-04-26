---
name: ollang-order-create
description: Create a translation order on the Ollang platform. Supports closed captions, subtitles, document translation, AI dubbing, and studio dubbing. Use when the user wants to start a translation, captioning, or dubbing job.
---

# Ollang Create Order

Create a new translation/captioning/dubbing order for an existing project.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

## Endpoint

**POST** `https://api-integration.ollang.com/integration/orders/create`

## Request Body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `projectId` | string | Yes | Project ID from file upload |
| `orderType` | string | Yes | Type of order (see Order Types below) |
| `level` | number | Yes | `0` = AI-generated, `1` = human-reviewed |
| `targetLanguageConfigs` | array | Yes | Target languages with optional rush delivery |
| `orderSubType` | string | No | `closedCaption` or `timecodedTranscription` |
| `dubbingStyle` | string | No | `overdub`, `lipsync`, or `audioDescription` |
| `autoQc` | boolean | No | Trigger automatic QC evaluation after completion |

### Order Types
| Value | Description |
|-------|-------------|
| `cc` | Closed Captions |
| `subtitle` | Subtitles (translation) |
| `document` | Document Translation |
| `aiDubbing` | AI Dubbing |
| `studioDubbing` | Studio Dubbing |

### targetLanguageConfigs Format
```json
[
  { "language": "es", "isRush": false },
  { "language": "fr", "isRush": true }
]
```

## Response (201)
Array of order objects, one per target language:
```json
[
  { "orderId": "507f1f77bcf86cd799439015" },
  { "orderId": "507f1f77bcf86cd799439016" }
]
```

## Example (curl)
```bash
curl -X POST https://api-integration.ollang.com/integration/orders/create \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "projectId": "PROJECT_ID",
    "orderType": "subtitle",
    "level": 1,
    "targetLanguageConfigs": [
      { "language": "es", "isRush": false },
      { "language": "fr", "isRush": false }
    ],
    "autoQc": true
  }'
```

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Confirm the `projectId` (from a previous upload or get-project)
3. Determine `orderType` and `level` from the user's intent
4. Collect target languages — support multiple at once
5. Submit the order and return all `orderId` values
6. Save the `orderId` values — needed for tracking, revisions, and QC

## Custom Instructions

Custom instructions created via `ollang-custom-instructions` are **automatically applied to all orders** created with your API key. You cannot override them per-order — they're account-wide AI guidance rules.

If you want different translation behavior for a specific order, you must:
1. Disable the custom instructions that conflict (via `ollang-custom-instructions` → update → set `active: false`)
2. Create the order
3. Re-enable the instructions afterward

There is no `customInstructionId` parameter in the create order request — instructions are applied implicitly based on your client account.

## Error Codes
- `400` - Invalid parameters (bad order type, invalid language code, etc.)
- `401` - Invalid or missing API key
- `404` - Project not found
