---
name: recordroom-tasks
description: Build and run regression test suites from confirmed issues. Use when the user asks to create tests, run benchmarks, check pass rates, generate task candidates, ship drafts, or verify fixes.
---

# RecordRoom Tasks — Benchmark Workflow

You are being asked to work with the regression test suite. Tasks are test cases derived from real conversations where issues were detected. Each task replays a conversation against the agent's API and grades the response.

## Prerequisites

- CLI must be authenticated: `npx recordroom auth status`
- Issues must exist (tasks require a parent issue): `npx recordroom issues list`
- An API config must exist for running tasks: `npx recordroom api-config list`

## How Tasks Work

Tasks sit on top of the evaluation test infrastructure. Every task:
1. Has a **parent issue** that defines the quality problem
2. **Inherits expected behavior** from the parent issue (or can override it)
3. Replays a multi-turn conversation against the agent API
4. Uses an LLM judge to grade the response as pass/fail

When a task linked to a resolved issue fails, the issue auto-regresses — creating a closed-loop quality system.

## Your Workflow

### Step 1: Check the benchmark

```bash
recordroom tasks benchmark
```

Shows overall pass rate, per-category breakdown, and counts of passing/failing/draft tasks. This is the health dashboard.

### Step 2: Generate tasks from an issue

After confirming an issue (see `recordroom-issues` skill), generate test candidates:

```bash
recordroom tasks generate <issue-id>
```

This uses AI to draft test cases from the issue's linked conversations. Review the candidates — they show the conversation turns, which turn is evaluated, and the source trace.

To persist the drafts and optionally auto-run them:
```bash
recordroom tasks ship <issue-id>
```

### Step 3: Review tasks

```bash
recordroom tasks list                           # all tasks
recordroom tasks list --failing                 # only failing
recordroom tasks list --drafts                  # only drafts
recordroom tasks list --issue <id>              # tasks for a specific issue
recordroom tasks list --category "Refund flow"  # by category

recordroom tasks show <task-id>                 # full detail with grading info
```

The list view shows a sparkline of recent run results so you can spot trends at a glance.

### Step 4: Run tasks

**Single task:**
```bash
recordroom tasks run <task-id>
```

**All tasks (full benchmark):**
```bash
recordroom tasks run --all
```

**Only failing tasks (re-verify):**
```bash
recordroom tasks run --failing
```

**Tasks for a specific issue:**
```bash
recordroom tasks run --issue <issue-id>
```

All batch runs poll until complete and show a summary with pass rate.

### Step 5: Review run history

```bash
recordroom tasks runs <task-id>     # per-task run history
recordroom runs list                # all runs across the project
recordroom runs show <run-id>       # per-task results with diff markers
```

The runs view shows which tasks newly passed or failed compared to the previous run.

### Step 6: Rerun or iterate

```bash
recordroom runs rerun <run-id>                  # rerun same task set
recordroom runs rerun <run-id> --only-failing   # rerun just failures
```

### Local testing (run locally, grade remotely)

When your agent isn't reachable from RecordRoom's servers (e.g., running on localhost), use the fetch + grade workflow:

```bash
# 1. Fetch the test case
recordroom tasks fetch <task-id>
# Shows: turns, evaluated turn, expected behavior, parent issue

# 2. Run against your local agent (however you want)
# curl, script, Claude Code calling functions — anything goes

# 3. Submit the output for grading
recordroom tasks grade <task-id> --output "The agent's response"
# Or from a file:
recordroom tasks grade <task-id> --output-file response.txt
```

The grade counts as a real run — it appears in run history, updates the benchmark, and triggers auto-regression if the task fails on a resolved issue.

**For Claude Code agents**, the workflow is:
```bash
recordroom tasks fetch <task-id> --json    # machine-readable test case
# (Claude Code runs against local agent)
recordroom tasks grade <task-id> --output "response from local agent"
```

**Batch local testing:**
```bash
recordroom tasks list --failing --json     # get all failing task IDs
# For each: fetch → run locally → grade
recordroom tasks benchmark                 # check updated pass rate
```

## Command Reference

| Command | Purpose |
|---------|---------|
| `recordroom tasks benchmark` | Pass rate + category breakdown |
| `recordroom tasks list` | List all tasks |
| `recordroom tasks list --failing` | Only failing tasks |
| `recordroom tasks list --drafts` | Only draft tasks |
| `recordroom tasks show <id>` | Full task detail |
| `recordroom tasks create --issue <id> --input "..."` | Manual task creation |
| `recordroom tasks runs <id>` | Run history for a task |
| `recordroom tasks run <id>` | Run one task |
| `recordroom tasks run --all` | Run all tasks |
| `recordroom tasks run --failing` | Re-run failing only |
| `recordroom tasks run --issue <id>` | Run tasks for an issue |
| `recordroom tasks generate <issue-id>` | AI-generate task candidates |
| `recordroom tasks ship <issue-id>` | Promote drafts to live + auto-run |
| `recordroom tasks fetch <id>` | Fetch test case for local execution |
| `recordroom tasks grade <id> --output "..."` | Grade locally-produced output |
| `recordroom tasks grade <id> --output-file f.txt` | Grade from file |
| `recordroom tasks delete <id>` | Deactivate a task |
| `recordroom runs list` | List all runs |
| `recordroom runs show <id>` | Run detail with per-task results |
| `recordroom runs rerun <id>` | Re-run a previous run |
| `recordroom runs cancel <id>` | Cancel a running batch |

All commands support `--json` for structured output.

## Key Concepts

- **Inheritance** — Tasks inherit `expected_behavior` from their parent issue. If the PM updates the issue's expected behavior, all inheriting tasks pick up the change on their next run. Tasks can override this with their own grading criteria.
- **Drafts** — Generated tasks start as drafts. They can be run for preview but don't count in the benchmark until shipped (promoted to live).
- **Task categories** — Capability dimensions like "Refund flow" or "Tool grounding". Auto-assigned at issue creation. The benchmark view breaks down pass rate by category.
- **Sparkline** — The task list shows a mini history of recent pass/fail results per task: green dots = pass, red dots = fail.
- **Auto-regression** — A failing task on a resolved issue automatically moves the issue to `regressed` state.
