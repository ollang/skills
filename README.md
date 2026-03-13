# Ollang Claude Code Skills

A collection of [Claude Code](https://claude.ai/claude-code) skills for interacting with the [Ollang](https://ollang.com) translation platform API via natural language.

## What are Skills?

Skills are prompt-based extensions for Claude Code that trigger automatically based on your intent. Once installed, you can say things like *"upload this video to Ollang"* or *"check my recent orders"* and Claude will know exactly what to do.

---

## Installation

Copy the skill folders into your Claude Code skills directory:

```bash
# Clone or download this repo, then:
cp -r skills/ollang-* ~/.claude/skills/
```

Or install individually:

```bash
cp -r skills/ollang-upload ~/.claude/skills/
```

---

## Available Skills

| Skill | Trigger Phrase Examples | Description |
|-------|------------------------|-------------|
| `ollang-health` | "is Ollang up?", "check API status" | Health check — no auth required |
| `ollang-upload` | "upload this file to Ollang", "create a project" | Direct file upload + VTT upload |
| `ollang-order-create` | "create a subtitle order", "translate this to Spanish" | Create CC, subtitle, document, or dubbing orders |
| `ollang-order-get` | "check order status", "get order details" | Get a single order by ID |
| `ollang-orders-list` | "list my orders", "show recent translations" | Paginated order list with filters |
| `ollang-order-cancel` | "cancel order", "stop this translation" | Cancel an active order |
| `ollang-order-rerun` | "rerun order", "regenerate translation" | Reprocess with latest AI models |
| `ollang-revision` | "report subtitle error", "create a revision" | Create, list, delete revisions |
| `ollang-human-review` | "request human review", "cancel human review" | Upgrade/downgrade to linguist review |
| `ollang-qc-eval` | "run QC", "evaluate translation quality" | AI quality scores (accuracy, fluency, tone, cultural fit) |
| `ollang-project` | "list my projects", "get project details" | Browse and inspect projects |
| `ollang-folder` | "list my folders", "find folder ID" | Browse folder structure |

---

## Authentication

All skills (except `ollang-health`) require an API key from [lab.ollang.com](https://lab.ollang.com).

Pass it as the `X-Api-Key` header — Claude will ask for it if not provided.

---

## API Base URL

All requests go to:
```
https://api-integration.ollang.com
```

---

## Typical Workflow

```
1. ollang-upload     →  upload file, get projectId
2. ollang-order-create →  create order with projectId, get orderId
3. ollang-order-get  →  poll order status
4. ollang-qc-eval    →  run quality check on completed order
5. ollang-revision   →  report any issues
```

---

## Notes

- Query params using bracket notation (e.g. `pageOptions[page]`) require curl's `-g` flag (`--globoff`) to prevent URL malformat errors. All skills handle this automatically.
- Skills are compatible with Claude Code CLI v1.x and above.
- API docs: [api-docs.ollang.com](https://api-docs.ollang.com)
