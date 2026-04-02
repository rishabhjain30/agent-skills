---
name: recordroom-for-claude
description: RecordRoom awareness — monitors AI agent conversations via Langfuse integration. Use when the user asks about trace quality, agent performance, error patterns, wants to check recent agent activity, reproduce issues, or run evaluation tests.
---

# RecordRoom

RecordRoom monitors AI agent conversations. It syncs traces from Langfuse (and other sources) into an inbox for review, annotation, and analysis. Issues found in traces become evaluation tests that can be replayed against the agent's API.

## Available commands

**Setup & sync:**
- `npx recordroom auth login` — authenticate via browser
- `npx recordroom connect langfuse` — connect a Langfuse instance
- `npx recordroom sync` — pull latest traces
- `npx recordroom status` — show connection and sync status

**API configuration (for running evals):**
- `npx recordroom api-config list` — list API configurations
- `npx recordroom api-config apply --file <path>` — create/update API config from JSON
- `npx recordroom api-config test <id> --input <text>` — test an API config
- `npx recordroom api-config show <id>` — inspect config details
- `npx recordroom api-config export <id>` — export config as JSON

**Evaluation tests:**
- `npx recordroom evals list` — list evaluation tests with pass/fail status
- `npx recordroom evals show <test-id>` — show full test briefing
- `npx recordroom evals run [test-id]` — run a test and show result
- `npx recordroom evals run --all` — run all active tests
- `npx recordroom evals run --failed` — re-run only failing tests

**Findings:**
- `npx recordroom findings list` — list analysis findings
- `npx recordroom findings summary` — show findings summary
- `npx recordroom findings accept <id>` — accept a finding (creates eval test)
- `npx recordroom findings dismiss <id>` — dismiss a finding

**Traces:**
- `npx recordroom traces show <trace-id>` — show raw trace data

## When to use RecordRoom

- User asks about recent agent traces or conversations
- User wants to debug agent behavior or errors
- User asks about trace quality, token usage, or costs
- User wants to connect monitoring for their AI agents
- User wants to reproduce or verify an agent issue
- User needs to set up API testing for their agent
- User wants to run evaluation tests or regression checks
- Before deploying changes that affect agent behavior (sync and check first)

## Quick start

If RecordRoom is not set up yet, run:
```bash
npx recordroom auth login
npx recordroom connect langfuse --public-key <KEY> --secret-key <KEY>
npx recordroom sync
```

## Reproduce issues

If eval tests exist but no API config is set up, see the `recordroom-api-config` skill to create one. Then:
```bash
npx recordroom evals run --all   # run all tests
```

See the `recordroom-evals` skill for the full workflow: diagnose, fix, verify, regression check.
