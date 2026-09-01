---
name: ollang-subtitle-embedding
description: Request subtitle embedding (burn-in) for an Ollang order so translated subtitles are hardcoded into the video file. Use when the user wants burned-in/hardcoded subtitles or an embedded-subtitle video.
---

# Ollang Subtitle Embedding

Burn (hardcode) an order's translated subtitles directly into the video stream. The resulting video has the subtitles permanently embedded.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

## Endpoint

**POST** `https://api-integration.ollang.com/integration/orders/{orderId}/subtitle-embedding`

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `orderId` | string | Yes | The order to embed subtitles for. Must be a **completed or delivered video order** with available subtitle files. |

### Request Body
No request body required.

## Response (200)
```json
{
  "success": true,
  "message": "Subtitle embedding initiated successfully",
  "orderId": "string"
}
```

## Example (curl)
```bash
curl -X POST https://api-integration.ollang.com/integration/orders/ORDER_ID/subtitle-embedding \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Ask for the `orderId` if not provided
3. Confirm the order is a completed/delivered video order with subtitles (check via `ollang-order-get` if unsure)
4. Send the embedding request; the process runs asynchronously
5. Suggest polling `ollang-order-get` — the embedded video appears in `orderDocs` (document types like `created_embedded_video` / `created_subtitle_embedded_video`)

## Error Codes
- `400` - Order not eligible (only completed video orders with subtitle files can be embedded)
- `401` - Invalid or missing API key
- `403` - Subtitle embedding not available in the current plan
- `404` - Order not found
