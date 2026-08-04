---
status: accepted
---

# Synthesize workflows within a constrained execution policy

The orchestrator will synthesize an ephemeral execution plan from current user context and workspace evidence instead of selecting one predefined primary skill. Skills remain optional Codex capabilities and do not control automatic orchestration. A global execution policy constrains delegation, maps requested capability roles to runtime agent profiles, and supervises waiting, intervention, retry, replacement, and fallback behavior. This supersedes the primary-skill selection and skill-routing-rule aspects of ADR-0001; its user-global plugin, permission-ceiling, and stateless-orchestration decisions remain accepted. It also supersedes ADR-0002: orchestration reads bounded task and workspace evidence directly and does not require a `UserPromptSubmit` hook.

Changing between direct and delegated mode is a material replan. The outgoing writer or workers stop at a verified ownership barrier that captures the current base revision, diff, worktree status, and evidence before the incoming execution strategy begins writing. Strategy disclosure is renewed at the transition.

`EXECUTION_POLICY.md` is the canonical runtime source. ADRs preserve rationale and history; they do not override the active policy.

## Consequences

The system can adapt to workflow shapes not anticipated at design time, but dynamic plans must pass policy validation before any subagent is spawned. Policy conflicts may be normalized automatically only when the repair preserves the user's goal, scope, quality, permissions, and completion definition; all other conflicts require replanning or user input. Before the first delegation, the orchestrator discloses the strategy, capability roles, agent count, and sequencing without exposing internal reasoning. If no compatible or higher-capability agent profile can safely take over, orchestration stops and asks the user rather than silently transferring the package to the main agent. Workflow consistency is tested through policy invariants and execution-plan scenarios rather than a finite catalog of workflow types.
