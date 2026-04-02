---
name: recordroom-evals
description: Fix failing evaluation tests, run regression checks, and diagnose agent issues. Use when the user asks to fix evals, reproduce issues, run tests, or debug why an eval is failing.
---

# RecordRoom Evals — Agent Workflow

You are being asked to fix a failing evaluation test. RecordRoom tracks AI agent quality issues as eval tests — each test replays a real user conversation against the system and checks if the output meets specific criteria.

## Prerequisites

- CLI must be authenticated: `npx recordroom auth status`
- A datasource must be connected: `npx recordroom status`
- If either fails, follow the recordroom-setup workflow first
- An API config must exist: `npx recordroom api-config list`
- If no API config exists, see the `recordroom-api-config` skill to create one

## Your Workflow

### Step 1: Identify what's failing

If the user gave you a URL like `https://app.recordroom.ai/evaluations?testId=f6f29e96-...`, extract the `testId` parameter and skip to Step 2.

If the user said "fix failing evals" without a specific ID:

```
recordroom evals list
```

This shows all tests with their status (PASS/FAIL/not run), severity, and short ID.

Use `--json` for machine-parseable output:
```
recordroom evals list --json
```

Filter to only failing or by severity:
```
recordroom evals list --status failing
recordroom evals list --severity critical
```

### Step 2: Understand the issue

```
recordroom evals show <test-id>
```

Accepts full or partial UUID (e.g., `f6f29e96` works). This returns everything you need:

- **Issue** — The original human feedback describing what went wrong in production
- **Original Conversation** — The full multi-turn trace as it happened, in `Turn N [ROLE]: content` format with attachment references. This is the real production conversation where the issue was observed.
- **Test Replay** — The subset of turns that get replayed when running this test. One turn is marked as "evaluated" — that's the turn whose output the grader judges.
- **Expected Behavior** — What the system should do instead
- **Grader Prompt** — The exact prompt the LLM judge uses to determine pass/fail. This is your acceptance criteria.
- **Last Run Result** — The most recent test result, including:
  - The full replayed conversation (every user turn paired with the system's actual response)
  - The grader's verdict: pass/fail, confidence percentage, and reasoning
  - HTTP status code and execution time
- **API Config** — The endpoint, method, and request chain being tested. This tells you which service and endpoint to look at in the codebase.

Use `--json` for machine-parseable output if you need to process the data programmatically:
```
recordroom evals show <test-id> --json
```

### Step 3: Diagnose

Read the grader reasoning carefully. There are two types of failures:

**The system produces wrong output** — The code has a bug or missing logic. The grader reasoning will describe what's wrong with the response content. Search the codebase for the relevant handler/service based on the API config endpoint.

**The test infrastructure fails** — You see HTTP errors (504, 401, etc.), HTML error pages, or "payment_required" responses. This is an API config or environment issue, not a code bug. See the `recordroom-api-config` skill for full config structure, variable resolution, and debugging guide. Quick inspection:

```
recordroom api-config show <config-id>
```

To fix an API config:
```
recordroom api-config export <config-id> > config.json
# edit config.json
recordroom api-config apply --file config.json
```

### Step 4: Fix the code

Make your changes based on what you learned from the show output.

### Step 5: Verify the fix

```
recordroom evals run <test-id>
```

This executes the test, polls until the run completes, and shows:
- The full replayed conversation with the system's new responses
- The grader's verdict with reasoning

The command waits for completion automatically — do not run additional commands while it's polling. Wait for the final output.

If it passes, proceed. If it fails, read the new reasoning and iterate.

### Step 6: Regression check

```
recordroom evals run --all
```

Runs all active tests and shows a summary of pass/fail counts. Make sure your fix didn't break other tests.

To re-run only previously failing tests:
```
recordroom evals run --failed
```

## Command Reference

| Command | Purpose |
|---------|---------|
| `recordroom evals list` | List all tests with status |
| `recordroom evals list --status failing` | Show only failing tests |
| `recordroom evals list --severity critical` | Filter by severity |
| `recordroom evals show <id>` | Full test briefing |
| `recordroom evals run <id>` | Run one test, show result |
| `recordroom evals run --all` | Run all active tests |
| `recordroom evals run --failed` | Re-run failing tests only |
| `recordroom api-config show <id>` | Inspect API config details |
| `recordroom api-config export <id>` | Export config as JSON |
| `recordroom api-config apply --file <path>` | Update config from JSON |

All commands support `--json` for structured output.

## Key Concepts

- **Evaluated turn** — In multi-turn tests, only one turn's output is judged. The other turns provide context (setup steps, prior conversation). The evaluated turn is marked in the output.
- **Grader prompt** — An LLM acts as judge. The grader prompt defines pass/fail criteria. If the grader prompt seems wrong, that's a human issue — flag it to the user rather than trying to fix it.
- **Original vs Replay** — The "Original Conversation" shows what happened in production (may be 20+ turns). The "Test Replay" is a shorter subset that reproduces the issue. They may show different results because the replay runs in a fresh context without the intermediate state built up in the original.
- **API Config** — Defines how to call the system under test: authentication steps, project creation, and the actual chat endpoint. Each step can be setup-only or per-turn. See the `recordroom-api-config` skill for full details.
