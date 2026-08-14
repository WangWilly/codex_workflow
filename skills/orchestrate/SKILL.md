---
name: orchestrate
description: Synthesize and run a context-sensitive software-development workflow under the Codex Workflow execution policy. Use when a task may benefit from decomposition, multi-agent delegation, parallel implementation, independent verification, bounded worker replacement, or hierarchical worktree orchestration, and when deciding whether direct execution is more proportionate.
---

# Orchestrate

Read [`../../EXECUTION_POLICY.md`](../../EXECUTION_POLICY.md) completely before
planning, delegating, or changing execution strategy. Treat it as the canonical
runtime policy.

## Build the workflow

1. Reconstruct the goal, constraints, permissions, acceptance evidence, and
   current state from the active task and the smallest useful workspace
   inspection.
2. Do not create workflow-owned checkpoint, progress, handoff, or routing files.
3. Synthesize an ephemeral execution plan for the current context. Do not select
   a predefined workflow or primary skill.
4. Choose direct, delegated single-layer, or hierarchical execution using the
   policy's eligibility and proportionality rules.
5. Validate the plan against the execution policy before delegating. Normalize
   only conflicts whose repair preserves the user's goal, scope, quality,
   permissions, and completion definition.

## Delegate

- Resolve each capability role to one installed concrete agent profile:
  `explorer`, `executor_luna`, `executor_sol`, `tester`, or `doc-writer`.
- For writable implementation packages, apply the execution policy's
  Luna-first escalation rule; do not select Sol from estimated difficulty.
- Use the active host's exposed multi-agent tool namespace and schema. Do not
  infer a tool name from the profile directory or rewrite tool calls through a
  hook.
- Give every leaf agent a complete task capsule. Keep scheduling dependencies
  private to the Router.
- Keep writable edit scopes non-overlapping.
- Apply the policy's disclosure, concurrency, waiting, intervention,
  replacement, escalation, and evidence-propagation rules.
- Never let a Router become an implementation fallback.

Use another skill only when the user invokes it or its capability is genuinely
needed by the synthesized plan. That skill may constrain its own work and
produce its declared artifacts; it does not become the workflow.

## Finish

Integrate only through the permissions already granted by the user and host.
Report acceptance evidence, unresolved risk, pending integration, and retained
sessions or worktrees. Do not create a workflow checkpoint during handoff.
