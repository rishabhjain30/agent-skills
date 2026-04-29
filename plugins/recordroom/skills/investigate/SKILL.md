---
name: investigate
description: Investigate recent traces to find errors, performance issues, or behavior patterns. Use when the user asks to analyze trace quality, debug agent runs, find error patterns, or summarize recent activity.
---

# RecordRoom Investigate

## Workflow

1. Sync latest traces: `npx recordroom sync --limit 100`
2. Check status: `npx recordroom status`
3. Browse traces:
   - `npx recordroom traces list` — list synced traces with IDs and sizes
   - `npx recordroom traces show <trace-id>` — show full raw trace data
4. Check findings: `npx recordroom findings list --status pending`
5. Check issues: `npx recordroom issues list --state for_review`

If the user describes a specific issue, help narrow down:
- Time window (when did it start?)
- Error messages or patterns
- Specific models or agents involved
- User or session IDs

## Analysis checklist

When reviewing traces, look for:
- **Errors and exceptions** — grouped by message and frequency
- **Slow or failing stages** — timeouts, high-latency LLM calls
- **Token/cost spikes** — unusually expensive traces
- **Model mismatches** — wrong model being used
- **Missing outputs** — traces with input but no output (crashes/timeouts)

## From investigation to action

Findings from analysis can be promoted through the quality pipeline:

1. **Findings** — Individual issues detected in specific conversations
   - Triage: `recordroom findings confirm <id>`, `recordroom findings accept <id>`, `recordroom findings dismiss <id>`
   - Fix description: `recordroom findings edit-accept <id> --description "corrected text"`
   - Summary: `recordroom findings summary`
2. **Issues** — Patterns grouped from findings
   - Review: `recordroom issues list --state for_review`
   - Search: `recordroom issues list --search "refund" --severity critical`
   - Confirm: `recordroom issues confirm <id> --expected-behavior "..."`
   - Update: `recordroom issues update <id> --severity high`
3. **Tasks** — Regression tests created from confirmed issues (`recordroom tasks generate <issue-id>`)
4. **Benchmark** — Pass rate tracked over time (`recordroom tasks benchmark`)

## Output format

- Start with a concise summary of findings
- Call out the most actionable issues first
- Include specific trace IDs or timestamps for follow-up
- Suggest next steps (fix code, adjust model, add guardrails, create issues)
