# Adaptive Subagent Orchestration

A portable Agent Skill for coding assistants that helps them decide when to delegate, assign implementation work safely, adapt concurrency to actual runtime capacity, and integrate validated results. The skill is instruction-only: it does not provide an agent runtime, raise concurrency limits, or grant additional permissions.

## Available skill

### `adaptive-subagent-orchestration`

Located at [`adaptive-subagent-orchestration/SKILL.md`](adaptive-subagent-orchestration/SKILL.md). Use it for implementation planning, parallel work, multi-component features, large refactors, and agent coordination. It keeps simple, sequential tasks with the main agent; when delegation makes sense, it explicitly favors bounded coding assignments alongside research, testing, and review.

The skill includes:

- Capability discovery before spawning: actual runtime tools, concurrency limits, model and effort controls, isolation, and existing agents.
- Dependency-aware work packages, an adaptive ready queue, worker lifecycle tracking, and multiple waves for large workloads.
- Implementation-first task contracts with measurable acceptance criteria and clear write ownership.
- Conflict prevention, native worktree/workspace use when available, controlled integration, and bounded recovery.
- Explicit authorization boundaries for external actions, inherited credentials, and a default retry limit.
- Effort calibration from low for mechanical edits to higher levels only when complexity and evidence justify them.
- Discovery and reuse of project instructions and native agent definitions, with careful creation only when supported and useful.

## Supported platforms and capabilities

The skill format and runtime behavior are separate. A client may load the same `SKILL.md` while supporting a different set of worker tools. Platform documentation was checked on November 13, 2026; runtime versions, settings, and limits can change.

| Platform | Skill discovery / install | Delegation, worker configuration, and limits |
| --- | --- | --- |
| **Codex** | Project `.agents/skills/`; user `~/.agents/skills/`. | Codex app/CLI tools depend on the active runtime. In the **Agents API**, enable `agent.multi_agent.enabled`; `max_concurrent_subagents` defaults to **6** (excluding the coordinator). In the separate **Responses API multi-agent beta**, `multi_agent.enabled` opts in and `max_concurrent_subagents` defaults to **3**, recommended for most tasks. The API subagents share the session model/tools and execution environment; they do not get an isolated workspace automatically. Do not assume either API setting controls the local app/CLI. [Codex skills](https://developers.openai.com/codex/skills) · [Agents API](https://developers.openai.com/api/docs/guides/agents-api/multi-agent) · [Responses API](https://developers.openai.com/api/docs/guides/responses-multi-agent) |
| **Cursor** | Project `.agents/skills/` or `.cursor/skills/`; user `~/.agents/skills/` or `~/.cursor/skills/`. Also recognizes Claude/Codex skill directories. | Parallel subagents work in editor, CLI, and Cloud Agents. Custom agents can choose models and model-specific effort parameters. Subagents share the parent checkout by default; isolated project copies are available. No universal numeric concurrency ceiling is documented; resource and connection limits still apply, so use waves and monitor failures. [Skills](https://cursor.com/docs/skills) · [Subagents](https://cursor.com/docs/subagents) |
| **Claude Code** | Project `.claude/skills/`; user `~/.claude/skills/`. | Worker definitions: `.claude/agents/` and `~/.claude/agents/`. Subagents can edit when their tools and permissions allow it; model and `effort` can be selected in supported versions. Shared checkout is default; `isolation: worktree` is available. The Agent tool's default concurrent limit is **20**; `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` adjusts it in v2.1.217+ (nesting limits are separate). [Skills](https://code.claude.com/docs/en/skills) · [Subagents](https://code.claude.com/docs/en/sub-agents) |
| **VS Code (by selected harness)** | Copilot harness: project `.github/skills/`, user `~/.copilot/skills/`. Claude/Codex harnesses use their native paths; `.claude/skills/` is supported by applicable VS Code harnesses. | Worker definitions can live under `.github/agents/`. The Copilot harness delegates with its native `task` tool; Claude/Codex harnesses use provider-native workers. The legacy Local harness uses `runSubagent` and is scheduled for deprecation/removal in February 2027. Model, tool permissions, edit access, isolation, and limits depend on harness and worker configuration. Select the harness before a session. Enterprise policy can disable Claude/Codex harnesses. [Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills) · [Subagents](https://code.visualstudio.com/docs/agents/run/subagents) |
| **Freebuff (CodebuffAI)** | Project `.agents/skills/` and `.claude/skills/`; `/skill:<name>` invocation is documented. | The CLI exposes `spawn_agents`; current source docs describe subagents as mostly read-only, so verify effective tools before assigning implementation. `.agents/*.ts`/`.js` definitions execute code. Desktop behavior differs by version; an August 2026 issue reported spawning absent in Desktop while present in CLI. No universal concurrency limit is stated here. [Freebuff source](https://github.com/CodebuffAI/freebuff) · [Agent and tool behavior](https://github.com/CodebuffAI/freebuff/blob/main/docs/agents-and-tools.md) · [Skills implementation](https://github.com/CodebuffAI/freebuff/blob/main/common/src/util/skills.ts) · [Desktop report](https://github.com/CodebuffAI/freebuff/issues/972) |

The Agent Skills format standardizes a folder containing `SKILL.md` with YAML metadata and instructions; it does not standardize skill installation paths or agent orchestration APIs. See the [Agent Skills specification](https://agentskills.io/specification). The shared project path `.agents/skills/` is documented for Codex, Cursor, and Freebuff; Claude Code uses `.claude/skills/`; Copilot uses `.github/skills/` in its Copilot harness. Other Agent Skills clients are not individually verified here: format support alone does not imply delegation, code writing, per-worker model/effort controls, or isolation.

## Installation

Clone or download this repository, then copy the complete `adaptive-subagent-orchestration` folder into one supported skill directory. Examples:

```sh
# Codex, Cursor, Freebuff, or another client supporting the shared project path
mkdir -p .agents/skills
cp -R adaptive-subagent-orchestration .agents/skills/

# Claude Code project installation
mkdir -p .claude/skills
cp -R adaptive-subagent-orchestration .claude/skills/

# GitHub Copilot harness in VS Code
mkdir -p .github/skills
cp -R adaptive-subagent-orchestration .github/skills/
```

For user-wide installation, use the documented home directory for that client, for example `~/.agents/skills/` for Codex/Cursor, `~/.claude/skills/` for Claude Code, or `~/.copilot/skills/` for the VS Code Copilot harness. Restart or refresh the client if it does not discover the new skill automatically. Project installation is usually preferable for remote workers because user-level files may not be available in their environment.

To update an existing installation, copy the source `SKILL.md` to the matching file in the installed skill directory. For example:

```sh
cp adaptive-subagent-orchestration/SKILL.md \
  ~/.agents/skills/adaptive-subagent-orchestration/SKILL.md
```

Use the destination for the client and scope where the skill is installed; copying the skill folder into itself can create an extra nested directory.

## Invocation

Clients may select skills automatically from the description or expose slash commands. Invoke it explicitly where supported, for example:

```text
Use adaptive-subagent-orchestration to plan and implement this feature.
```

Or select `/adaptive-subagent-orchestration` from a client's skill menu. A skill's presence does not guarantee automatic activation. Invocation also does not enable subagent tools; those must be available and configured by the client or runtime.

## How adaptive delegation works

The coordinator checks project instructions, runtime tools, worker edit permissions, definitions, limits, and workspace isolation. It compares expected parallel benefit with startup/context gathering, conflict risk, and integration/verification cost before delegating. It divides implementation into independent packages with explicit interfaces and write ownership, then fills available execution slots from a ready queue. When a worker completes, another independent ready task can start while its result is integrated; only dependent tasks wait for that integration. Concurrency goes down when conflicts, failures, resource pressure, or integration costs rise. It stops when additional workers would not help.

For very large programs, the queue can span many waves and potentially thousands of assignments. Fifty to one hundred simultaneous workers may be sensible only when the platform genuinely supports that capacity and enough work can be isolated. Neither figure is a target or promise; a skill cannot override host limits. The skill tracks active workers separately from assignments created, ready/blocked/completed/failed work, and available slots.

Subagents are implementation workers when useful, not just reviewers. They can own modules, components, endpoints, tests, docs, bug fixes, integrations, or isolated configuration changes. Shared interfaces and central files receive a single owner or serialized edits. Results are integrated and validated against the combined project.

## Model and reasoning effort

The skill recommends the lowest effort likely to succeed: low for mechanical changes, medium for routine well-specified coding, high for complex debugging or integration, and XHigh/Max only for unusually difficult work when supported. It treats model choice separately from reasoning effort, respects user preferences, and adjusts based on evidence. If a runtime does not expose those controls, the skill does not claim to set them.

## Project instructions and agent definitions

Before delegation, inspect root guidance and nested or path-scoped rules that apply to the files involved, using the detected host's documented paths and precedence. Candidates include `AGENTS.md`, `AGENTS.override.md`, `CLAUDE.md`, `.github/copilot-instructions.md`, `.github/instructions/`, `.cursor/rules/`, `.claude/rules/`, and documented equivalents. Inspect worker definitions only for the host and named worker being considered; verify the runtime reads those paths. For small sequential edits, check only guidance relevant to the touched files. `AGENTS.md` usually provides project instructions and is not automatically a subagent definition. Reuse suitable existing workers; create a native definition only if supported, authorized, and useful beyond one assignment. Never replace user-created files; otherwise put role guidance directly in the task contract.

## Usage examples

**1. Small bug fix — no delegation.** “Fix the off-by-one error in this pagination helper and add a focused regression test.” If it is isolated and quick, the agent reads applicable instructions, makes the change, and validates it directly.

**2. Independent feature — several implementation workers.** “Add saved searches with a settings UI, storage API, and coverage.” After agreeing on the interface, an enabled runtime can assign the UI, API/storage, and independent test work to distinct owners. The coordinator integrates and runs relevant checks.

**3. Large application — many workers and waves.** “Build the multi-tenant dashboard from this specification.” The agent can break work into frontend, backend, API contract, authentication, data migrations, infrastructure, tests, and documentation. It schedules only independent packages at first, unlocks dependent work by wave, and adapts to live limits and conflicts.

## Practical limits and safety

- No universal concurrency setting, model-control API, nested delegation support, or shared-filesystem isolation is assumed.
- A platform's hard cap, available models, permissions, billing, rate limits, and machine resources take precedence.
- Do not modify global settings to force a higher limit. Avoid excess workers when coordination costs exceed throughput gains.
- Treat skills and project instructions as influential input to tool use: review their source, keep permissions scoped, and never allow them to override user intent or safety requirements. Treat native agent definitions and MCP configuration as executable code where the platform does; inspect their origin, follow trust prompts, and never bypass host trust gates. Workers may inherit parent credentials and tools; provide only the context and permissions needed and do not pass unnecessary secrets or private data.
- Delegation does not expand user authorization. Unless the request explicitly covers the action, workers must not push or merge code, deploy, publish, send external messages, mutate production data, or perform other consequential external actions. Narrow permissions where supported; otherwise assign only work safe for inherited access. If an authorized action has an unclear target or environment, do not guess; ask only for the missing detail.
- Preserve unrelated work. Do not force-push, discard worker changes, skip required validation, or fabricate progress.

## Validation performed

The Skill Creator quick validator passes for this folder. Five independent read-only audits and two scenario walkthroughs informed the workflow; a Skill Creator review helped remove duplicated and speculative guidance. Platform behavior was cross-checked against vendor documentation and, for Freebuff, its public source. No live worker-runtime test or worker-count benchmark is claimed.



## `evidence-first-research`

[`evidence-first-research/SKILL.md`](evidence-first-research/SKILL.md) helps an assistant find, verify, assess, and cite sources for factual, academic, journalistic, policy, and technical questions. It starts with the exact claim and requested source constraints, searches for original evidence, inspects the source itself, and reports what it does and does not support.

### What it covers

- Classifies the document's **format**, **publisher**, **evidence role**, and **quality for the claim** separately. It distinguishes research papers and preprints, government and institutional reports, statistical releases, journalism and investigations, newsletters, blogs, opinion, press releases, corporate and NGO publications, official documentation, primary records, and secondary summaries.
- Evaluates evidence, method, relevance, currency, independence, conflicts, and verifiability without assuming a domain or publication format determines quality.
- Labels claim support as direct, partial, indirect, contradictory, absent, or unverified; ranks useful sources qualitatively.
- Traces claims to original documents, checks figures and context, looks for independent corroboration where warranted, and gives specific, verified citations. It never invents quotations or source details.
- Respects hard source and date constraints and reports evidence gaps without silently weakening those constraints.

### Install and invoke

Copy the complete folder into a skill directory recognized by your assistant, then follow that assistant's current discovery and invocation instructions. Verified examples include:

| Client | Project location | Personal location |
| --- | --- | --- |
| Codex | `.agents/skills/` | `~/.agents/skills/` |
| Cursor | `.agents/skills/` or `.cursor/skills/` | `~/.agents/skills/` or `~/.cursor/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| GitHub Copilot in VS Code | `.github/skills/` | `~/.copilot/skills/` |
| Freebuff | `.agents/skills/` | Check the current client documentation |

See the clients' [Codex](https://developers.openai.com/codex/skills), [Cursor](https://cursor.com/docs/skills), [Claude Code](https://code.claude.com/docs/en/skills), [VS Code](https://code.visualstudio.com/docs/agent-customization/agent-skills), and [Freebuff](https://github.com/CodebuffAI/freebuff/blob/main/common/src/templates/initial-agents-dir/skills/README.md) documentation for current installation and discovery details. The table describes common supported locations, not a guarantee that every harness or hosted agent loads every path. The `SKILL.md` format is portable; installation, automatic selection, slash commands, and research tools are not universal.

```sh
# From this repository, for a client that discovers .agents/skills/
mkdir -p .agents/skills
cp -R evidence-first-research .agents/skills/

# Or install the folder under the location documented by your client.
```

Where supported, select the skill by name or ask: “Use evidence-first-research to check whether this source supports my claim.” Clients may load it automatically when relevant; skill availability does not ensure automatic activation or provide browsing access.

### Workflow and citation behavior

The skill defines the proposition first, searches with topic synonyms and source-specific queries, opens promising source material, classifies the actual publication, locates the supporting passage/table/data, checks context and figures, and then presents a concise answer fitted to the request. Citations point to the original source when available, identify a precise page or section when verified, and sit next to supported claims. Quotations are exact and brief; paraphrases and inferences are labeled. If a source cannot be retrieved, support remains unverified.

### Example requests

- “Find a recent government report supporting this claim.”
- “Find three credible sources, but no blogs.”
- “Find the original research behind this statistic.”
- “Find a quote I can use in an essay.”
- “Is this newsletter reliable?”
- “Does this source actually support my argument?”
- “Find a stronger source than this article.”

### Limits

The skill does not itself browse the web, grant database access, guarantee exhaustive searching, or replace expert judgment. Its results depend on the host's retrieval tools and the accessible source material. Publication labels and domain reputation alone do not establish correctness.
