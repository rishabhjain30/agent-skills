---
name: recordroom-for-claude
description: RecordRoom awareness — monitors AI agent conversations, triages quality issues, builds regression test suites. Use when the user asks about trace quality, agent performance, error patterns, wants to triage issues, run benchmarks, reproduce issues, or manage evaluation tests.
---

# RecordRoom

RecordRoom monitors AI agent conversations. It syncs traces from Langfuse (and other sources) into an inbox for review. The analysis pipeline detects findings, groups them into issues, and users build regression test tasks from confirmed issues.

## Core workflow

1. **Sync** traces from your data source
2. **Analyze** conversations with skills to detect findings
3. **Triage** findings into issues (confirm or ignore)
4. **Test** — generate regression tasks from confirmed issues
5. **Benchmark** — run the test suite, track pass rate over time

## Available commands

**Setup & sync:**
- `npx recordroom auth login` — authenticate via browser
- `npx recordroom connect langfuse` — connect a Langfuse instance
- `npx recordroom sync` — pull latest traces
- `npx recordroom status` — show connection and sync status

**Skills & analysis:**
- `npx recordroom skills list` — list analysis skills
- `npx recordroom skills create --name "..." --description "..."` — create a skill
- `npx recordroom skills run <id> --all` — run analysis against traces
- `npx recordroom skills preview <id> --trace <trace-id>` — preview LLM input

**Findings:**
- `npx recordroom findings list` — list analysis findings
- `npx recordroom findings show <id>` — full finding detail
- `npx recordroom findings summary` — show findings summary
- `npx recordroom findings create --trace <id> --issue <id> --description "..."` — create manual finding
- `npx recordroom findings confirm <id>` — confirm a finding
- `npx recordroom findings accept <id>` — accept a finding
- `npx recordroom findings edit-accept <id> --description "..."` — accept with edit
- `npx recordroom findings dismiss <id>` — dismiss a finding
- `npx recordroom findings update <id> --severity high` — change severity/category/issue
- `npx recordroom findings categories` — list finding categories

**Issues:**
- `npx recordroom issues list` — list quality issues
- `npx recordroom issues list --state for_review` — issues awaiting triage
- `npx recordroom issues list --search "..."` — search title/description
- `npx recordroom issues show <id>` — full issue detail + findings summary
- `npx recordroom issues conversations <id>` — conversations linked to an issue
- `npx recordroom issues create --title "..."` — manually create an issue
- `npx recordroom issues update <id> --severity critical` — update issue fields
- `npx recordroom issues confirm <id> --expected-behavior "..."` — confirm as real
- `npx recordroom issues ignore <id>` — mark as noise
- `npx recordroom issues resolve <id>` — mark as resolved
- `npx recordroom issues reopen <id>` — reopen a resolved issue

**Tasks (regression tests):**
- `npx recordroom tasks benchmark` — pass rate + category breakdown
- `npx recordroom tasks list` — list all tasks
- `npx recordroom tasks show <id>` — full task detail
- `npx recordroom tasks run <id>` — run a single task
- `npx recordroom tasks run --all` — run all tasks (benchmark)
- `npx recordroom tasks run --failing` — re-run failing only
- `npx recordroom tasks generate <issue-id>` — AI-generate test candidates
- `npx recordroom tasks ship <issue-id>` — promote drafts to live
- `npx recordroom tasks create --issue <id> --input "..."` — manually create a task
- `npx recordroom tasks fetch <id>` — fetch test case for local execution
- `npx recordroom tasks grade <id> --output "..."` — grade locally-produced output
- `npx recordroom tasks update <id> --severity critical` — update task fields
- `npx recordroom tasks runs <id>` — run history for a task
- `npx recordroom tasks categories` — list task categories

**Runs (test execution history):**
- `npx recordroom runs list` — list recent runs
- `npx recordroom runs show <id>` — per-task results with diff markers
- `npx recordroom runs results <id>` — full grader verdicts for a run
- `npx recordroom runs rerun <id>` — re-run same task set
- `npx recordroom runs cancel <id>` — cancel a running batch

**Evaluation tests (legacy):**
- `npx recordroom evals list` — list evaluation tests with pass/fail status
- `npx recordroom evals show <test-id>` — show full test briefing
- `npx recordroom evals update <test-id> --grader "..."` — update grader/severity
- `npx recordroom evals run [test-id]` — run a test and show result
- `npx recordroom evals run --all` — run all active tests

**API configuration (for running evals/tasks):**
- `npx recordroom api-config list` — list API configurations
- `npx recordroom api-config apply --file <path>` — create/update API config
- `npx recordroom api-config test <id> --input <text>` — test an API config
- `npx recordroom api-config show <id>` — inspect config details
- `npx recordroom api-config export <id>` — export config as JSON

**Traces:**
- `npx recordroom traces list` — list synced traces
- `npx recordroom traces show <trace-id>` — show raw trace data

## When to use RecordRoom

- User asks about recent agent traces or conversations
- User wants to debug agent behavior or errors
- User wants to triage quality issues or review findings
- User wants to build regression tests from real conversations
- User wants to run benchmarks or check pass rates
- User wants to connect monitoring for their AI agents
- User wants to reproduce or verify an agent issue
- User needs to set up API testing for their agent
- Before deploying changes that affect agent behavior (run tasks first)

## Quick start

If RecordRoom is not set up yet, run:
```bash
npx recordroom auth login
npx recordroom connect langfuse --public-key <KEY> --secret-key <KEY>
npx recordroom sync
```

## Reproduce issues

See the `recordroom-issues` skill for the triage workflow, and `recordroom-tasks` for building and running regression tests.

For legacy eval tests without parent issues, see the `recordroom-evals` skill.
