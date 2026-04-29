---
name: issues
description: Triage quality issues detected in AI agent conversations. Use when the user asks to review issues, triage findings, confirm or ignore issues, manage issue state, or investigate recurring patterns.
---

# RecordRoom Issues — Triage Workflow

You are being asked to triage quality issues detected in AI agent conversations. Issues are patterns of failure grouped from individual findings — each issue may appear across many conversations.

## Prerequisites

- CLI must be authenticated: `npx recordroom auth status`
- A project must be selected (happens automatically during login)

## Issue Lifecycle

```
  for_review ──→ open ──→ resolved
       │            │          │
       │            │          ↓
       │            └──→ regressed ──→ resolved
       │
       └──→ ignored
```

- **for_review** — Newly detected by the analysis pipeline, awaiting human triage
- **open** — Confirmed as a real issue, expected behavior defined
- **resolved** — Fix deployed, tests passing
- **regressed** — Was resolved, but a task started failing again (auto-set)
- **ignored** — Marked as noise / not actionable

## Your Workflow

### Step 1: See what needs triage

```bash
recordroom issues list --state for_review
```

Shows issues sorted by impact (occurrence count x severity). Each row shows severity, state, occurrence count, and title.

Other useful filters:
```bash
recordroom issues list                          # all issues, sorted by impact
recordroom issues list --state open             # confirmed issues
recordroom issues list --severity critical      # critical only
recordroom issues list --sort trend             # most recently seen first
recordroom issues list --search "refund"        # search title and description
recordroom issues list --date-from 2026-04-01   # recent issues only
```

### Step 2: Investigate an issue

```bash
recordroom issues show <issue-id>
```

Shows the full issue detail: title, description, severity, occurrence count, expected behavior, and a findings summary (how many findings are pending/confirmed/dismissed).

To see which conversations are affected:
```bash
recordroom issues conversations <issue-id>
```

Lists linked conversations with user info, finding counts, and the turn where the issue was detected. Use `recordroom traces show <trace-id>` to inspect a specific conversation.

To review the individual findings on those conversations:
```bash
recordroom findings list --status pending       # all pending findings
recordroom findings show <finding-id>           # detail of one finding
```

### Step 3: Triage findings

Before confirming the issue, you can triage individual findings:

```bash
recordroom findings confirm <finding-id>                              # confirm a finding
recordroom findings accept <finding-id>                               # accept (adds positive example to skill)
recordroom findings edit-accept <finding-id> --description "..."      # accept with corrected description
recordroom findings dismiss <finding-id>                              # dismiss (adds negative example)
```

### Step 4: Decide on the issue

**Confirm** — The issue is real. You must define what the system should do instead:
```bash
recordroom issues confirm <issue-id> --expected-behavior "The agent should refuse to answer medical questions and redirect to a doctor."
```
This transitions the issue to `open` and unlocks task generation.

**Ignore** — The issue is noise or not worth fixing:
```bash
recordroom issues ignore <issue-id> --reason "User was testing the system"
```

**Update** — Edit title, description, severity, or state:
```bash
recordroom issues update <issue-id> --severity critical
recordroom issues update <issue-id> --title "New title" --description "Better description"
recordroom issues update <issue-id> --state resolved
```

### Step 5: Generate regression tests

After confirming, generate test tasks from the issue's evidence:
```bash
recordroom tasks generate <issue-id>
```

See the `tasks` skill for the full task workflow.

### Step 6: Resolve when fixed

After deploying a fix and verifying tasks pass:
```bash
recordroom issues resolve <issue-id>
```

If a task later fails, the issue auto-regresses back to `regressed`.

## Command Reference

| Command | Purpose |
|---------|---------|
| `recordroom issues list` | List all issues |
| `recordroom issues list --state for_review` | Issues awaiting triage |
| `recordroom issues list --severity critical` | Critical issues only |
| `recordroom issues list --search "..."` | Search title/description |
| `recordroom issues list --category <id>` | Filter by task category |
| `recordroom issues list --date-from YYYY-MM-DD` | Issues seen after date |
| `recordroom issues show <id>` | Full issue detail + findings summary |
| `recordroom issues conversations <id>` | Conversations linked to the issue |
| `recordroom issues create --title "..."` | Manually create an issue |
| `recordroom issues update <id> --severity critical` | Update issue fields |
| `recordroom issues confirm <id> --expected-behavior "..."` | Confirm (for_review -> open) |
| `recordroom issues ignore <id>` | Mark as noise |
| `recordroom issues resolve <id>` | Mark as resolved |
| `recordroom issues reopen <id>` | Reopen a resolved issue |
| `recordroom findings confirm <id>` | Confirm a finding |
| `recordroom findings accept <id>` | Accept a finding |
| `recordroom findings edit-accept <id> --description "..."` | Accept with edited text |
| `recordroom findings dismiss <id>` | Dismiss a finding |

All commands support `--json` for structured output.

## Key Concepts

- **Occurrence count** — How many conversations this issue appeared in. Higher count = more impactful.
- **Expected behavior** — Defined when confirming an issue. This becomes the grading criteria for regression tests (tasks inherit it).
- **Task categories** — Issues are auto-categorized into capability dimensions (e.g., "Refund flow", "Tool grounding"). Tasks inherit the category from their parent issue.
- **Auto-regression** — When a task linked to a resolved issue fails, the issue automatically moves to `regressed` state.
- **Finding triage** — Individual findings can be confirmed, accepted, dismissed, or accepted with an edited description. Accepted findings become positive examples for the skill; dismissed ones become negative examples.
