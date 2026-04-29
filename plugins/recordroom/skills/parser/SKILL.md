---
name: parser
description: Create or modify trace parsers for RecordRoom accounts. Use when the user asks to set up a parser, write a parser, fix a parser, or onboard a new account's trace format.
---

# RecordRoom Parser

Create JavaScript `parseTrace(data)` functions that transform raw trace data into RecordRoom's standard format.

## Workflow

### 1. Understand the data

Fetch several traces from the datasource to understand the raw structure:
- Use existing APIs or database access to get raw trace data
- Look at diverse traces to discover all message types, roles, and edge cases
- Note: don't rely on just 1-2 traces — examine enough to see the full variety

### 2. Understand the intent

Ask the user:
- What does this account/product do?
- Which message types matter? Which are noise to skip?
- How should complex content (tool calls, citations, media) be rendered?
- Any specific fields or attachments to highlight?

### 3. Write the parser

Create a `parseTrace(data)` function in a local `.js` file:

```javascript
function parseTrace(data) {
  var items = [];
  var turnNum = 0;
  // ... transform data into items ...
  return {
    items: items,
    metadata: { /* optional trace-level metadata */ }
  };
}
```

**Output schema — `ParsedTrace`:**
- `items[]` — array of `TraceItem`:
  - `turn` (int, required) — sequential starting at 1
  - `role` (string, required) — one of: `user`, `assistant`, `tool`, `system`
  - `content` (string, required) — non-empty text content
  - `tool_name` (string, optional) — name of tool for `role: "tool"`
  - `attachments` (array, optional) — list of `Attachment`
  - `timestamp` (string, optional) — ISO timestamp
  - `raw_index` (int, optional) — index in original data
- `metadata` (object, optional) — trace-level info (trace_id, duration, etc.)

**Attachment types:** `image`, `video`, `audio`, `pdf`, `citation`, `json`, `link`

Each attachment has: `type`, `url` (optional), `label` (optional), `inline_text` (optional), `metadata` (optional object).

**Reference parsers** in the codebase:
- `backend/app/services/trace_parser/parsers/frameo.js` — complex, with tool call handlers and media extraction
- `backend/app/services/trace_parser/parsers/finrep.js` — simpler, with citation parsing

### 4. Validate iteratively

```bash
npx recordroom parser validate parser.js --datasource <id> --json
```

- Read the JSON output, fix any failures
- Repeat until all traces pass
- Pay attention to: empty items, missing content, invalid roles, malformed attachments
- Use `--samples <n>` to test against more traces (default: 10)
- Supports stdin: `cat parser.js | npx recordroom parser validate - --datasource <id> --json`

### 5. Deploy

```bash
npx recordroom parser apply parser.js --datasource <id>
```

This validates first, then deploys. Previous version is automatically saved to history.

## Other commands

- `npx recordroom parser show --datasource <id>` — fetch current deployed parser
- `npx recordroom parser show --datasource <id> --output parser.js` — save to file
- `npx recordroom parser versions --datasource <id>` — list version history
- `npx recordroom parser rollback --datasource <id>` — restore previous version

## Tips

- Use ES5-style JavaScript (var, no arrow functions) — the V8 sandbox is minimal
- Parser has a 5-second timeout — avoid expensive operations
- No network access, no filesystem, no external modules in the sandbox
- Keep parsers self-contained — all helper functions inline
- Test against diverse traces, not just the first few
