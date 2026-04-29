---
name: setup
description: Set up and authenticate RecordRoom CLI. Use when the user asks to install, configure, or log in to RecordRoom, or when RecordRoom commands fail due to missing auth.
---

# RecordRoom Setup

## Goals
- Ensure `recordroom` CLI is installed and authenticated.
- Connect a data source (Langfuse) if not already connected.

## Workflow

1. Check if CLI is available: `npx recordroom --version`
2. Check auth: `npx recordroom auth status`
3. If not authenticated:
   - If `RECORDROOM_TOKEN` env var is set, it will be used automatically.
   - Otherwise run `npx recordroom auth login` — this opens a browser for the user to authorize.
   - Tell the user to complete authorization in the browser, then wait for confirmation.
4. Check connection: `npx recordroom status`
5. If no data source connected, guide through connecting Langfuse:
   ```bash
   npx recordroom connect langfuse \
     --public-key <PUBLIC_KEY> \
     --secret-key <SECRET_KEY> \
     --host <HOST>
   ```
6. Run initial sync: `npx recordroom sync`

## Headless / CI

- Set `RECORDROOM_TOKEN=rr_xxx` environment variable.
- Do not attempt `auth login` in non-interactive environments.
- Set `RECORDROOM_SERVER` to override the server URL if needed.

## Troubleshooting

- `recordroom: command not found` → ensure Node.js is installed, use `npx recordroom`
- Auth failures → re-run `npx recordroom auth login` or check `RECORDROOM_TOKEN`
- Connection failures → verify Langfuse API keys and host URL
