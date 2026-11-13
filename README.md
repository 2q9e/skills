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
- Effort calibration from low for mechanical edits to higher levels only when complexity and evidence justify them.
- Discovery and reuse of project instructions and native agent definitions, with careful creation only when supported and useful.

## Supported platforms and capabilities

The skill format and runtime behavior are separate. A client may load the same `SKILL.md` while supporting a different set of worker tools. Platform documentation was checked on November 13, 2026; runtime versions, settings, and limits can change.

| Platform | Skill discovery / install | Worker capability | Runtime caveat |
| --- | --- | --- | --- |
| **Codex (CLI / IDE / app)** | Project `.agents/skills/`; user `~/.agents/skills/`. Codex skill behavior is documented by OpenAI. | Some Codex runtimes expose native subagents; the OpenAI Agents API and Responses multi-agent interface support delegation when enabled. | Skill installation does not turn multi-agent on. API multi-agent uses a configured concurrency ceiling (documented default 3, recommended for most workloads); no universal local CLI/app cap is implied. [Skills](https://developers.openai.com/codex/skills) · [Agents API multi-agent](https://developers.openai.com/api/docs/guides/agents-api/multi-agent) |
| **Cursor** | Project `.agents/skills/` or `.cursor/skills/`; user `~/.agents/skills/` or `~/.cursor/skills/`. Also recognizes Claude/Codex skill directories. | Supports parallel custom subagents in editor, CLI, and Cloud Agents, plus isolated cloud work. | Custom agents can select a model; per-model parameters such as effort vary by model and plan. User-level skills are not automatically copied to remote workers. [Skills](https://cursor.com/docs/skills) · [Subagents](https://cursor.com/docs/subagents) |
| **Claude Code** | Project `.claude/skills/`; user `~/.claude/skills/`. | Native subagents can implement code. Custom definitions live under `.claude/agents/`; built-in worktree isolation is available for subagents. | Model and effort behavior depends on model/version and agent definition. Use Claude Code's documented native definition and isolation behavior; a portable skill does not configure them. [Skills](https://docs.anthropic.com/en/docs/claude-code/skills) · [Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents) |
| **VS Code / GitHub Copilot** | Copilot harness project `.github/skills/`; user `~/.copilot/skills/`. VS Code also supports `.claude/skills/` in applicable harnesses. | Copilot harness supports subagent delegation; implementation permissions, model choices, and isolation are harness-specific. | In VS Code, select the harness before a session; enterprise policy can disable Claude/Codex harnesses. [Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills) · [Subagents](https://code.visualstudio.com/docs/agents/run/subagents) |
| **Freebuff (CodebuffAI)** | Project `.agents/skills/` and `.claude/skills/`; `/skill:<name>` invocation is documented. | Freebuff CLI documents `spawn_agents`; its current source says subagents are mostly read-only, so check the tools actually available before assigning implementation. | Do not infer Desktop parity: an August 2026 issue reported spawning absent in Desktop while available in CLI. Confirm the installed version. Local `.agents/*.ts`/`.js` definitions are executable and must be trusted. [Freebuff source](https://github.com/CodebuffAI/freebuff) · [Agent and tool behavior](https://github.com/CodebuffAI/freebuff/blob/main/docs/agents-and-tools.md) · [Skills implementation](https://github.com/CodebuffAI/freebuff/blob/main/common/src/util/skills.ts) · [Desktop spawning issue](https://github.com/CodebuffAI/freebuff/issues/972) |
| **Other Agent Skills clients** | Follow the client's documented skill directories and activation rules. | Varies. | Format compatibility does not imply subagent spawning, code writing, model/effort selection, or workspace isolation. |

The Agent Skills format standardizes a folder containing `SKILL.md` with YAML metadata and instructions; it does not standardize skill installation paths or agent orchestration APIs. See the [Agent Skills specification](https://agentskills.io/specification). The shared project path `.agents/skills/` is documented for Codex, Cursor, and Freebuff; Claude Code uses `.claude/skills/`; Copilot uses `.github/skills/` in its Copilot harness.

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

For user-wide installation, use the documented home directory for that client, for example `~/.agents/skills/` for Codex/Cursor or `~/.claude/skills/` for Claude Code. Restart or refresh the client if it does not discover the new skill automatically. Project installation is usually preferable for remote workers because user-level files may not be available in their environment.

## Invocation

Clients may select skills automatically from the description or expose slash commands. Invoke it explicitly where supported, for example:

```text
Use adaptive-subagent-orchestration to plan and implement this feature.
```

Or select `/adaptive-subagent-orchestration` from a client's skill menu. A skill's presence does not guarantee automatic activation. Invocation also does not enable subagent tools; those must be available and configured by the client or runtime.

## How adaptive delegation works

The coordinator checks project instructions, runtime tools, worker definitions, limits, and workspace isolation. It divides implementation into independent packages with explicit interfaces and write ownership, then fills available execution slots from a ready queue. Completion unlocks dependent work and opens capacity for the next package. Concurrency goes down when conflicts, failures, resource pressure, or integration costs rise. It stops when additional workers would not help.

For very large programs, the queue can span many waves and potentially thousands of assignments. Fifty to one hundred simultaneous workers may be sensible only when the platform genuinely supports that capacity and enough work can be isolated. Neither figure is a target or promise; a skill cannot override host limits. The skill tracks active workers separately from assignments created, ready/blocked/completed/failed work, and available slots.

Subagents are implementation workers when useful, not just reviewers. They can own modules, components, endpoints, tests, docs, bug fixes, integrations, or isolated configuration changes. Shared interfaces and central files receive a single owner or serialized edits. Results are integrated and validated against the combined project.

## Model and reasoning effort

The skill recommends the lowest effort likely to succeed: low for mechanical changes, medium for routine well-specified coding, high for complex debugging or integration, and XHigh/Max only for unusually difficult work when supported. It treats model choice separately from reasoning effort, respects user preferences, and adjusts based on evidence. If a runtime does not expose those controls, the skill does not claim to set them.

## Project instructions and agent definitions

Before delegation, inspect applicable root and nested instruction files and actual precedence, including `AGENTS.md`, `AGENTS.override.md`, `CLAUDE.md`, `.cursor/rules/`, `.claude/agents/`, `.codex/agents/`, `.agents/`, and documented native equivalents. Reuse suitable existing workers. `AGENTS.md` usually provides project instructions and is not automatically a subagent definition. Create a native worker definition only if supported, authorized, and useful beyond a single assignment. Never replace user-created files; otherwise put role guidance directly in the task contract.

## Usage examples

**1. Small bug fix — no delegation.** “Fix the off-by-one error in this pagination helper and add a focused regression test.” If it is isolated and quick, the agent reads applicable instructions, makes the change, and validates it directly.

**2. Independent feature — several implementation workers.** “Add saved searches with a settings UI, storage API, and coverage.” After agreeing on the interface, an enabled runtime can assign the UI, API/storage, and independent test work to distinct owners. The coordinator integrates and runs relevant checks.

**3. Large application — many workers and waves.** “Build the multi-tenant dashboard from this specification.” The agent can break work into frontend, backend, API contract, authentication, data migrations, infrastructure, tests, and documentation. It schedules only independent packages at first, unlocks dependent work by wave, and adapts to live limits and conflicts.

## Practical limits and safety

- No universal concurrency setting, model-control API, nested delegation support, or shared-filesystem isolation is assumed.
- A platform's hard cap, available models, permissions, billing, rate limits, and machine resources take precedence.
- Do not modify global settings to force a higher limit. Avoid excess workers when coordination costs exceed throughput gains.
- Treat skills and project instructions as influential input to tool use: review their source, keep permissions scoped, and never allow them to override user intent or safety requirements. Treat native agent definitions and MCP configuration as executable code where the platform does; inspect their origin, follow trust prompts, and never bypass host trust gates.
- Preserve unrelated work. Do not force-push, discard worker changes, skip required validation, or fabricate progress.

## Validation performed

This skill was statically checked for required frontmatter, name/folder agreement, README links and paths, and its core behavior requirements. Platform behavior was cross-checked against primary vendor documentation and, for Freebuff, its public source. No live worker-count benchmark is claimed; scale behavior is conditional on the selected runtime.

