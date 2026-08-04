---
status: accepted
---

# Allow two-layer orchestration for isolated delivery tracks

The Router will use a single-layer topology by default:

```text
Router -> leaf subagents
```

When an execution plan contains multiple independently deliverable feature- or pull-request-sized tracks and isolated parallel execution provides a material benefit, the Router may use one additional Router layer:

```text
Main Router -> Sub-routers -> leaf subagents
```

This hierarchical mode is subject to the following policy:

- Router depth is limited to two. A Sub-router cannot create another Router.
- The Main Router directs only Sub-routers. It does not directly dispatch leaf subagents or take ownership of a feature work package.
- Before creating the first Sub-router session, the Main Router discloses the hierarchical topology, track and wave counts, concurrent Sub-router count, maximum possible leaf concurrency, worktree isolation, and integration strategy.
- Selecting hierarchical mode does not require additional confirmation when the work is already authorized, the topology remains within applicable permission ceilings, and no unapproved commit, push, pull-request, or external side effect is introduced. The user may override the disclosed topology.
- Project-local policy may disable hierarchical mode or lower `max_concurrent_subrouters` and `max_leaf_agents_per_router`. It cannot enable a user-globally disabled topology or raise either user-global ceiling.
- A project-local value above its user-global ceiling is a policy conflict reported before delegation. It is not silently clamped. When no project-local override exists, the user-global policy applies.
- A project expresses these settings only through an optional `Orchestration policy override` section in the Git repository root `AGENTS.md`; no dedicated project policy file is required.
- The section may contain explanatory Markdown, but hard ceilings appear in a structured YAML code block. The Router parses only this explicitly delimited section and does not infer policy values from unrelated prose.
- Nested `AGENTS.md` files may constrain development behavior within their normal scope but cannot change Router topology or concurrency. If the repository root has no `AGENTS.md` or no override section, no project-local orchestration override applies.
- A present but malformed override produces `policy_conflict: invalid_project_override`. The Router must not replace it with user-global defaults or start any delegation.
- Override validation is strict. Unknown keys, duplicate keys, invalid types, and values above user-global ceilings are conflicts rather than ignored fields. Conflict output identifies the offending field and the accepted schema.
- While that conflict remains unresolved, the main agent may perform bounded read-only inspection to identify the invalid setting. Writes and delegation require the user to correct the override or explicitly authorize ignoring it.
- User authorization to ignore an invalid override applies only to the current execution plan. It does not modify `AGENTS.md`, persist into a new task or session, or survive a material replan.
- When such an exception is active, the Router uses the user-global policy and identifies the exception in its delegation disclosure.
- A material replan occurs when the user goal or completion definition changes, a feature- or pull-request-sized track is added, acceptance standards change, Router topology changes, or required permissions expand.
- Work-package decomposition, wave reordering, leaf-agent replacement within ceilings, defect or integration-conflict repair, and implementation changes within an accepted contract do not create a new execution-plan boundary.
- Each Sub-router runs in a new Codex session and a separate Codex-managed Git worktree.
- Each Sub-router uses the same Router runtime as the Main Router: model, reasoning effort, Router developer instructions, and user-global execution policy. Its project-local permission ceiling is identical or narrower.
- Runtime identity does not copy the Main Router conversation. The Sub-router receives its track capsule and isolated worktree context. If the host cannot guarantee the same Router runtime, hierarchical mode is unavailable.
- In hierarchical mode, the Main Router is a repository-read-only control plane. It may inspect diffs, commits, verification evidence, and workspace metadata and may operate Sub-router session lifecycle controls.
- Code, test, documentation, merge, and conflict-resolution writes are performed through a Sub-router-managed work chain. The Main Router can become a writable work agent only after a material replan to single-layer topology.
- Hierarchical mode may start from a dirty checkout only after the Main Router freezes a reproducible worktree baseline containing the Git base revision, baseline patch or snapshot reference and digest, and captured worktree status.
- Every Sub-router worktree starts from the same baseline, and each integration candidate contains only changes relative to it. Establishing the baseline does not require an otherwise unauthorized commit.
- An unresolved merge or rebase, an incomplete snapshot, or any inability to reproduce the same baseline in every worktree makes hierarchical mode unavailable. Topology disclosure identifies when a dirty baseline is used.
- Ignored files are not part of the worktree baseline or an integration candidate. The Router does not infer or modify `.worktreeinclude`; the Codex host may propagate ignored setup files only through the repository's existing `.worktreeinclude` and configured environment setup.
- A missing ignored dependency is a blocker rather than permission to copy it. Adding or broadening `.worktreeinclude`, especially for credentials or secrets, requires user authorization. Disclosure may identify that existing ignored-file propagation is active but never reveals secret values.
- A Sub-router is also repository-read-only. It plans its track, delegates to leaf agents, inspects evidence, supervises local verification, and prepares the integration packet without directly modifying the worktree.
- A Sub-router cannot take over implementation after leaf-agent failure. It follows the profile-selection, replacement, and escalation policy and reports an unrecoverable track failure to the Main Router when no safe worker remains.
- The same boundary applies in single-layer delegated mode: the Router remains read-only and leaf agents perform writes. When the main agent writes directly, the execution strategy is direct mode and no Router role is active.
- The Main Router delegates a track through a session-local track capsule containing the track identifier, outcome, base revision, edit scope, acceptance evidence, applicable contracts, available upstream integration candidates, permission ceiling, three-leaf-agent ceiling, and integration-packet requirements.
- Every track capsule has a monotonically increasing revision. An update identifies the revision it supersedes, explains the change, and identifies acceptance evidence invalidated by that change. A Sub-router accepts only a revision newer than the one it currently holds.
- Capsule history remains in the session rather than a workflow-owned file. Adding an upstream candidate or contract-preserving context may update a capsule within the current execution plan; changes to goal, feature scope, acceptance, topology, or permissions follow the material-replan policy.
- A track capsule omits the complete execution graph, sibling Sub-router transcripts, unrelated workspace history, the Main Router's internal reasoning, and rejected plans. Sibling scheduling remains private to the Main Router.
- The Main Router owns the global goal, cross-track dependency graph, integration order, final acceptance, and go/no-go decisions.
- Each Sub-router owns the workflow for one feature-, pull-request-, or integration-sized track, including task decomposition, leaf-subagent coordination, local verification, and evidence handoff.
- Sibling Sub-routers never merge or integrate each other's work. When integration requires code changes or conflict resolution, the Main Router assigns that work to a dedicated integration Sub-router.
- Feature and integration Sub-routers consume the same three-slot pool. No additional slot is reserved for integration.
- An integration Sub-router may start as soon as its dependency closure is available; it does not need to wait for every feature track. If all slots are occupied, integration remains queued until one is released.
- An integration Sub-router has the same three-leaf-subagent ceiling and replacement policy as any other Sub-router.
- A Sub-router reports through an ephemeral integration packet in the session rather than creating a workflow-owned checkpoint or handoff file.
- Before releasing its slot, a Sub-router must include a stable integration candidate in its integration packet. If local commit creation is authorized, the candidate contains the commit SHA, base revision, verification evidence, and worktree status. Otherwise it contains a complete diff or snapshot reference, base revision, verification evidence, and worktree status.
- Hierarchical mode does not grant permission to commit, push, or create a pull request. Those actions remain subject to the user's explicit authorization.
- The Main Router inspects integration candidates and plans their integration. Cherry-picking commits, applying diffs, or resolving conflicts is writable work assigned to an integration Sub-router.
- Leaf-agent proof, defect, and routine retry events remain in the owning Sub-router session. The Sub-router aggregates them into track-level evidence rather than forwarding the raw event stream.
- A Sub-router promotes shared-contract blockers, permission blockers, unrecoverable track failure, replacement needs, and final integration packets to the Main Router.
- User-facing lifecycle updates from the Main Router cover wave or track start, material blockers, integration readiness, rework, and final outcomes. Detailed leaf evidence remains inspectable in the Sub-router session.
- No aggregate `max_concurrent_agents` ceiling is used because it would ambiguously combine Router sessions and leaf subagents.
- The Main Router may run at most three Sub-router sessions concurrently.
- The Sub-router ceiling limits concurrent activity, not the total number of Sub-router sessions created over an execution plan. Additional tracks are queued and started in later waves when active Sub-routers release their slots.
- A downstream track with a hard dependency remains queued until the upstream integration candidate is available.
- Tracks with a contract dependency may run in parallel only when the Main Router supplies an explicit interface contract. The integrated result must be reverified against that contract.
- The Main Router cannot infer parallel safety merely from an absence of known conflicts; it must identify either track independence or an explicit contract dependency.
- The Main Router has exclusive authority to issue or revise a contract shared by multiple tracks. A Sub-router that discovers a required contract change reports a blocker with the cause, affected surface, and proposed revision rather than changing the shared contract unilaterally.
- The Main Router pauses affected work, updates the dependency graph and contract, and supplies revised context to affected Sub-routers. Acceptance evidence for impacted behavior under the old contract becomes invalid and must be regenerated.
- A contract revision that changes the user's requested behavior, a public API, or the acceptance definition requires user input before work resumes.
- A Sub-router releases its slot after it submits a final integration packet and the Main Router confirms that the packet is structurally complete. The associated session and worktree remain available while inactive.
- If later integration evidence requires more work from that Sub-router, reactivating it consumes a slot again. The track becomes completed only after integration acceptance.
- The Main Router may create at most one replacement Sub-router for a failed track. The replacement uses the same model as the Main Router and a new session.
- A replacement resumes the existing worktree only through a host-supported safe Handoff. Otherwise it reconstructs the track from accepted commits, diffs, and integration evidence. Orchestration must not silently discard uncommitted work from the original worktree.
- A replacement consumes a Sub-router slot. If the replacement also fails, orchestration stops and asks the user because the Router layer has no higher-capability model available for escalation.
- After integration acceptance, the Main Router marks the Sub-router completed but does not delete its worktree, branch, commit, snapshot, or session.
- Archival and worktree retention remain under user control and the Codex host retention policy. Orchestration never automatically cleans a worktree that is unintegrated, blocked, contains uncommitted changes, or may require rework.
- Final reporting identifies completed, pending-integration, blocked, and user-cleanable Sub-router sessions and worktrees without creating a cleanup checkpoint file.
- Any Router that dispatches leaf subagents may run at most three of them concurrently. This applies to the Router in single-layer mode and independently to each Sub-router in hierarchical mode; sibling Sub-routers do not share a global leaf-agent pool.
- If the active host cannot create a new Codex session with an isolated Git worktree, the Router must remain single-layer rather than emulate hierarchical mode in a shared checkout.

Hierarchical mode is not justified merely because a task is large. Before selecting it, the Main Router must establish that there are at least two tracks with distinct deliverables, non-overlapping initial edit scopes, independently evaluable acceptance evidence, and explicit integration boundaries. It must also establish that the expected parallelism benefit exceeds session, worktree, and integration overhead.

## Consequences

Feature- and pull-request-sized work can proceed concurrently without sharing a working tree or mixing local Router decisions. Integration authority remains centralized even though implementation is distributed. The Main Router becomes a coordinator rather than a worker in hierarchical mode, while each Sub-router retains full dynamic workflow planning within its assigned track and the global execution policy. A single-layer Router can run at most three leaf subagents concurrently. With at most three active Sub-routers and three leaf subagents per Sub-router, hierarchical mode can run at most nine leaf subagents concurrently. Execution plans with more than three tracks proceed in waves rather than being rejected or exceeding the ceiling. Waiting for integration does not consume active Sub-router capacity, but rework does.

Dependency classification determines whether parallelism is safe. Hard dependencies create wave boundaries. Contract dependencies permit parallel work but add an integration-time contract verification obligation.

Centralized contract authority prevents sibling tracks from drifting into mutually incompatible assumptions. It also makes contract changes observable policy events rather than incidental implementation details.

Sub-router failure recovery preserves worktree evidence rather than treating a fresh session as a clean restart. The one-replacement ceiling prevents repeated Router-session churn when no higher-capability Router model exists.

Worktree cleanup remains a host and user concern. Orchestration reports lifecycle state but does not add a second retention system or perform destructive cleanup.

Integration packets remain useful without turning orchestration into a publishing mechanism. A local implementation can become ready for integration while commit, push, and pull-request creation remain separate permission boundaries.

Topology disclosure keeps the cost and shape of hierarchical orchestration visible without turning routine, policy-compliant delegation into an approval gate.

Hierarchical evidence propagation protects the Main Router's context from leaf-level event volume while preserving detailed evidence in the Sub-router sessions where it was produced.

Git worktrees require a Git repository, are created from a selected starting branch, and may begin in detached `HEAD`. A branch cannot be checked out in more than one worktree simultaneously, so integration must use distinct branches or Codex Handoff rather than shared branch ownership. Worktree setup, ignored-file propagation, disk use, and cleanup are operational costs considered when choosing the topology.
