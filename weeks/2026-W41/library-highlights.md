# Library highlights: 2026-W41

Source-checked notes added to the library this week (✅ primary source checked · 🟡 partly checked).

## Security

- **SkillSpector: security scanner for agent skills** 🟡: Open-source scanner that looks for prompt-injection and malicious patterns in agent skill files. Worth running before installing community skills. ([source](https://github.com/NVIDIA/SkillSpector))
- **Prompt injection across skills, tools and protocol ecosystems** 🟡: Systematic analysis of how agentic coding assistants can be attacked through skills, tool descriptions and MCP servers. Useful as a checklist when reviewing what an agent is allowed to load. ([source](https://arxiv.org/abs/2601.17548))
- **Microsoft MXC: policy-driven sandbox for agent shell commands** ✅: Open-source execution containers with reviewable policies for file access (read-only, read-write, denied paths) and network (proxy, outbound controls); backends include Bubblewrap and Seatbelt. A concrete template for a deny-by-default sandbox policy for any coding agent. Docs and README read; not run. ([source](https://github.com/microsoft/mxc))
- **PyCache Trap: skill scanners inspect source, runtimes execute bytecode** ✅: Across 100 skills and seven scanners, swapping a bundled Python bytecode cache for a malicious one evaded detection 94-100% of the time. Proposed fix is to validate the executable artifacts a runtime can select, not only the visible source. Abstract read; full paper not read. ([source](https://arxiv.org/abs/2610.10612))
- **Skill Constellations: agent skills are a supply chain with no registry or versions** ✅: A dated copy network of 2.19 million skill adoptions on GitHub shows a few source repositories account for most copies, stars do not identify them, and fixes at the source rarely reach copies. Auditing the model's top 100 sources would prevent 14.9% of later high-risk adoptions versus 0.5% for the 100 most starred. Abstract read. ([source](https://arxiv.org/abs/2610.11169))
- **Scoped credentials contain a hijacked agent even when injection succeeds** ✅: In a replay test, broad bearer tokens allowed harmful execution in 8.9% to 37.8% of attacked decisions depending on model, while all three scoped-authorization conditions recorded zero. This contains consequences; it does not prevent the injection. Abstract read; authors list limits. ([source](https://arxiv.org/abs/2610.05840))
- **MCP SDK advisories: cross-origin redirects and OAuth credential routing** 🟡: Three early-October advisories against the TypeScript and Python MCP SDKs, fixed in newer releases, with no CVE at the time and zero findings from npm audit or OSV. Check lockfiles directly. Secondary write-up; primary advisories not individually opened. ([source](https://al-ice.ai/posts/2026/10/mcp-sdk-october-advisories-cross-origin-redirect-task-session-scanner-gap/))
- **GhostAction: fake security-audit workflows steal CI secrets and git-history credentials** ✅: Attackers with stolen maintainer tokens pushed a workflow named security-audit.yml that sends Actions secrets and credentials found in the working tree or git history to a hardcoded IP. Deleting a leaked key from a file does not help because the workflow reads history. Two independent write-ups agree on the workflow name and the IP. ([source](https://socket.dev/blog/ghostaction-cloud-credentials))
- **tensorlake npm 0.5.144: provenance-valid publish with a credential-stealing worm** ✅: A compromised maintainer account published a release with a valid provenance attestation and a preinstall hook that harvests developer secrets and AI-tool and MCP config files, then republishes itself to packages the victim can publish. A valid signature proves where a package was built, not that the build was safe. Third-party incident write-up; details not independently reproduced. ([source](https://pkgwarden.com/incidents/tensorlake-npm-october-2026/))

## Coding agents & tooling

- **Claude Code keeps session transcripts for 30 days by default** ✅: The cleanupPeriodDays setting controls retention and the cleanup sweep runs without any notice. Raise it in settings.json if transcripts are raw material for finding repeated work worth automating. ([source](https://code.claude.com/docs/en/settings-reference))
- **Advisor model in Claude Code: consultation is a request, not a gate** ✅: A cheaper main model can consult a stronger advisor model via /advisor or advisorModel. The docs state there is no setting to force or cap advisor calls, so 'checkpoint' instructions are best-effort. Enforce must-pass checks with hooks or tests instead. ([source](https://code.claude.com/docs/en/advisor))
- **Choosing the subagent model in Claude Code** ✅: Use the CLAUDE_CODE_SUBAGENT_MODEL environment variable or a model field in the subagent definition. A widely shared '--subagents' CLI flag does not appear in the CLI reference. ([source](https://code.claude.com/docs/en/model-config))
- **oh-my-mermaid: architecture diagrams from a codebase** 🟡: Generates Mermaid architecture diagrams from a repository with a coding agent. Repo exists; README not yet reviewed. ([source](https://github.com/oh-my-mermaid/oh-my-mermaid))
- **Claude Code prompt caching: subagent cache lifetime accepts only 5m or 1h** ✅: The docs say each cache-lifetime control takes 5m or 1h and ignores any other value, so a viral setting using a bare number does nothing. Check the exact setting against the docs before copying a tweet. ([source](https://code.claude.com/docs/en/prompt-caching))
- **Asana: append-only prompt history cut a browser agent's cost** 🟡: Asana reports 89% of input served from the prompt cache after caching the history and no longer editing earlier turns. Costs are Asana's own estimates; the vendor case study with the headline multiple was not readable. ([source](https://asana.com/inside-asana/cut-browsers-agent-cost))

## Evals & verification

- **Premature commitment: agents editing before they have evidence** ✅: ECLoop compiles per-task conditions describing what the agent must observe before each kind of edit or patch submission, and enforces them in an execution layer between agent and repository. ([source](https://arxiv.org/abs/2607.28815))
- **Auditing automatic evaluators for web agents** ✅: A human audit of all 165 WebArena-Lite tasks under six evaluation conditions corrects false negatives from automatic evaluators. Outcome-only scoring hides trajectory problems, so audit your evaluator, not just your agent. ([source](https://arxiv.org/abs/2610.01491))
- **Mulligan: collect robot training data starting from observed failures** 🟡: Starts each round of episodes at previously observed failures and untried states. The abstract reports gains of 10 to 34 points on real tasks. The same rule applies to any eval: sample test cases from where the system already failed. Abstract only. ([source](https://arxiv.org/abs/2610.05882))

## Agent-first design

- **Event-triggered agents (MCP Events)** 🟡: Apps can push events that wake an agent task. Sensible rollout: start with read-only actions, deduplicate on the source event id rather than the payload, and keep a stop-only kill switch. Treat every event payload as untrusted input. ([source](https://developers.openai.com/plugins/build/mcp-events))

## Voice, video, creativity, animation

- **Gemini 3.8 Live: real-time voice with async function calling and visual context** ✅: Google's announcement lists asynchronous function calling, visual context and 97+ languages, with published per-minute audio prices. Vendor claims; not tried. ([source](https://blog.google/innovation-and-ai/technology/developers-tools/build-real-time-voice-applications-gemini-audio/))

## Physical AI

- **tau-0-VLA: open hierarchical robot foundation model with LIBERO simulation eval** 🟡: Apache-2.0 code and weights, with a LIBERO post-training and simulation evaluation released in September. README read; hardware needs and results not checked. ([source](https://github.com/sii-research/tau-0-vla))
