---
name: connect
description: Connect RecordRoom to Langfuse or other trace sources. Use when the user needs to set up a new data source, provides API keys, or when status shows no connected sources.
---

# RecordRoom Connect

## Workflow

1. Check current status: `npx recordroom status`
2. If Langfuse is already connected, report status and stop.
3. If not connected, collect from the user:
   - **Public key** — starts with `pk-lf-`
   - **Secret key** — starts with `sk-lf-`
   - **Host** — defaults to `https://cloud.langfuse.com`
4. Run:
   ```bash
   npx recordroom connect langfuse \
     --public-key <PUBLIC_KEY> \
     --secret-key <SECRET_KEY> \
     --host <HOST>
   ```
5. Run `npx recordroom status` to confirm connection.
6. Run `npx recordroom sync` to pull initial traces.

## Where to find Langfuse API keys

- Open Langfuse → Settings → API Keys
- Create a new key if needed
- Copy the public key and secret key
- For self-hosted Langfuse, use your instance URL as the host

## Important

- Treat the secret key as sensitive — do not echo it back in summaries or logs.
- If the user provides keys inline, proceed directly without asking again.
