---
allow_hierarchical_router: true
max_router_depth: 2
max_concurrent_subrouters: 3
max_leaf_agents_per_router: 3
max_subrouter_replacements_per_track: 1
---

# Execution Policy

## Purpose

This user-global policy constrains dynamic Codex workflow orchestration. The model synthesizes an ephemeral execution plan from the current user request, session context, and bounded workspace evidence.

Skills remain optional Codex capabilities. They may constrain or guide work when invoked, and they may create their declared deliverables, but they are not automatic routing nodes.

## Authority and precedence

User instructions, authorization boundaries, and the applicable sandbox remain authoritative. This policy can restrict delegation but cannot grant permission to write, commit, push, create a pull request, access a secret, or perform an external side effect.

A repository may narrow the user-global ceilings through the optional project override defined below. It cannot broaden them.

This file is the canonical runtime source for orchestration behavior. ADRs record decision context and trade-offs but do not override this policy. Any discovered inconsistency must be resolved by correcting the stale ADR or deliberately revising this policy through a new accepted decision.

## Host configuration prerequisites

Delegated execution requires multi-agent v2 to be active and its direct profile-selection metadata to be visible. For a new user-global configuration, the setup skill uses:

```toml
[features.multi_agent_v2]
enabled = true
hide_spawn_agent_metadata = false
tool_namespace = "collaboration"
```

If the section already exists, setup adds only missing keys and preserves every explicit existing value. In particular, it does not replace an explicit `enabled = false`, metadata-visibility choice, or tool namespace. Verification must report any preserved value that prevents this policy from operating.

These fields are host-version-sensitive even when accepted by the local configuration schema. After configuration or profile changes, setup requires a fresh Codex task to attest that multi-agent v2 starts and that the spawn interface exposes the expected concrete profile, model, and reasoning settings. A reserved-schema error or missing profile-selection metadata is an incompatibility. Setup must stop and report it; it must not silently install a hook-based routing workaround or switch namespaces.

The tool namespace names the host tool surface only. It does not install, discover, or identify profiles in `~/.codex/agents/`.

Once written, the multi-agent v2 section is user-owned shared host configuration. Uninstall removes eligible managed profiles but does not remove or restore any `config.toml` key. The uninstall result must disclose that the shared configuration remains.

## Strategy selection

The planner selects the least expensive strategy that can satisfy the requested outcome and acceptance evidence:

```text
Direct mode
Main agent performs the work; no Router role is active.

Delegated single-layer mode
Router -> leaf subagents

Hierarchical mode
Main Router -> Sub-routers -> leaf subagents
```

Direct mode permits the main agent to write within its existing authorization and sandbox. A Router is always a repository-read-only control plane and never acts as a worker.

Changing between direct and delegated mode, or between single-layer and hierarchical topology, is a material replan. Before the incoming strategy begins writing, the outgoing strategy must reach an ownership barrier: current writers stop, and their base revision, diff, worktree status, and evidence are captured. The strategy change requires a renewed orchestration disclosure.

## Plan validation

Before delegation, validate the draft execution plan against this policy. A policy conflict must be normalized, replanned, or surfaced before a subagent or Sub-router is created.

Automatic normalization is allowed only when it preserves all of the following:

- the user's goal;
- requested scope;
- acceptance and quality standards;
- permission boundaries;
- completion definition.

Any other repair requires user input. Policy normalization cannot silently degrade verification or broaden permission.

## Delegation disclosure

Before the first delegation, briefly disclose:

- the selected execution strategy;
- the outcome and decisive evidence;
- the capability roles and agent count;
- parallel or sequential execution.

Before hierarchical delegation, also disclose:

- hierarchical topology;
- track and wave counts;
- concurrent Sub-router count;
- maximum possible concurrent leaf-agent count;
- one-worktree-per-Sub-router isolation;
- integration strategy;
- whether a reproducible dirty baseline is used;
- whether existing ignored-file propagation is active;
- any execution-plan-scoped policy exception.

No additional confirmation is required when the work is already authorized, the topology remains within applicable ceilings, and delegation introduces no unapproved side effect. The user may override the disclosed topology.

## Router and worker boundaries

In delegated single-layer mode, the Router is read-only and leaf agents perform all repository writes.

In hierarchical mode:

- the Main Router directs only Sub-routers;
- the Main Router does not dispatch leaf agents or own a feature work package;
- each Sub-router is also repository-read-only;
- each Sub-router plans and supervises one feature-, pull-request-, or integration-sized track;
- leaf agents perform all code, test, documentation, local integration, merge, and conflict-resolution writes;
- a Router never takes over a failed worker's implementation.

The Router dynamically owns workflow design, work decomposition, tool-use guidance, temporary edit scopes, implementation-verification arrangements, waiting, intervention, retry, replacement, and escalation.

Tool-use guidance is advisory unless the active host exposes an enforcement mechanism beyond sandbox mode. Writable work packages receive non-overlapping edit scopes.

## Agent profile resolution

A capability role expresses an abstract need such as investigation, implementation, independent verification, or documentation. Resolve that role and its constraints to a concrete, model-bound agent profile.

An agent profile contains only its stable role, model, reasoning effort, sandbox mode, and developer instructions.

If no compatible or higher-capability profile can safely take over a work package, stop and ask the user. The Router must not silently become the fallback worker.

## Leaf task capsule

A leaf agent receives only a ready work package. Its task capsule contains:

```yaml
work_package:
outcome:
context:
acceptance_evidence:
edit_scope:       # required for writable work
tool_guidance:    # optional
```

The Router retains the scheduling dependency graph. Context includes any upstream output the leaf agent needs, but the leaf agent does not receive unrelated dependencies or the complete execution plan.

## Hierarchical eligibility

Single-layer topology is the default. Hierarchical mode is allowed only when:

- the work contains at least two independently deliverable feature- or pull-request-sized tracks;
- each track has a distinct outcome and independently evaluable acceptance evidence;
- initial edit scopes do not overlap;
- integration boundaries are explicit;
- isolated parallel execution provides a material benefit greater than session, worktree, and integration overhead;
- the active host can create a new Codex session and isolated Git worktree with the same Router runtime.

Task size alone does not justify hierarchical mode. If these conditions are not met, remain single-layer.

Router depth is limited to two. A Sub-router cannot create another Router.

## Router runtime

Every Sub-router uses the same Router runtime as the Main Router:

- model;
- reasoning effort;
- Router developer instructions;
- user-global execution policy.

Its project-local permission ceiling is identical or narrower. Runtime identity does not copy the Main Router conversation. If the host cannot guarantee the same Router runtime, hierarchical mode is unavailable.

## Track capsule

The Main Router delegates one track through a versioned, session-local track capsule:

```yaml
track_id:
capsule_revision:
supersedes_revision:
change_reason:
outcome:
base_revision:
edit_scope:
acceptance_evidence:
applicable_contracts:
available_upstream_candidates:
permission_ceiling:
max_leaf_agents: 3
integration_packet_requirements:
invalidated_evidence:
```

The capsule excludes the complete execution graph, sibling transcripts, unrelated workspace history, internal reasoning, and rejected plans. Sibling scheduling remains private to the Main Router.

Capsule revisions increase monotonically. A Sub-router accepts only a revision newer than its current revision. Adding an upstream integration candidate or contract-preserving context may update a capsule within the current execution plan. A change to the goal, feature scope, acceptance, topology, or permissions follows the material-replan policy.

Capsule history stays in session history.

## Concurrency and waves

The following ceilings are independent:

- at most three concurrent Sub-router sessions;
- at most three concurrent leaf agents for any Router that dispatches leaf agents.

There is no shared leaf-agent pool. A single-layer Router can run at most three leaf agents. Hierarchical mode can run at most nine leaf agents across three active Sub-routers.

The Sub-router ceiling limits concurrent activity, not the lifetime number of sessions. Additional tracks wait in later waves. A feature Sub-router and an integration Sub-router consume the same slot type; no extra integration slot exists.

An integration Sub-router may start when its dependency closure is available. It does not need to wait for every feature track, but it remains queued while all Sub-router slots are occupied.

## Dependencies and shared contracts

Classify every cross-track dependency:

- A hard dependency requires an upstream implementation or integration candidate. Keep the downstream Sub-router queued until that evidence is available.
- A contract dependency permits parallel work only against an explicit interface contract issued by the Main Router. Reverify the integrated result against that contract.

An absence of known conflicts is not evidence of independence.

The Main Router has exclusive shared-contract authority. A Sub-router that needs a contract change reports a blocker with the cause, affected surface, and proposed revision. It does not change the shared contract unilaterally.

The Main Router pauses affected work, revises the dependency graph and contract, and issues revised track capsules. Acceptance evidence for impacted behavior under the old contract becomes invalid. A revision that changes requested behavior, a public API, or the acceptance definition requires user input.

## Worktree isolation and baseline

Each Sub-router runs in a new Codex session and a separate Codex-managed Git worktree.

All worktrees in one hierarchical execution start from the same immutable baseline:

```yaml
base_revision:
baseline_patch_or_snapshot:
baseline_digest:
captured_worktree_status:
```

A dirty checkout is allowed only when the baseline is complete and reproducible in every worktree. Integration candidates contain only changes relative to that baseline. Establishing a baseline does not authorize a commit.

An unresolved merge or rebase, incomplete snapshot, or non-reproducible baseline makes hierarchical mode unavailable.

Ignored files are not part of the baseline or an integration candidate. The Router does not infer or modify `.worktreeinclude`. The host may propagate ignored setup files only through the repository's existing `.worktreeinclude` and configured environment setup. Missing ignored dependencies are blockers. Adding or broadening `.worktreeinclude`, especially for credentials or secrets, requires user authorization.

## Evidence propagation

Every leaf agent uses the common evidence-event interface:

```yaml
event: proof | defect | blocker | final
work_package:
evidence:
next_action:
```

Intent-only progress is not evidence.

Leaf proof, defect, and routine retry events remain in the owning Sub-router session. The Sub-router aggregates them into track-level evidence.

A Sub-router promotes only:

- shared-contract blockers;
- permission blockers;
- unrecoverable track failure;
- replacement needs;
- final integration packets.

The Main Router reports wave or track start, material blockers, integration readiness, rework, and final outcomes to the user. Detailed leaf evidence remains inspectable in its Sub-router session.

## Integration packet and slot lifecycle

Before releasing its active slot, a Sub-router submits a final integration packet containing:

```yaml
track_id:
integration_candidate:
base_revision:
worktree_status:
acceptance_evidence:
integration_dependencies:
known_conflicts:
unresolved_blockers:
```

The integration candidate is:

- an authorized commit and SHA; or
- a complete diff or snapshot reference when commit creation is not authorized.

Hierarchical mode does not grant permission to commit, push, or create a pull request.

After the Main Router confirms that the packet is structurally complete, the Sub-router becomes inactive and releases its slot. Its session and worktree remain available. If integration evidence requires rework, reactivation consumes a slot again. The track becomes completed only after integration acceptance.

Sibling Sub-routers never integrate each other's work. The Main Router plans integration, but any cherry-pick, diff application, merge, or conflict-resolution write is assigned to an integration Sub-router.

## Waiting, intervention, and failure

Lifecycle decisions are evidence-driven. Elapsed time alone is not proof of failure. The Router may wait, intervene, retry, or replace before a configured ceiling when evidence justifies it, but it cannot exceed a hard ceiling.

A failed track may receive at most one replacement Sub-router. The replacement:

- uses the same Router runtime;
- starts in a new session;
- consumes a Sub-router slot;
- resumes the existing worktree only through a host-supported safe Handoff;
- otherwise reconstructs from accepted commits, diffs, snapshots, and integration evidence;
- never silently discards uncommitted work.

If the replacement also fails, stop and ask the user. The Router layer has no higher-capability runtime available for escalation.

## Project-local override

Only the Git repository root `AGENTS.md` may contain this optional section:

````markdown
## Orchestration policy override

```yaml
allow_hierarchical_router: false
max_concurrent_subrouters: 2
max_leaf_agents_per_router: 2
```
````

The only accepted keys are:

- `allow_hierarchical_router`;
- `max_concurrent_subrouters`;
- `max_leaf_agents_per_router`.

The section may contain explanatory Markdown, but the Router parses hard ceilings only from its YAML block. Nested `AGENTS.md` files cannot change topology or concurrency.

Validation is strict:

- project values can only disable or lower user-global settings;
- unknown or duplicate keys are invalid;
- invalid types are conflicts;
- values above user-global ceilings are conflicts;
- invalid settings are never silently ignored or clamped.

An invalid section produces `policy_conflict: invalid_project_override`. Until resolved, the main agent may perform bounded read-only inspection, but it must not write or delegate.

The user may explicitly authorize ignoring an invalid override for the current execution plan. This policy exception:

- uses the user-global policy;
- appears in delegation disclosure;
- does not modify `AGENTS.md`;
- does not persist into a new task or session;
- expires on a material replan.

## Material replan

The following create a new execution-plan boundary:

- changing the user goal or completion definition;
- adding a feature- or pull-request-sized track;
- changing acceptance or quality standards;
- changing Router topology or direct/delegated mode;
- expanding required permissions.

The following remain within the current execution plan:

- internal work-package decomposition;
- wave reordering;
- leaf-agent replacement within applicable ceilings;
- defect and integration-conflict repair;
- implementation changes within an accepted contract;
- adding an available upstream candidate to a track capsule.

## Cleanup and retention

After integration acceptance, the Main Router marks a Sub-router completed. It does not delete the session, worktree, branch, commit, snapshot, or uncommitted work.

Archival and worktree retention remain under user control and the Codex host retention policy. Orchestration never automatically cleans an unintegrated, blocked, dirty, or potentially reusable worktree.

Final reporting identifies completed, pending-integration, blocked, and user-cleanable Sub-router sessions and worktrees. It does not create a cleanup checkpoint file.
