---
name: better-subagent-use
description: Plan and coordinate coding work with subagents when useful. Use for implementation planning, parallel coding, multi-component features, large refactors, independent testing, or agent coordination; handle small sequential tasks directly.
---

# Better subagent use

Use available agents to finish more validated implementation work in less time. Delegation is a means, not a goal. This skill does not create agent tools, raise platform limits, grant permissions, or guarantee parallel execution. Use only capabilities exposed by the current runtime; adapt or work directly when a needed capability is absent, and never claim unsupported capabilities or results.

## 1. Discover the runtime and project

Before delegating:

1. Read the user request and derive concrete acceptance criteria. Preserve user constraints and requested validation.
2. Inspect root instructions and nested or path-scoped rules that apply to the files involved, using the active host's documented locations and precedence. Distinguish project guidance from worker definitions. Inspect native worker definitions only for a host and named worker you may use; verify the runtime reads them and inspect executable definitions or MCP configuration before using them. Follow trust prompts and never bypass a trust gate.
3. If delegation is useful, discover the runtime controls relevant to this task: worker availability and edit access, permission scope, concurrency, and any isolation or coordination features you plan to use. Narrow permissions where supported. If they cannot be narrowed, delegate only work safe with inherited tools and credentials; do not provide secrets or unrelated private data. Do not infer capabilities from the installed skill or agent name.
4. Reuse a suitable existing worker when available. Create a native definition only when supported, authorized, and useful beyond one assignment; never overwrite user files. Otherwise put role guidance in the brief. Report definitions created or changed.
5. Estimate the parallel benefit against startup and context-gathering time, shared-state and write-conflict risk, and integration/verification overhead. A small, serial, single-area task normally stays with the coordinator.

## 2. Decompose implementation work

Map the work into bounded packages by feature, component, or subsystem:

1. Identify shared interfaces, dependencies, high-risk changes, and validation needs.
2. Map dependencies and readiness in a small graph or checklist.
3. Define the minimum shared contract before parallel work that depends on it. Resolve it from the user's criteria and existing project behavior when possible; keep unresolved product decisions with the coordinator and clarify or record a supported assumption before assigning dependent work.
4. Put dependent work later in the queue.
5. Prefer assignments that produce code, tests, documentation, or configuration. When independent implementation exists, do not substitute a collection of audits or research tasks for implementation.
6. Give each package a clear benefit over doing it directly. Do not fragment work just to increase worker count.

Keep the coordinator productive on unowned work while workers run. Avoid changing files or interfaces owned by active workers. A review or integration step for one completed package should not hold up unrelated ready packages; start independent work while integration proceeds, and wait only for dependencies that actually require the result or a shared interface.

## 3. Schedule adaptively

Treat concurrency as a live capacity limit, not a target. Discover the host's available slots and documented caps. Maintain a ready queue and fill free slots with independent, useful work. When a worker completes, record its result and free its slot; immediately launch other ready work while integration proceeds. Unlock dependent packages only after their required inputs or shared interfaces are validated. Reduce concurrency when conflicts, failures, resource pressure, or integration cost rise; increase it when more independent work is ready and the system is healthy. Stop when remaining work is dependent, unsafe to split, or cheaper to finish directly.

Judge throughput by completed, validated work and integration effort against the direct-work estimate, not by worker count. Use observed delays, rework, failures, and cost to adjust later waves. Keep tracking lightweight; do not add instrumentation unless it helps make a real scheduling decision.

If capacity is unknown, start with a modest wave using host defaults and adjust from observed reliability and resource pressure. Never treat host limits as targets or change global settings to force more slots. Prevent recursive spawning unless the host supports it and the task tree and budget are explicit.

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
- In a shared checkout, wait until a worker has stopped writing, then inspect the current diff against the pre-task state and leave its already-applied edits in place. Do not cherry-pick or reapply the same changes. In an isolated workspace, review and selectively integrate its diff into the main checkout.
- Integrate deliberately. Preserve unrelated user changes. Resolve conflicts by inspection and coordination; never force-push, reset, or discard another worker's work to make integration easier.

## 6. Assign with a task contract

Give each worker enough context to act without rediscovering the task. Adapt this compact template:

```text
ID and objective:
Context and acceptance criteria:
Dependencies / inputs:
Owned files or component:
Allowed edits and prohibited overlap:
Authorized external actions and exact target/environment (if any); prohibited actions:
Relevant project instructions / existing agent role:
Available worker tools and permission scope:
Suggested model and reasoning effort (if supported):
Implementation and validation requested:
Definition of done:
Report: implementation, files changed, checks and outcomes, unresolved issues,
integration needs, and complete/blocked status.
```

Delegation never expands the user's authorization. Unless the request explicitly authorizes the action, workers must not push or merge code, deploy, publish, send external messages, mutate production data, or perform other consequential external side effects. If an authorized action has an unclear target or environment, do not guess; keep it with the coordinator and ask only for the missing detail.

Ask workers to report concise results, not raw logs or hidden reasoning. A coding assignment must name measurable outputs. When the runtime permits writes, make independent implementation workers edit-capable and assign them implementation; do not default to read-only solely because they are subagents. Confirm tool access and permissions first. If available workers are read-only, use them for tasks they can perform and keep implementation with an agent that has write access; do not describe research as implementation. A review-only assignment is appropriate when review is the actual need, not as the default role for all delegated work.

## 7. Monitor, integrate, and recover

Maintain a lightweight ledger of owners, dependencies, status (`ready`, `blocked`, `active`, `retrying`, `complete`, `failed`), active slots versus total assignments, retry cause and changed approach, changed areas, and validation. Use native messaging and wait/resume controls when available. For long efforts, checkpoint at phase boundaries only when a suitable, authorized artifact or tracker exists; do not assume persistence or create a file by default.

On failure, diagnose before retrying and mark the assignment `retrying` with a bounded attempt count. Default to one retry per assignment (two attempts total), and retry only when the cause is plausibly transient or the brief, context, or approach has materially changed. Never repeat an identical failed attempt. Allow another attempt only when there is a specific corrective reason and expected value; record it. Stop on missing authorization or capability, or when the same failure recurs without a new remedy.

- Missing context or unclear acceptance: repair the brief.
- File collision: reassign ownership, isolate work, or serialize.
- Worker crash, timeout, context exhaustion, tool/API error, permission failure, or environment fault: distinguish a transient failure from a missing capability; resume or reassign only when useful, otherwise report the blocker.
- Incomplete implementation: identify remaining acceptance items and use an available retry only with a revised brief or approach.
- Test failure: determine whether the change, test setup, or environment caused it.
- Reasoning failure: change the approach, scope, or justified effort.

Detect deadlocks, duplicate assignments, uncontrolled nested spawning, and diminishing returns; simplify, reassign, or finish directly.

## 8. Verify completion

Inspect each worker's changed-file list or diff and validate the combined result against the user's acceptance criteria; do not treat a completion message as proof. Run checks requested by the user or required by applicable project/host instructions, within the permissions and authorization provided. Do not run checks the user or host explicitly prohibits. Report checks that could not or were not run and why. Confirm required dependencies are complete, unresolved conflicts are handled, and active work has stopped. Report implemented work, validation outcomes, and material blockers clearly.

