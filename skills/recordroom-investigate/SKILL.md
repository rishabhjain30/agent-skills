---
name: recordroom-investigate
description: Investigate recent traces to find errors, performance issues, or behavior patterns. Use when the user asks to analyze trace quality, debug agent runs, find error patterns, or summarize recent activity.
---

# RecordRoom Investigate

## Workflow

1. Sync latest traces: `npx recordroom sync --limit 100`
2. Check status: `npx recordroom status`
3. Direct the user to open RecordRoom web UI for full trace browsing.
4. If the user describes a specific issue, help narrow down:
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

## Output format

- Start with a concise summary of findings
- Call out the most actionable issues first
- Include specific trace IDs or timestamps for follow-up
- Suggest next steps (fix code, adjust model, add guardrails)

## Limitations

- The CLI currently syncs traces but doesn't have a local query command.
- For detailed trace inspection, direct the user to the RecordRoom web UI.
- Future versions will add `recordroom traces list` and `recordroom traces search`.
