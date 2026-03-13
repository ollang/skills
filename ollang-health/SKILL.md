---
name: ollang-health
description: Check the health status of the Ollang integration API. Use when the user wants to verify the API is up and running, or diagnose connection issues.
---

# Ollang Health Check

Check the availability and response time of the Ollang integration API. This is a public endpoint — no authentication required.

## Endpoint

**GET** `https://api-integration.ollang.com/health`

### Authentication
None required — public endpoint.

## Response (200 - Healthy)
```json
{
  "status": "OK",
  "time": 42
}
```

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `OK` if healthy, `NOK` if degraded |
| `time` | number | Response time in milliseconds |

## Response Codes
| Code | Status |
|------|--------|
| `200` | API is healthy |
| `500` | Internal server error |
| `503` | Service unavailable |

## Example (curl)
```bash
curl -X GET https://api-integration.ollang.com/health
```

## Behavior

1. Send a GET request to the health endpoint
2. Display the status and response time
3. If `status` is `OK`, confirm the API is operational
4. If `status` is `NOK` or a 5xx error is returned, report the issue and suggest checking https://status.ollang.com or retrying later
