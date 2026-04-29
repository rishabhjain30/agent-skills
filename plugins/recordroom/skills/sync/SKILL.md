---
name: sync
description: Sync traces from connected sources into RecordRoom. Use when the user mentions traces, recent conversations, debugging agent behavior, checking latest runs, or before any investigation.
---

# RecordRoom Sync

## When to use
- Before investigating agent behavior or errors
- After deployments or config changes
- When the user asks about recent traces or conversations
- Periodically during active debugging sessions

## Workflow

1. Run sync:
   ```bash
   npx recordroom sync
   ```
2. Check status:
   ```bash
   npx recordroom status
   ```
3. Report: how many new traces were synced, total count, last sync time.
4. If sync fails with "No Langfuse connection", guide the user to run setup first.

## Options

- `--limit <N>` — max traces to sync per run (default: 500)
- Use `--limit 50` for quick checks, `--limit 500` for full sync

## After syncing

Tell the user they can open the RecordRoom web UI to browse traces in the inbox, or ask you to investigate patterns.
