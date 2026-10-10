# Library highlights: 2026-W41

Source-checked notes added to the library this week (✅ primary source checked · 🟡 partly checked).

## Coding agents & tooling

- **Claude Code keeps session transcripts for 30 days by default** ✅: The cleanupPeriodDays setting controls retention and the cleanup sweep runs without any notice. Raise it in settings.json if transcripts are raw material for finding repeated work worth automating. ([source](https://code.claude.com/docs/en/settings-reference))
- **Advisor model in Claude Code: consultation is a request, not a gate** ✅: A cheaper main model can consult a stronger advisor model via /advisor or advisorModel. The docs state there is no setting to force or cap advisor calls, so 'checkpoint' instructions are best-effort. Enforce must-pass checks with hooks or tests instead. ([source](https://code.claude.com/docs/en/advisor))
- **Choosing the subagent model in Claude Code** ✅: Use the CLAUDE_CODE_SUBAGENT_MODEL environment variable or a model field in the subagent definition. A widely shared '--subagents' CLI flag does not appear in the CLI reference. ([source](https://code.claude.com/docs/en/model-config))
- **oh-my-mermaid: architecture diagrams from a codebase** 🟡: Generates Mermaid architecture diagrams from a repository with a coding agent. Repo exists; README not yet reviewed. ([source](https://github.com/oh-my-mermaid/oh-my-mermaid))

## Evals & verification

- **Premature commitment: agents editing before they have evidence** ✅: ECLoop compiles per-task conditions describing what the agent must observe before each kind of edit or patch submission, and enforces them in an execution layer between agent and repository. ([source](https://arxiv.org/abs/2607.28815))
- **Auditing automatic evaluators for web agents** ✅: A human audit of all 165 WebArena-Lite tasks under six evaluation conditions corrects false negatives from automatic evaluators. Outcome-only scoring hides trajectory problems, so audit your evaluator, not just your agent. ([source](https://arxiv.org/abs/2610.01491))

## Security

- **SkillSpector: security scanner for agent skills** 🟡: Open-source scanner that looks for prompt-injection and malicious patterns in agent skill files. Worth running before installing community skills. ([source](https://github.com/NVIDIA/SkillSpector))
- **Prompt injection across skills, tools and protocol ecosystems** 🟡: Systematic analysis of how agentic coding assistants can be attacked through skills, tool descriptions and MCP servers. Useful as a checklist when reviewing what an agent is allowed to load. ([source](https://arxiv.org/abs/2601.17548))

## Agent-first design

- **Event-triggered agents (MCP Events)** 🟡: Apps can push events that wake an agent task. Sensible rollout: start with read-only actions, deduplicate on the source event id rather than the payload, and keep a stop-only kill switch. Treat every event payload as untrusted input. ([source](https://developers.openai.com/plugins/build/mcp-events))
