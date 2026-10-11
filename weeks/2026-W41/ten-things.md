# Ten things to try: week of Oct 5–11, 2026

Compiled Oct 11, 2026 from the week's recurring applied-AI themes plus web research. Every URL below was fetched and logged before publication. arXiv abstracts were read on arXiv; full papers were **not** read, and no code was run. To use a build brief, paste it into Codex or Claude Code and fill in the `<placeholders>`.
Status: ✅ primary source fetched and claim matches · 🟡 source exists, claim only partly checked · ⚠️ unverified · 🧪 = deliberately unconventional idea

## 1. "TestJack-lite": a Stop hook that audits requirements before Claude can say done. `[B5][B1]` practical
- **Idea:** before Claude Code ends a turn, an auditor generates a few tests aimed at *the task's stated requirements* (not the existing tests), runs them, and sends Claude back if any fail.
- **Why now:**
  - [TestJack, arXiv 2610.10619](https://arxiv.org/abs/2610.10619) (Oct 7) ✅: "about 34.4% of the model trials currently judged correct violate the task requirements, lowering the overall resolution rate from 50.6% to 33.2%."
  - [Claude Code hooks](https://code.claude.com/docs/en/hooks) ✅: exit code 2 on Stop "sends Claude back to keep working". `stop_hook_active` prevents loops, and there's an 8-continuation cap.
- **Build brief (Claude Code):**
```
GOAL: Add a requirement-audit gate to <repo> so Claude Code cannot end a task while stated requirements are unmet.
CONTEXT: TestJack (arXiv 2610.10619) shows fixed tests miss ~1/3 of requirement violations. Claude Code Stop hooks: exit 2 = keep working; check stop_hook_active.
CONSTRAINTS: Time box 2h. No network calls in the hook. Hook must finish < 60s. Audit tests live in tests/audit/ and are never deleted by the agent. Show the diff before enabling.
ACCEPTANCE CRITERIA:
 1. .claude/settings.json registers a Stop hook script; `claude --debug` shows it firing.
 2. On a seeded task whose README requirement is unmet but existing tests pass, the hook exits 2 and Claude resumes (NEGATIVE TEST: must block).
 3. On a correct solution, the hook exits 0 within 60s.
 4. With stop_hook_active=true and the same failure twice, the hook stops blocking and writes AUDIT-UNRESOLVED.md (no infinite loop).
 5. Every audit test must FAIL on the pre-change commit (git stash check), the substitute for TestJack's "passes ground truth" filter.
 6. A log line per run in .claude/audit.log (actually-ran evidence).
FIRST STEPS: (1) Add a CLAUDE.md rule: write task requirements to .claude/requirements.md at task start. (2) Write an auditor subagent (model: sonnet) that turns each requirement into one pytest/jest test in tests/audit/. (3) Write hooks/stop_audit.sh to run tests/audit + the suite and exit 2 with failures on stderr. (4) Seed the negative test case. (5) Run criteria 1–6.
REPORT BACK: diff, test output for criteria 2–5, and anything not done.
```

## 2. "ECLoop-lite": no edits until the evidence has been read. `[B1][B5]` practical
- **Idea:** a PreToolUse hook denies `Edit/Write` on a file until the session has `Read` that file and the related test, and `git commit` until the tests have run.
- **Why now:**
  - [ECLoop, arXiv 2607.28815](https://arxiv.org/abs/2607.28815) ✅: blocking "premature commitment" raised SWE-bench Verified Pass@1 by "4.8-11.8 percentage points without model retraining or scaffold changes", with lower token use.
  - [Hooks docs](https://code.claude.com/docs/en/hooks) ✅: PreToolUse can return `permissionDecision: "deny"`.
- **Build brief (Claude Code or Codex):**
```
GOAL: Enforce "observe before modify" in <repo>: deny edits to files not yet read in this session, and deny commits before tests ran.
CONTEXT: ECLoop (arXiv 2607.28815). Claude Code PreToolUse hook with permissionDecision "deny" + reason.
CONSTRAINTS: Pure local script (bash or python stdlib). Session state in .claude/evidence/<session_id>.json. Must never block Read/Grep/Glob. Escape hatch: env EVIDENCE_GATE=off.
ACCEPTANCE CRITERIA:
 1. Edit to an unread file → denied with a reason naming the file to read (NEGATIVE TEST).
 2. Read file → Edit allowed.
 3. git commit before any test command → denied; after tests → allowed.
 4. Gate adds < 200 ms per tool call (measure 50 calls).
 5. One-week log summary script: # denials, top denied files.
FIRST STEPS: (1) Write a PostToolUse hook that records Read paths and test commands. (2) Write a PreToolUse hook that checks the state and denies. (3) Write fixtures for criteria 1–3. (4) Benchmark latency.
REPORT BACK: diff, fixture results, latency numbers.
```

## 3. Admit skills by what the runtime will execute, not just what the scanner can read. `[B2][B1]` practical · combo verification × skills
- **Idea:** scan every skill and plugin you have written or installed with an open-source scanner, then add one check scanners tend to miss: compare the readable source with every executable artifact the runtime could load (for Python, a bundled bytecode cache). Make "passes both" a precondition for installing a new skill.
- **Why now:**
  - [PyCache Trap, arXiv 2610.10612](https://arxiv.org/abs/2610.10612) ✅ (abstract): "Across 100 skills and seven scanners, PyCache Trap achieves 94-100% attack success, with no semantic recognition of the cache-resident behavior." Their proposed execution-aware validation "detects all 100 evaluated source-present cache substitutions".
  - [Skill Constellations, arXiv 2610.11169](https://arxiv.org/abs/2610.11169) ✅ (abstract): skills are copied between repositories with "no registry, versions or provenance"; "Skill copies almost never change with their source", so a fix upstream rarely reaches copies. Auditing the model's top 100 source repos "prevents 14.9% of later adoptions of high-risk skills, against 0.5% for the 100 most starred".
  - [NVIDIA/SkillSpector](https://github.com/NVIDIA/SkillSpector) ✅: README and install commands read (`uv tool install git+https://github.com/NVIDIA/skillspector.git`). ⚠️ Its own figure that 26.1% of skills in a 31,132-skill subset contain vulnerabilities and 5.2% show likely malicious intent is their dataset, not independently checked.
  - [Snyk Agent Scan](https://github.com/invariantlabs-ai/mcp-scan) 🟡: README read; scans local agents, MCP servers and skills for prompt injection, tool poisoning and malware payloads. Not run.
- **Build brief (Codex):**
```
GOAL: Produce a security report for all my skills/plugins and a pre-install gate that also checks executable artifacts.
CONTEXT: SkillSpector (https://github.com/NVIDIA/SkillSpector; offline mode `skillspector scan <path> --no-llm --format json --output report.json`). PyCache Trap (arXiv 2610.10612): a skill can ship benign .py source plus a substituted __pycache__ .pyc that Python loads instead.
CONSTRAINTS: Run in a container first. --no-llm for the first pass (nothing leaves the machine). Do not auto-fix; propose diffs. Never execute skill code outside the container.
ACCEPTANCE CRITERIA:
 1. reports/skills-<date>.json covers every skill directory under ~/.claude, ~/.codex and my repos (list scanned and skipped paths with reasons).
 2. Summary table: skill -> findings by severity.
 3. artifact_check.py flags any skill directory that contains .pyc, .so, .node, binary or minified files without a matching readable source, and any .pyc whose recompiled source differs from the shipped bytecode.
 4. NEGATIVE TEST: a planted test skill with an exfiltration instruction is flagged by the scanner; a second planted skill with benign source plus a mismatched .pyc is flagged by artifact_check.py.
 5. scripts/install-skill.sh refuses to install when either check fails (exit non-zero, shown by running it on both planted skills).
FIRST STEPS: (1) Install in a container. (2) Enumerate skill dirs. (3) Batch scan. (4) Write artifact_check.py. (5) Plant both bad skills and run criteria 4 and 5. (6) Write the gate script.
REPORT BACK: summary table, results for both planted skills, skipped paths, anything you could not check.
```

## 4. Tool-poisoning and rule-file drill with a scoped-credential containment test. `[B2]` practical · combo injection × credentials
- **Idea:** build a local test MCP server and a test rule file that carry harmless canary instructions (write `CANARY.txt`; swap a dependency for a canary package name). Measure which of your coding agents obey them. Then rerun the same attacks with the agent holding only a narrowly scoped token and see which consequences stop even when the agent is fooled.
- **Why now:**
  - [arXiv 2603.21642](https://arxiv.org/abs/2603.21642) ✅ compared seven MCP clients including Claude Code, Cursor and Gemini CLI; some, "such as Cursor, exhibit high susceptibility to cross-tool poisoning, hidden parameter exploitation, and unauthorized tool invocation."
  - [arXiv 2603.22489](https://arxiv.org/abs/2603.22489) 🟡 names tool poisoning "the most prevalent and impactful client-side vulnerability" (abstract only).
  - [Package hallucination via rule files, arXiv 2610.09264](https://arxiv.org/abs/2610.09264) ✅ (abstract): an attacker injects prompts into benign rule files (CLAUDE.md-style, `.cursorrules`) to make agents replace legitimate dependencies with attacker-controlled packages.
  - [Compromise Is Not Consequence, arXiv 2610.05840](https://arxiv.org/abs/2610.05840) ✅ (abstract): broad bearer tokens allowed harmful execution in 8.9% to 37.8% of attacked decisions; "all three scoped conditions record zero". The authors state this limits consequences and does not prevent injection, and that the test used researcher-supplied task authority.
- **Build brief (Claude Code):**
```
GOAL: Measure whether my coding agents obey malicious instructions hidden in MCP tool metadata or rule files, and whether scoped credentials contain the damage.
CONTEXT: arXiv 2603.21642 (client comparison), arXiv 2610.09264 (rule-file dependency swap), arXiv 2610.05840 (scoped vs broad credentials, paired replay).
CONSTRAINTS: Local only. Test server exposes no real data or credentials. Payloads are harmless canaries. Never point at production servers or real package registries (use a local fake registry). Use throwaway tokens.
ACCEPTANCE CRITERIA:
 1. test-mcp/ server with 3 benign tools, 1 poisoned-description variant, 1 hidden-parameter variant; plus a rule file variant that asks to swap dependency X for canary-X from a local registry.
 2. Harness runs N=10 identical tasks per client (Claude Code, Codex) per variant and records attack success rate (CANARY.txt written, or canary-X installed).
 3. Static linter flags imperative/instruction-like text in tool descriptions; catches both poisoned variants and none of the 3 benign descriptions (NEGATIVE TEST).
 4. Containment replay: same attacked actions submitted once with a broad throwaway token and once with a token scoped to the task's resources; report harmful-execution counts for each.
 5. Results table plus a recommendation of which clients may connect to third-party servers.
FIRST STEPS: (1) Scaffold server and local fake registry. (2) Write variants. (3) Write headless harness loop. (4) Write linter. (5) Add scoped-token replay. (6) Write the report.
REPORT BACK: attack success table, linter precision, containment counts, raw log path, what could not be run.
```

## 5. 🧪 Silent-failure canaries: plant known failures and grade whether your agents *report* them. `[B5][B4]` unconventional · combo silent failure × event-triggered agents
- **Idea:** invert the eval. Instead of checking that agents succeed, seed every recurring run (research agents, digest jobs, chat bots) with **known-bad inputs**: a dead URL, an empty search, a tool that returns 200 with no body. Score whether the output admits the failure. Treat any run that hides a canary as a failed run.
- **Why now:**
  - [WebArena-Lite audit, arXiv 2610.01491](https://arxiv.org/abs/2610.01491) ✅: human review recovered "5.45 to 8.49 percentage points of success missed by the evaluator", and failures included "premature answers".
  - [WebFovea, arXiv 2610.03036](https://arxiv.org/abs/2610.03036) / [repo](https://github.com/jianganghan/WebFovea) ✅: on live sites, actions "failed silently". They frame four stages that must each go right (parse → take effect → report back → show the model what it needs). The harness alone moved their score from 31.0 to 57.0.
  - A developer-forum report this week described MCP Events webhooks returning HTTP 200 while the triggered task never ran ([thread](https://community.openai.com/t/mcp-events-verified-subscription-and-webhook-http-200-but-dot-task-never-runs/1404198)) 🟡.
- **Build brief (Claude Code):**
```
GOAL: A canary harness that injects known failures into my recurring agent runs and scores failure-reporting honesty.
CONTEXT: My systems: <system 1>, <system 2>, <system 3> (e.g. a research agent, a digest job, a chat bot). WebFovea's 4-stage model (parse, effect, report, observation).
CONSTRAINTS: Canaries are clearly tagged (CANARY-<id>) and never sent to real recipients. ≤1 canary per 10 real tasks. No changes to production prompts in v1, observe only.
ACCEPTANCE CRITERIA:
 1. canaries.yaml defines ≥6 types: dead URL, empty search result, tool 200-with-empty-body, timeout, partial page, contradictory source.
 2. Harness injects them via a proxy/mocked tool layer; each run logs canary id → stage where it was introduced.
 3. Grader marks each run HONEST (output names the failure) / SILENT (output omits it) / FABRICATED (output invents content). Verified by hand on 10 runs (agreement ≥ 9/10).
 4. Weekly report: silent-failure rate per system and per stage.
 5. NEGATIVE TEST: a deliberately "cheating" stub agent that ignores errors scores SILENT on 100% of canaries.
FIRST STEPS: (1) Pick one system. (2) Add a mock tool layer. (3) Write 6 canaries. (4) Write the grader prompt + regex checks. (5) Run 20 tasks.
REPORT BACK: rates table, 3 example SILENT outputs, hand-check agreement.
```

## 6. Practice-generated memory for personal bots, kept in a notes vault. `[B3]` practical
- **Idea:** let an "explorer" bot invent realistic tasks against your own tools (chat-bot commands, scheduled assistant routines, MCP tools). A "solver" attempts them. A lesson ("heuristic") is saved to a memory note (e.g. in Obsidian) only after it **repeatedly** helps the solver succeed.
- **Why now:** [DAEDALUS, arXiv 2610.08048](https://arxiv.org/abs/2610.08048) ✅ builds agent memory from self-generated practice "without existing tasks or oracle verifiers". A heuristic is accepted only after repeated success. Reported: mean success "up to 15.9 points" higher and pass^5 up to 2.2x over a no-memory baseline. [Code](https://github.com/illuin-tech/daedalus) 🟡 (repo exists; 7 stars; pushed Oct 7).
- **Build brief (Codex):**
```
GOAL: Bootstrap a verified heuristics file for <bot> from self-generated practice tasks.
CONTEXT: DAEDALUS (arXiv 2610.08048): explorer generates solvable tasks; heuristic accepted only after repeated solver success with it in context.
CONSTRAINTS: Sandbox copies of tools only (no real messages sent, no real calendar/email writes). Budget cap $5 per run. Memory written to <vault>/agents/<bot>/heuristics.md with provenance per line.
ACCEPTANCE CRITERIA:
 1. ≥30 generated tasks, each with an automatic success check.
 2. A heuristic is accepted only if success rate with it ≥ 3/3 on fresh variants AND it doesn't lower success on 5 control tasks.
 3. Before/after success rate on a held-out set of 15 tasks is reported (no claimed improvement without this).
 4. Every accepted line cites the task ids that justified it.
FIRST STEPS: (1) Wrap the bot's tools in a sandbox. (2) Write explorer and solver prompts. (3) Write the acceptance loop. (4) Run the held-out eval.
REPORT BACK: held-out before/after, accepted vs rejected heuristics, cost.
```

## 7. 🧪 Replay your own Claude Code history as a test world before changing anything. `[B4][B5]` unconventional · combo history × verification
- **Idea:** turn your retained session transcripts into a "worldbook" that simulates how your tools and repos responded. Then test a new CLAUDE.md, skill or routing setup against the *simulation* before running it for real: offline regression testing for agent configs.
- **Why now:**
  - [Trace2Env, arXiv 2610.06100](https://arxiv.org/abs/2610.06100) ✅ reconstructs environments from historical interaction traces. Across "nine environments, Trace2Env improves both next-observation fidelity and long-horizon interaction consistency".
  - Transcripts only exist if you keep them: Claude Code's `cleanupPeriodDays` defaults to 30 ([docs](https://code.claude.com/docs/en/settings-reference) ✅).
- **Build brief (Claude Code):**
```
GOAL: Build a minimal trace-based simulator from my Claude Code transcripts and use it to compare two CLAUDE.md variants.
CONTEXT: Trace2Env (arXiv 2610.06100). Transcripts in ~/.claude/projects/**.jsonl (ensure cleanupPeriodDays is raised first).
CONSTRAINTS: Read-only on transcripts; redact secrets/tokens before any LLM sees them (regex + allowlist). Simulator answers only from worldbook evidence; when it lacks evidence it must return UNKNOWN, never invent.
ACCEPTANCE CRITERIA:
 1. worldbook/ built from ≥20 sessions of one repo (schemas of tool outputs + grounded examples).
 2. Fidelity check: on 30 held-out real steps, simulator next-observation judged consistent ≥ 70% (hand-check 10).
 3. UNKNOWN rate reported. A simulator that never says UNKNOWN fails (NEGATIVE TEST: query a tool never seen in traces).
 4. A/B report: two CLAUDE.md variants on 10 replayed tasks, with the caveat that results need confirming on real runs.
FIRST STEPS: (1) Write a transcript parser + redactor. (2) Write the worldbook builder. (3) Write the simulator agent. (4) Run the fidelity check. (5) Run the A/B.
REPORT BACK: fidelity numbers, UNKNOWN rate, A/B table, redaction test results.
```

## 8. A show-and-ask voice science companion for children on Gemini 3.8 Live, with a child-safety eval set. `[B7][B5]` practical
- **Idea:** build (or upgrade) a voice companion on the new live models. A child points the camera at something and asks about it. The app answers by voice while looking things up in the background (async tools). A 30-question eval set, scored on factual support, completeness, applicability and calibration, gates each release.
- **Why now:**
  - [Google blog, Sep 15, 2026](https://blog.google/innovation-and-ai/technology/developers-tools/build-real-time-voice-applications-gemini-audio/) ✅: Gemini 3.8 Live and 3.8 Live Extended Thinking, with "Asynchronous function calling", "Visual context", 97+ languages, priced "$0.005/min for audio input and $0.018/min for audio output".
  - Google publishes a [gemini-live-api-dev skill](https://github.com/google-gemini/gemini-skills) ✅ (SKILL.md read; covers migrating "legacy Live models (2.0/2.5/3.1) to Gemini 3.8 Live").
  - [Live API best practices](https://ai.google.dev/gemini-api/docs/live-api/best-practices) 🟡 (page reachable; contents via search summary).
  - Offline/local alternative: [huggingface/speech-to-speech](https://github.com/huggingface/speech-to-speech) 🟡.
- **Build brief (Claude Code, with the Gemini skill installed):**
```
GOAL: Build a children's science companion on Gemini 3.8 Live with camera "show me" mode and background lookups, gated by a child-safety + accuracy eval.
CONTEXT: Google blog (Sep 15, 2026); google-gemini/gemini-skills → skills/gemini-live-api-dev; app repo <path>.
CONSTRAINTS: Ephemeral tokens client-side (no API key in the app). No audio/video stored by default; if stored for evals, local only and deletable. Age-appropriate system instructions; refusal/redirect policy written down first. Cost meter shown in dev mode.
ACCEPTANCE CRITERIA:
 1. 30-item eval set (20 science questions with reference answers, 5 "show me" image questions, 5 unsafe/off-limits prompts), scored on factual support, completeness, applicability, and calibration ("I'm not sure" when appropriate).
 2. ≥ 90% safe handling on the 5 unsafe prompts (NEGATIVE TESTS must be refused/redirected).
 3. Async tool call verified: audio keeps streaming while a lookup runs (log timestamps).
 4. Measured cost per 10-minute session reported against the published per-minute rates.
 5. Old vs new model comparison on the same 30 items.
FIRST STEPS: (1) Install the gemini-live-api-dev skill. (2) Write the eval set before touching code. (3) Write or migrate the session code. (4) Add a camera frame stream. (5) Run the evals.
REPORT BACK: eval table old vs new, cost, failures with transcripts.
```

## 9. 🧪 Worm rehearsal: plant fake credentials and see what a malicious install script could reach on your machine. `[B2]` unconventional · combo supply-chain incident × agent config
- **Idea:** run a harmless "stealer" that behaves like the worms in this week's incidents (reads home-directory secrets, editor and MCP config files, shell history) inside a container or throwaway user account that mirrors your real layout, but holds only fake honeytoken credentials. Count which fake secrets it reaches, then tighten until the count drops, and verify the stolen-token alarm fires.
- **Why now:**
  - [tensorlake@0.5.144 write-up](https://pkgwarden.com/incidents/tensorlake-npm-october-2026/) 🟡 (third-party): published by the project's own release workflow "with a valid provenance attestation", with a `preinstall` hook; targets include "the Claude, Cursor, Kiro, Windsurf and Zed configuration and MCP files" and a `gh-token-monitor` that, per the write-up, wipes the home directory if a stolen GitHub token is revoked. The write-up says its two sources disagree on exactly when that trap arms. Do not revoke a real token on an infected machine without reading current guidance.
  - [GhostAction, Socket](https://socket.dev/blog/ghostaction-cloud-credentials) ✅ and [StepSecurity](https://www.stepsecurity.io/blog/ghostaction-returns) ✅ (both fetched, both name `security-audit.yml`): credentials in git history are exposed even after you delete them from the file.
  - [CISA KEV catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) ✅ fetched as the exploited-in-the-wild baseline: this week's additions were not developer-tooling issues, so the risk here is the supply chain, not a patchable CVE.
- **Build brief (Codex):**
```
GOAL: Measure the blast radius of a malicious install script on my dev setup using fake credentials only, then reduce it.
CONTEXT: Incidents: tensorlake@0.5.144 npm worm (reads ~/.ssh, .npmrc, .env, cloud credential files, editor/MCP config), GhostAction (reads credentials from git history).
CONSTRAINTS: Container or throwaway OS user only. Never run it on my real home directory. No real tokens anywhere. The simulated stealer only READS and writes a local report; no network, no deletion, no republishing. Delete the harness when done.
ACCEPTANCE CRITERIA:
 1. honeypot-home/ mirrors my real layout (paths only) with ≥12 fake secrets, each with a unique ID: ssh key, .npmrc token, .env files, gh hosts.yml, cloud credential files, MCP config files for each agent I use, shell history lines, a fake secret committed in git history.
 2. simulate_stealer.py (read-only) lists which fake secrets it reached, by ID, with the path it used.
 3. Baseline reach count recorded. After each hardening step (e.g. disabling package install scripts, moving tokens out of plain files, narrowing token scopes) the count is re-measured.
 4. NEGATIVE TEST: a secret placed outside any path the stealer searches must NOT be reported (proves the report is not just listing everything).
 5. Alarm test: reading any honeytoken triggers a log line or notification within 60s.
 6. Final table: step -> reach count -> effort. Hardening steps are proposed as diffs, not applied to the real machine.
FIRST STEPS: (1) Build honeypot-home. (2) Write the read-only simulator. (3) Baseline. (4) Try each hardening step in the container. (5) Add the alarm. (6) Write the table.
REPORT BACK: baseline and final reach counts, the negative test result, which steps cost the most effort, anything not tested.
```

## 10. 🧪 Grade a robot foundation model like you grade agents (simulation only). `[B6][B5]` unconventional
- **Idea:** you don't need a robot to build intuition for physical AI. Run an open VLA's published simulation benchmark and apply a multi-dimension eval rubric (factual support, completeness, actual delivery, calibration) to its episodes: did it *actually* complete the task, and does its own "success" signal agree with what the video shows? Physical AI is where "looks done but isn't" has physical consequences.
- **Why now:** [sii-research/tau-0-vla](https://github.com/sii-research/tau-0-vla) ✅ README read: a hierarchical robot foundation model, Apache-2.0 code and weights. "[2026.09.20] We release LIBERO post-training and simulation evaluation" and "[2026.09.21] … high-level proposal and world model". ⚠️ Hardware needs not checked. Probably needs a GPU.
- **Build brief (Codex; GPU machine required):**
```
GOAL: Reproduce τ₀-VLA's LIBERO simulation eval on a small task subset and audit success labels against episode videos.
CONTEXT: https://github.com/sii-research/tau-0-vla (LIBERO guide in repo).
CONSTRAINTS: Cloud GPU budget cap $<X>; stop if setup exceeds 2h. Simulation only. Record exact commit + checkpoint hashes.
ACCEPTANCE CRITERIA:
 1. Setup reproducible from a single script (fresh machine test).
 2. ≥ 50 episodes across ≥ 5 LIBERO tasks with videos saved.
 3. Hand-audit 20 episodes: agreement between simulator success flag and visual completion reported (disagreements listed).
 4. Results compared with the repo's published LIBERO numbers (match/mismatch noted, not explained away).
 5. One-page write-up scored with the rubric above.
FIRST STEPS: (1) Read the LIBERO guide. (2) Provision GPU. (3) Run setup.sh. (4) Run 5 tasks × 10 episodes. (5) Audit.
REPORT BACK: success rates, audit agreement, cost, what broke during setup.
```

---

## Security notes this week
- A fake `security-audit.yml` workflow pushed with stolen maintainer tokens (GhostAction) sends CI secrets and credentials found in git history to a hardcoded address; two independent write-ups agree on the details ([Socket](https://socket.dev/blog/ghostaction-cloud-credentials) ✅, [StepSecurity](https://www.stepsecurity.io/blog/ghostaction-returns) ✅).
- A compromised npm release (tensorlake 0.5.144) carried a credential-stealing worm that targets editor and MCP configuration files, per one [third-party write-up](https://pkgwarden.com/incidents/tensorlake-npm-october-2026/) 🟡.
- Claude Code published an [advisory](https://github.com/anthropics/claude-code/security/advisories/GHSA-5j29-h97v-84ch) ✅ for a symlink race that could redirect a write outside the project; the advisory says auto-updating installs already have the fix.
- Three early-October advisories were reported for the MCP TypeScript and Python SDKs, with fixed releases, and the write-up says `npm audit` showed nothing at the time ([write-up](https://al-ice.ai/posts/2026/10/mcp-sdk-october-advisories-cross-origin-redirect-task-session-scanner-gap/) 🟡). Check lockfiles by version.

## Considered and not included
- **Policy-to-tool-call gates** ([NOMOS, arXiv 2610.11030](https://arxiv.org/abs/2610.11030) ✅ abstract): compiles written policies into tool-call gates and reports violations falling from 66.3% to 2.6% on an airline benchmark. A good follow-on to #4, but research-grade to build.
- **Newly published MCP scanner repos** ([mcp-doctor](https://github.com/xka0085-byte/mcp-doctor), [agent-audit-kit](https://github.com/sattyamjjain/agent-audit-kit)): days old with under 15 stars, unvetted. Prefer the established scanners in #3.
