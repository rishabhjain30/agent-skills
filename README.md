# RecordRoom for Claude Code

A Claude Code plugin that helps Claude monitor AI agent conversations, triage quality issues, and run regression tests against your agent — using the [RecordRoom](https://recordroom.ai) platform.

## Install

In Claude Code:

```
/plugin marketplace add rishabhjain30/recordroom-plugin
/plugin install recordroom@recordroom
```

Then ask Claude:

> Set up RecordRoom for this project — connect Langfuse, sync traces, and show me the top issues.

Claude will route through the bundled skills to handle setup, connection, sync, investigation, and test runs.

## Skills

| Skill | Invoked when… |
|-------|---------------|
| **setup** | User asks to install, log in to, or configure the RecordRoom CLI |
| **connect** | User wants to connect a trace source (Langfuse, etc.) |
| **sync** | User mentions syncing traces or pulling latest agent runs |
| **investigate** | User asks to analyze recent traces for errors or patterns |
| **issues** | User asks to review, triage, confirm, or ignore detected issues |
| **tasks** | User asks to build regression tests, run benchmarks, or check pass rates |
| **parser** | User asks to write or fix a parser for a new trace format |
| **api-config** | User asks to set up how RecordRoom calls their agent's API |
| **evals** | User asks to fix failing eval tests or run regression checks |
| **for-claude** | General awareness — loaded so Claude knows what RecordRoom is |

Skills are invoked automatically by Claude based on intent — you don't need to call them by name.

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) v2.0+
- Node.js 18+ (for the `recordroom` CLI invoked by skills)
- A RecordRoom account at [recordroom.ai](https://recordroom.ai)

## Layout

```
.claude-plugin/
  marketplace.json             # marketplace manifest (single-plugin)
plugins/
  recordroom/
    .claude-plugin/
      plugin.json              # plugin manifest
    skills/
      setup/SKILL.md
      connect/SKILL.md
      sync/SKILL.md
      investigate/SKILL.md
      issues/SKILL.md
      tasks/SKILL.md
      parser/SKILL.md
      api-config/SKILL.md
      evals/SKILL.md
      for-claude/SKILL.md
```

## License

MIT
