# Codex Workflow Orchestration

This context defines the language used to package, install, and activate reusable Codex orchestration behavior across development projects.

## Language

**User-global installation**:
An installation scoped to one Codex user under that user's Codex home, available across all of their projects while remaining overridable or disableable per project.
_Avoid_: System-level installation, machine-wide installation, global installation

**Project-local installation**:
An installation copied into one project and available only when Codex operates in that project.
_Avoid_: Project-level installation

**Dynamic workflow orchestration**:
A mixed-autonomy model that synthesizes an ephemeral workflow from current user context and workspace evidence, then delegates through a constrained execution policy. It does not require selecting a predefined skill or workflow type.
_Avoid_: Skill routing, fixed workflow selection

**Global capability**:
Reusable orchestration behavior installed once in the Codex user's home and shared across projects, including skills, subagent definitions, routing policy, and generic workflow rules.
_Avoid_: Global project state

**Project state**:
Project-specific knowledge and execution history retained inside a project, including progress, architectural context, decisions, configuration overrides, and handoffs.
_Avoid_: Workflow installation

**Development state**:
The current kind of development activity, such as exploration, planning, implementation, diagnosis, verification, review, or handoff. It describes what is happening, independently of who performs the work.
_Avoid_: Route, workload size

**Execution strategy**:
The policy for performing work, including whether the main agent works alone or coordinates subagents and how much orchestration is proportionate.
_Avoid_: Development state, route

**Route preset**:
A compatibility shortcut such as light, medium, or heavy that selects an execution strategy without defining the development state.
_Avoid_: Workflow state

**Orchestration evidence**:
An ordered signal used to synthesize a workflow: explicit user direction, observable events, workspace state, then inferred natural-language intent. Low-confidence evidence cannot justify expensive or state-changing delegation.
_Avoid_: Trigger keyword, router guess

**Permission ceiling**:
The user-global maximum autonomy available to routing and skills. Project-local configuration may narrow this ceiling but cannot enable disabled capabilities, raise a safety level, or remove a required confirmation.
_Avoid_: Project permission, inherited trust

**Orchestration plugin**:
The installable and versioned distribution unit for global capabilities, bundling the router, skills, subagent definitions, metadata, and optional lifecycle resources.
_Avoid_: Workflow installer, global skill folder

**Stateless orchestration**:
An orchestration model that reconstructs development state from the current session, subagent reports, and existing workspace or tracker evidence without creating workflow-owned checkpoints, progress files, or handoff files. This does not prevent an explicitly used skill from producing its declared deliverables.
_Avoid_: Lazy checkpoint, workflow state store

**Execution plan**:
An ephemeral task graph synthesized for the current context, containing work packages, dependencies, capability roles, ownership, acceptance criteria, and evidence requirements.
_Avoid_: Workflow type, checkpoint, project plan file

**Evidence collector**:
A deterministic, read-only hook that gathers bounded session and workspace signals before a turn and emits orchestration context without designing a workflow or selecting a subagent.
_Avoid_: Workflow planner, state detector

**Orchestration context**:
The compact, structured evidence supplied to the model so it can synthesize an execution plan for the current turn.
_Avoid_: Checkpoint, prompt rewrite

**Orchestration disclosure**:
A brief notice shown when dynamic orchestration delegates work, naming the chosen execution strategy and decisive evidence while omitting internal reasoning and rejected alternatives.
_Avoid_: Routing trace, status report

**Capability role**:
An implementation-independent statement of the specialized ability a workflow needs, such as investigation, implementation, independent verification, or documentation.
_Avoid_: Subagent type, model role, worker name

**Execution policy**:
The canonical user-global runtime policy that decides whether work stays with the main agent or is delegated, then applies concurrency, sequencing, retry, replacement, and resource limits. ADRs explain why it exists but do not override it.
_Avoid_: Workflow, capability role

**Policy conflict**:
A typed incompatibility between a draft execution plan and the execution policy that must be normalized, repaired, or surfaced before delegation begins.
_Avoid_: Task failure, subagent failure

**Policy normalization**:
An automatic repair to a policy conflict that preserves the user's goal, scope, quality, permissions, and completion definition.
_Avoid_: Silent degradation, plan rewrite

**Agent profile**:
A concrete, model-bound Adapter that combines a stable role, model, reasoning effort, sandbox mode, and developer instructions.
_Avoid_: Capability role, workflow

**Agent constraint**:
A typed condition used to select a concrete agent profile for one work package, including required capability, task difficulty, sandbox compatibility, availability, and the global resource ceiling.
_Avoid_: Prompt hint, workflow step

**Compatible profile**:
An available agent profile that satisfies the requested capability role and every agent constraint for a work package. If no compatible or higher-capability profile can safely take over, orchestration stops and asks the user.
_Avoid_: Preferred agent, default worker

**Edit scope**:
The temporary writable surface assigned to one work package by the Router to prevent concurrent edits from overlapping. It is carried in the task capsule and is not a persistent ownership registry.
_Avoid_: Ownership, sandbox mode

**Escalation guidance**:
Execution-policy instructions for recognizing ineffective subagent work, replanning, selecting a higher-capability profile, and stopping for user input when no safe takeover profile exists.
_Avoid_: Profile fallback chain, automatic main-agent takeover

**Hard ceiling**:
A user-global maximum for concurrency, evidence-free retries, replacements, or another resource dimension. The Router may stop earlier based on evidence but cannot exceed the ceiling.
_Avoid_: Target, fixed workflow step

**Per-router leaf ceiling**:
The maximum number of leaf subagents that one Router may run concurrently. In hierarchical mode, each Sub-router receives its own ceiling; leaf concurrency is not pooled across sibling Sub-routers.
_Avoid_: Global agent ceiling, shared leaf pool

**Evidence event**:
A compact subagent message that identifies its work package, classifies the event as proof, defect, blocker, or final, provides observable evidence, and states the next action.
_Avoid_: Progress update, intent report

**Task capsule**:
A session-local work-package message containing its identifier, outcome, minimal context, and acceptance evidence, plus an edit scope or tool-use guidance only when needed. Scheduling dependencies remain private to the Router.
_Avoid_: Workflow plan, conversation summary

**Router topology**:
The delegation structure selected for one execution plan. It is single-layer by default and may expand to exactly two Router layers only for multiple independently deliverable feature- or pull-request-sized tracks that benefit from isolated parallel execution.
_Avoid_: Workflow type, unlimited recursion

**Main Router**:
The top-level Router in a two-layer topology. It directs only Sub-routers and owns the global goal, cross-track dependency graph, integration order, final acceptance, and go/no-go decisions.
_Avoid_: Leaf worker, feature implementer

**Sub-router**:
A Router running in a separate Codex session and Git worktree for one feature-, pull-request-, or integration-sized track. It uses the same model as the Main Router and owns local planning, leaf-subagent coordination, verification, and evidence handoff. It cannot create another Router.
_Avoid_: Nested Main Router, leaf subagent

**Integration packet**:
An ephemeral session message from a Sub-router containing the track identity, an integration candidate, acceptance evidence, integration dependencies or conflicts, and unresolved blockers. It is not a workflow-owned checkpoint file.
_Avoid_: Handoff document, progress file

**Integration candidate**:
The stable Git artifact offered for integration: an authorized commit with its SHA, or otherwise a complete diff or snapshot reference. It always identifies its base revision and worktree status and does not imply permission to commit, push, or create a pull request.
_Avoid_: Published branch, automatic commit

**Sub-router slot**:
One unit of the Main Router's concurrent Sub-router capacity. An active or reactivated Sub-router occupies a slot; an inactive Sub-router retains its session and worktree without occupying one while it awaits integration.
_Avoid_: Worktree, session lifetime

**Hard dependency**:
A relationship in which a downstream track requires an upstream implementation or integration candidate before it can be developed or verified correctly. The downstream Sub-router remains queued until that evidence exists.
_Avoid_: Scheduling preference

**Contract dependency**:
A relationship in which tracks can proceed independently against an explicit interface contract supplied by the Main Router. Parallel execution is allowed, but the integrated result must be reverified against that contract.
_Avoid_: Assumed compatibility, no dependency

**Shared contract authority**:
The Main Router's exclusive authority to issue or revise an interface contract used by multiple tracks. Sub-routers may propose changes through blocker evidence but cannot unilaterally change the shared contract.
_Avoid_: Contract ownership by a feature track

**Topology disclosure**:
The pre-delegation notice that identifies hierarchical orchestration, track and wave counts, concurrent Sub-router use, maximum leaf concurrency, worktree isolation, and the integration strategy. It informs the user without adding approval when execution remains within existing permission ceilings.
_Avoid_: Approval request, internal reasoning trace

**Orchestration policy override**:
An optional, explicitly delimited section in a project's `AGENTS.md` that narrows user-global orchestration settings. Its Markdown may explain intent, while a structured YAML block contains the machine-validated ceilings.
_Avoid_: Project-local skill installation, inferred prose setting

**Policy exception**:
An explicit, execution-plan-scoped user authorization to ignore an invalid project override and use the user-global policy. It does not modify project files, persist across sessions, or authorize a materially replanned execution.
_Avoid_: Policy normalization, permanent override

**Material replan**:
An execution-plan boundary caused by a change to the user goal, completion definition, feature- or pull-request-sized scope, acceptance standard, Router topology, or required permissions. Internal work-package decomposition, wave scheduling, ordinary worker replacement, defect repair, and contract-preserving implementation changes remain within the current plan.
_Avoid_: Any adaptive scheduling change

**Evidence propagation**:
The hierarchical rule that keeps leaf-agent evidence within its Sub-router and promotes only track-level contract or permission blockers, unrecoverable track failure, replacement needs, and final integration packets to the Main Router.
_Avoid_: Forward every leaf event, hide track blockers

**Track capsule**:
A versioned, session-local Main-Router-to-Sub-router message containing one track's identifier, outcome, base revision, edit scope, acceptance evidence, applicable contracts, available upstream integration candidates, permission ceiling, leaf-agent ceiling, and integration-packet requirements. It excludes the full execution graph and sibling session context.
_Avoid_: Leaf task capsule, complete Main Router context

**Router runtime**:
The concrete Router execution identity comprising its model, reasoning effort, Router developer instructions, and user-global execution policy. A Sub-router uses the same Router runtime as the Main Router while receiving only its track capsule and isolated workspace context.
_Avoid_: Model name alone, copied Main Router conversation

**Router control plane**:
The repository-read-only role of a Router while it coordinates delegated work. It may inspect evidence and operate delegation lifecycle controls but does not perform repository writes or take over failed worker tasks.
_Avoid_: Integration worker, shared-checkout editor

**Direct mode**:
An execution strategy in which the main agent performs the work itself and no Router role is active. The main agent may write within its permission and sandbox boundaries.
_Avoid_: Single-layer Router

**Delegated mode**:
An execution strategy in which a read-only Router coordinates writable leaf agents, either directly in single-layer topology or through read-only Sub-routers in hierarchical topology.
_Avoid_: Main-agent implementation

**Ownership barrier**:
The verified transition point between direct and delegated modes at which current writers have stopped, their base revision, diff, worktree status, and evidence are captured, and the incoming execution strategy has not yet begun writing.
_Avoid_: Concurrent handoff, persistent ownership registry

**Worktree baseline**:
The immutable starting state shared by every Sub-router worktree in one hierarchical execution, identified by a Git base revision plus an optional reproducible baseline patch, snapshot reference, digest, and captured worktree status. Integration candidates contain only changes relative to this baseline.
_Avoid_: Required checkpoint commit, mutable local checkout

**Profile source bundle**:
The plugin-root `agents/` directory containing the canonical custom-agent TOML files distributed with the plugin. Codex does not load personal agents directly from this directory.
_Avoid_: Installed agent directory

**Profile deployment**:
The explicit installation step performed through the plugin's bundled setup skill after the plugin itself is installed. It copies bundled agent profiles into `~/.codex/agents/`, checks conflicts, and does not rely on symlinks, an external installer runtime, or implicit plugin behavior.
_Avoid_: Automatic plugin installation

**Multi-agent v2 compatibility configuration**:
The user-owned, user-global `config.toml` section that explicitly enables multi-agent v2 and exposes the spawn metadata required for direct concrete-profile selection. New configuration uses the plugin's `agents` tool namespace. Existing explicit values are preserved and verified rather than silently replaced, and plugin uninstall does not remove or revert them.
_Avoid_: Agent-profile installation, project routing configuration

**Fresh-task compatibility attestation**:
A verification performed in a newly started Codex task after global configuration or profile changes. It confirms that multi-agent v2 starts successfully and that the host exposes the expected profile, model, and reasoning metadata without a reserved-schema error.
_Avoid_: Config-file parse check, installer success

**Self-describing managed profile**:
An installed custom-agent TOML file whose comment header identifies the managing plugin, source profile and version, installation time, and payload checksum. Install, upgrade, and uninstall discover ownership from this header rather than a filename prefix or central receipt.
_Avoid_: Filename-owned profile, external installation receipt
