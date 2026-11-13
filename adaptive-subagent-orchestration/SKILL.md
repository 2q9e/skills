---
name: adaptive-subagent-orchestration
description: Plan and coordinate coding work with subagents when useful. Use for implementation planning, parallel coding, multi-component features, large refactors, independent testing, or agent coordination; handle small sequential tasks directly.
---

# Adaptive subagent orchestration

Use available agents to finish more validated implementation work in less time. Delegation is a means, not a goal. This skill does not create agent tools, raise platform limits, grant permissions, or guarantee parallel execution. Use only capabilities actually exposed by the current runtime. If no usable delegation mechanism exists, work directly and report the limitation accurately.

## 1. Discover the runtime and project

Before delegating:

1. Read the user request and derive concrete acceptance criteria. Preserve user constraints and requested validation.
2. Inspect the project structure and applicable instructions. Check root and relevant nested `AGENTS.md`, `AGENTS.override.md`, `CLAUDE.md`, `.cursor/rules/`, `.claude/agents/`, `.codex/agents/`, `.agents/`, and documented equivalents supported by this runtime. Follow actual platform precedence; these files are not interchangeable. In particular, `AGENTS.md` is generally project guidance, not automatically a per-worker definition.
3. Discover the runtime's actual delegation, messaging, wait/resume, interruption, model/effort selection, concurrency, and workspace-isolation features. Do not infer these from the installed skill.
4. Check existing named agents and their configurations. Reuse a suitable agent if available.
5. Estimate independent work, resource pressure, and conflict risk. A small, serial, single-area task normally stays with the coordinator.

Treat worker definitions and MCP configuration as code when the host executes them. Check their source and required tools, honor platform trust prompts, and never bypass a trust gate. A Markdown skill does not make an executable agent definition safe.

If a suitable persistent agent definition is missing, first decide whether reuse will justify creating it. Only create a minimal native definition when supported, authorized, and useful beyond one short assignment. Never overwrite existing user files or create a permanent file for every worker. Otherwise put role-specific guidance in the assignment itself. Report any persistent definitions created or changed.

## 2. Decompose implementation work

Map the work into bounded packages by feature, component, or subsystem:

1. Identify shared interfaces, dependencies, high-risk changes, and validation needs.
2. Record dependencies as a small graph or checklist. Mark each package `ready`, `blocked`, `active`, `complete`, or `failed`; track active slots separately from total assignments.
3. Define shared contracts before parallel work when packages depend on them. Put dependent work later in the queue.
4. Prefer assignments that produce code, tests, documentation, or configuration. When independent implementation exists, do not substitute a collection of audits or research tasks for implementation.
5. Give each package a clear benefit over doing it directly. Do not fragment work just to increase worker count.

Keep the coordinator productive on unowned work while workers run. Avoid changing files or interfaces owned by active workers.

## 3. Schedule adaptively

Treat concurrency as a live capacity limit, not a target. Discover the host's available slots and documented caps. Maintain a ready queue and fill free slots with independent, useful work. When a worker completes, integrate or validate its result, unlock dependent work, then assign the next ready package. Reduce concurrency when conflicts, failures, resource pressure, or integration cost rise; increase it when more independent work is ready and the system is healthy. Stop when remaining work is dependent, unsafe to split, or cheaper to finish directly.

For exceptionally large projects, multiple waves may create 2,000 or more bounded assignments over time, and a runtime may support 50–100 active workers. These are conditional scale possibilities, never quotas. Do not claim or assume such capacity unless the runtime and workload support it. Never alter global settings to force a larger limit without explicit user direction. Prevent recursive spawning unless nesting is supported, useful, and bounded by an explicit task tree and budget.

## 4. Match model and reasoning to work

Use the least expensive, lowest reasoning level that is likely to succeed. Respect the user's choices and the model/runtime's documented controls. Model selection and reasoning effort are different settings. A skill cannot set either when the host does not expose the control.

| Work | Starting effort |
| --- | --- |
| Mechanical edit, formatting, narrow file update | Low |
| Well-specified feature, routine test, simple bug fix | Medium |
| Complex implementation, integration, difficult debugging | High |
| Unusually hard architecture or reasoning with material consequences | XHigh/Max only when supported and justified |

Use a faster or less costly model for routine, bounded work when available; reserve stronger models for uncertainty, complexity, or high-impact decisions. Raise effort or change the approach after evidence of a reasoning-related failure. Do not escalate for missing context, unavailable tools, permissions, or environmental faults.

## 5. Protect concurrent edits

Choose isolation based on real runtime behavior:

- Prefer native isolated workspaces, branches, or worktrees when available and practical.
- Otherwise assign disjoint file ownership in the shared workspace.
- Coordinate changes to shared interfaces, manifests, lockfiles, migrations, generated outputs, and central configuration. Give one worker ownership or serialize those edits.
- Do not assume workers have separate filesystems. State the write boundary and prohibited overlap in every assignment.
- Integrate deliberately. Preserve unrelated user changes. Resolve conflicts by inspection and coordination; never force-push, reset, or discard another worker's work to make integration easier.

## 6. Assign with a task contract

Give each worker enough context to act without rediscovering the task. Adapt this compact template:

```text
ID and objective:
Context and acceptance criteria:
Dependencies / inputs:
Owned files or component:
Allowed edits and prohibited overlap:
Relevant project instructions / existing agent role:
Suggested model and reasoning effort (if supported):
Implementation and validation requested:
Definition of done:
Report: implementation, files changed, checks and outcomes, unresolved issues,
integration needs, and complete/blocked status.
```

Ask workers to report concise results, not raw logs or hidden reasoning. A coding assignment must name measurable outputs. A review-only assignment is appropriate when review is the actual need, not as the default role for all delegated work.

## 7. Monitor, integrate, and recover

Keep a compact ledger of assignment IDs, owners, dependencies, status, changed areas, and validation. Use native messaging, wait, resume, and interruption tools only when available. Reuse a worker when continuity helps and the runtime supports it. For long efforts, save concise checkpoints in an appropriate project artifact only when authorized and useful; do not invent persistence features.

On failure, diagnose before retrying:

- Missing context or unclear acceptance: repair the brief.
- File collision: reassign ownership, isolate work, or serialize.
- Tool, permission, environment, or timeout fault: fix the cause or report the blocker.
- Incomplete implementation: identify the remaining acceptance items and resume or reassign once.
- Test failure: determine whether the change, test setup, or environment caused it.
- Reasoning failure: change the approach, scope, or justified effort.

Bound retries. Do not repeatedly send the same failed assignment unchanged or immediately escalate every failure to maximum effort. Detect deadlocks, duplicates, uncontrolled nested spawning, and diminishing returns; simplify, reassign, or finish directly.

## 8. Verify completion

Integrate finished work and check the combined result against the user's acceptance criteria. Run relevant tests, builds, type checks, and other requested verification using project instructions. Do not claim checks ran when they did not. Confirm required dependencies are complete, unresolved conflicts are handled, and active work has stopped. Report implemented work, validation outcomes, and material blockers clearly.

## Capability fallback

Use this conditional mapping without assuming equivalent features:

- Skills provide reusable instructions only; they do not create subagent support.
- If the host offers subagent spawning, assign bounded implementation and validation work through its documented interface.
- If it offers concurrency, use independent ready packages within the live limit.
- If it offers isolated workspaces, use them for likely conflicts.
- If it exposes per-worker model or effort controls, apply the guidance above; otherwise rely on configured defaults and task briefs.
- If agent definitions are supported, inspect and reuse native definitions; otherwise inline worker guidance.
- If a capability is absent, use the closest honest fallback or work directly. Never fabricate workers, isolation, limits, or completed results.
