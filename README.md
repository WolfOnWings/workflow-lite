# workflow-lite

A small workflow suggestion for Claude Code: the `research` skill plus the `research-analyst` subagent, packaged so you can lift them into your own setup. Ask Claude to add these to your tools.

## The pattern

Usually research is the first step when creating something, but LLMs doing informal research drift in predictable ways. This agent employs specific techniques in an attempt to avoid common failure modes. The skill here is just a thin orchestration prompt that dispatches a subagent for the heavy lift; the subagent's intermediate tool calls — searches, fetches, dead ends — never enter the orchestrator's context window, so you get the work without the noise, and it's easier to call when a skill is used. More importantly, the skill prompt acts as a defense membrane: it's where you encode rules the orchestrator will ignore under context pressure. Without that membrane, an orchestrator usually poisons the agent call — paraphrased question, leaked hypothesis, grand-ceremony preamble bloat, etc. Together: the worker keeps the orchestrator's context clean; the skill keeps the worker's input clean. Beyond the specific orch-worker framework, the **skill-wraps-agent shape** is itself worth lifting.

## Layout

```
.claude/
  agents/research-analyst.md      # the worker
  skills/research/SKILL.md  # the orchestrator wrapper
```

Invoke with `/research <topic>`. Brief lands at `.scratch/research/<topic-slug>-MM-DD.md`. These are just the file paths I use. You should modify these tools to fit your own workflow.

## License

MIT — see `LICENSE`.
