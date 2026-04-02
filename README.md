# RecordRoom Agent Skills

Monitor, test, and reproduce AI agent issues. Works with Claude Code, Cursor, Windsurf, Copilot, and 40+ coding agents.

## Install

```bash
npx skills add rishabhjain30/agent-skills
```

## Skills

| Skill | Description |
|-------|-------------|
| **recordroom-setup** | Install and authenticate the RecordRoom CLI |
| **recordroom-connect** | Connect RecordRoom to Langfuse or other trace sources |
| **recordroom-sync** | Sync traces from connected sources |
| **recordroom-investigate** | Analyze traces for errors, performance issues, and patterns |
| **recordroom-parser** | Create trace parsers to normalize raw trace data |
| **recordroom-api-config** | Set up API configurations for running evaluations |
| **recordroom-evals** | Fix failing evals, run regression checks, diagnose agent issues |
| **recordroom-for-claude** | General RecordRoom awareness and command index |

## Quick start

After installing, ask your coding assistant:

> Help me set up RecordRoom API testing to reproduce the issues found in our agent

The assistant will handle CLI setup, API configuration, and running evaluations.

## Claude Code plugin

```
/plugin marketplace add rishabhjain30/agent-skills
/plugin install recordroom@recordroom
```
