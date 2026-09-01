---
name: ollang
description: Master skill for the Ollang translation platform. Routes to the right Ollang sub-skill based on intent — upload files, create orders, check status, manage revisions, run QC, manage translation memories, translate Figma files, export content and spreadsheets, check credits, browse projects and folders. Use when the user mentions Ollang or wants to perform any translation/captioning/dubbing/localization workflow.
---

# Ollang — Master Skill

This is the entry point for all Ollang API operations. Based on the user's intent, delegate to the appropriate sub-skill below.

## Sub-Skills

| Sub-Skill | When to Use |
|-----------|-------------|
| `ollang-health` | User wants to check if the API is up |
| `ollang-upload` | User wants to upload a video, audio, document, or VTT file |
| `ollang-custom-instructions` | User wants to create, manage, or view custom translation instructions |
| `ollang-order-create` | User wants to create a translation, CC, subtitle, or dubbing order |
| `ollang-order-get` | User wants to check the status or details of a specific order |
| `ollang-orders-list` | User wants to list, search, or filter their orders |
| `ollang-order-cancel` | User wants to cancel an order |
| `ollang-order-rerun` | User wants to rerun or regenerate an order |
| `ollang-revision` | User wants to report an issue or manage revisions on an order |
| `ollang-human-review` | User wants to request or cancel human (linguist) review |
| `ollang-qc-eval` | User wants to run a quality control evaluation on an order |
| `ollang-subtitle-embedding` | User wants subtitles burned/hardcoded into the video |
| `ollang-memory` | User wants to create or manage translation memories, or import translation units |
| `ollang-content` | User wants to export/import content translations (i18n strings) as JSON |
| `ollang-xlsx-export` | User wants an XLSX/Excel export of an order's or folders' segments |
| `ollang-figma` | User wants to translate a Figma design file or track Figma orders |
| `ollang-credits` | User wants their credit balance or a consumption/spend breakdown |
| `ollang-project` | User wants to list or inspect projects |
| `ollang-folder` | User wants to list or find folders, or batch-assign/unassign translators on a folder's orders |

## Full Workflow

A complete end-to-end translation workflow looks like this:

```
1. Upload file          →  ollang-upload        →  returns projectId
2. Create order         →  ollang-order-create  →  returns orderId(s)
3. Monitor status       →  ollang-order-get     →  poll until "completed"
4. Quality check        →  ollang-qc-eval       →  scores + segment analysis
5. Report issues        →  ollang-revision      →  create revisions if needed
6. Upgrade to human     →  ollang-human-review  →  optional linguist review
```

Optional steps: set up translation memories first (`ollang-memory`, applied via `selectedMemories` in `ollang-order-create`); after completion, burn subtitles into the video (`ollang-subtitle-embedding`) or export segments as a spreadsheet (`ollang-xlsx-export`).

## Authentication

All endpoints (except health check) require the `X-Api-Key` header.
The API key is read from the `OLLANG_API_KEY` environment variable.

If the variable is not set, instruct the user to configure it:
```bash
export OLLANG_API_KEY=<your-api-key>
```
Get your API key at https://lab.ollang.com.

## API Base URL

```
https://api-integration.ollang.com
```

## Language Codes

Language codes are mostly ISO 639-1 (`en`, `fr`, `de`), with regional and platform-specific variants for some locales. Watch for these platform conventions:

- `pt` = Portuguese (**Brazil**), `pt-PT` = Portuguese (Portugal)
- `es` = Spanish (**Spain**), `es-MX` = Spanish (LATAM)
- `zh` = Chinese (Simplified), `zh-Hant` = Chinese (Traditional), `zh-TW` = Chinese (Taiwan)
- `en` = English, `en-UK` = English (United Kingdom)
- Arabic regional variants like `ar-EG`, `ar-AE`; French Canadian is `fr-CA`

The full list lives at https://api-docs.ollang.com/apis/ollang-api-reference/supported-languages — pass the exact string shown there.

## Behavior

1. Identify the user's intent from their message
2. Map it to the correct sub-skill from the table above
3. Read the API key from the `OLLANG_API_KEY` environment variable. If not set, tell the user to set it with: `export OLLANG_API_KEY=<your-api-key>`
4. Execute the operation and present results clearly
5. Suggest logical next steps (e.g., after upload → offer to create an order)

---

## Reading Error Responses

API errors use a generic envelope with three fields — **always inspect the `detail` field**, not `error` or `message`, as it's the only signal that distinguishes them:

```json
{
  "code": -1,
  "error": "UnknownError",
  "message": "unknown error",
  "detail": "<actual message — this is what matters>"
}
```

### Common Detail Substrings & Root Causes

| Detail Substring | Meaning | Fix |
|------------------|---------|-----|
| `Cannot POST /...` | Wrong endpoint path (typo) | Check endpoint URL in the skill docs |
| `Unexpected token` | Request body is not valid JSON | Check shell quoting in curl `-d` flag; use single quotes around JSON |
| `The first argument must be of type string ...` | Downstream microservice bug with Node fetch (chunked encoding issue) | Use curl instead, or set `Content-Length` header explicitly |
| `Invalid or missing API key` | API key missing or invalid | Run `export OLLANG_API_KEY=<your-api-key>` and get key from https://lab.ollang.com |

---

## Recommended HTTP Client

**Prefer curl** for all requests. Node 18+ built-in fetch has a known issue with chunked transfer encoding that causes 500s on some endpoints.

If using Node fetch (or axios), **always set the `Content-Length` header explicitly** to avoid chunked encoding. Alternatively, use `undici` or `node-fetch` with Content-Length enforcement.

---

## Custom Instructions Scope

Custom instructions created via `ollang-custom-instructions` are **client-scoped** (tied to your API key) and **automatically applied to all orders** created with that key. There is no per-order override — instructions are account-wide guidance for the AI translator.

**Distinction:**
- **Custom Instructions** — AI guidance rules (tone, terminology, style) — apply to all orders
- **Project Notes** — Context for a specific project — visible only on that project
