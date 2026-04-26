---
name: ollang-custom-instructions
description: Manage custom translation instructions — create, list, update, and delete per-account guidelines that guide AI translators. Use when the user wants to set up or modify translation preferences, rules, or constraints for their account.
---

# Ollang Custom Instructions

Create, manage, and track custom translation instructions for your account. These instructions guide AI translation workflows and ensure consistent terminology, tone, and style.

## Authentication

All requests require the `X-Api-Key` header. The API key is read from the `OLLANG_API_KEY` environment variable. If not set, instruct the user to run: `export OLLANG_API_KEY=<your-api-key>` (get it from https://lab.ollang.com).

---

## List Custom Instructions

**GET** `https://api-integration.ollang.com/integration/custom-instructions`

Retrieve all custom instructions configured for your account.

### Response (200)
Array of instruction objects:
```json
[
  {
    "id": "string",
    "key": "string",
    "value": "string",
    "active": boolean,
    "description": "string",
    "scopeRefId": "string",
    "createdAt": "ISO8601",
    "updatedAt": "ISO8601"
  }
]
```

### Example
```bash
curl -X GET https://api-integration.ollang.com/integration/custom-instructions \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Create Custom Instruction

**POST** `https://api-integration.ollang.com/integration/custom-instructions`

Create a new custom instruction for your account.

### Request Body (JSON)
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | Yes | Machine-friendly identifier (e.g., `tone_and_register`, `technical_terms`) |
| `value` | string | Yes | The instruction text content |
| `description` | string | No | Human-readable explanation of the instruction's purpose |

### Response (201)
Returns created instruction object with `id`, `key`, `value`, `active`, `description`, `scopeRefId`, `createdAt`, `updatedAt`.

### Example
```bash
curl -X POST https://api-integration.ollang.com/integration/custom-instructions \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "tone_professional",
    "value": "Use formal, professional language suitable for business documents",
    "description": "Ensures translations maintain a professional tone for corporate clients"
  }'
```

---

## Update Custom Instruction

**PATCH** `https://api-integration.ollang.com/integration/custom-instructions/{instructionId}`

Update an existing custom instruction (supports partial updates).

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `instructionId` | string | Yes | Unique identifier of the instruction to update |

### Request Body (JSON)
All fields optional; include at least one:
| Field | Type | Description |
|-------|------|-------------|
| `key` | string | Updated machine-friendly name |
| `value` | string | Updated instruction text |
| `active` | boolean | Enable/disable flag |
| `description` | string | Updated description |

### Response (200)
Returns updated instruction object.

### Example
```bash
curl -X PATCH https://api-integration.ollang.com/integration/custom-instructions/INSTRUCTION_ID \
  -H "X-Api-Key: $OLLANG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "value": "Use formal, professional language for all business documents. Avoid colloquialisms.",
    "active": true
  }'
```

---

## Delete Custom Instruction

**DELETE** `https://api-integration.ollang.com/integration/custom-instructions/{instructionId}`

Remove a custom instruction permanently.

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `instructionId` | string | Yes | Unique identifier of the instruction to delete |

### Response
- **204 No Content** — Instruction deleted successfully

### Example
```bash
curl -X DELETE https://api-integration.ollang.com/integration/custom-instructions/INSTRUCTION_ID \
  -H "X-Api-Key: $OLLANG_API_KEY"
```

---

## Scope & Auto-Application

**Important:** Custom instructions are **client-scoped** (tied to your API key) and **automatically applied to all orders** created with that key. There is no per-order override — these are account-wide guidelines that the AI translator follows for every translation job.

To understand how instructions relate to your orders:
1. You create custom instructions via this skill (stored at your client level)
2. When you create an order (via `ollang-order-create`), the system automatically applies all active custom instructions
3. You cannot opt-out of specific instructions per order — disable instructions here if needed

**vs. Project Notes:** Project notes are different — they're context for a *specific project* (uploaded file) and visible in the project details, but they don't affect translation behavior the same way.

## Behavior

1. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
2. Determine the action: **list**, **create**, **update**, or **delete**
3. For **list**: Display all instructions in a table format with id, key, value, active status, and description
4. For **create**: Ask for `key`, `value`, and optional `description`; confirm before proceeding
5. For **update**: Ask for `instructionId` and which fields to update; confirm before proceeding
6. For **delete**: Ask for `instructionId`; confirm before deleting (this is destructive)
7. Return confirmation and instruction details
8. Suggest next steps (e.g., "Create an order to apply these instructions")

## Common Use Cases

- **Terminology glossary:** "Remember to translate 'AI' as 'IA' in French contexts"
- **Tone guidance:** "Use informal, conversational tone for social media content"
- **Cultural adaptation:** "Adapt idioms and cultural references for the target audience"
- **Technical constraints:** "Preserve all placeholders like {userName} and {date}"
- **Style guide:** "Use Oxford commas and American English spelling"

## Error Codes
- `400` - Invalid parameters or validation error
- `401` - Invalid or missing API key
- `403` - Access denied
- `404` - Instruction not found
- `409` - Conflict (e.g., duplicate key)
