---
status: accepted
---

# Keep agent profiles concrete and orchestration policy separate

Agent profiles remain concrete, model-bound worker configurations containing a stable role, model, reasoning effort, sandbox mode, and developer instructions. The Router dynamically owns workflow design, tool-use guidance, task decomposition, temporary edit scopes, implementation-verification arrangements, and escalation; independence, fallback chains, tool lists, and persistent ownership do not belong in profile metadata. This preserves simple profile Interfaces while allowing the Router to adapt orchestration to the current context.

`Router` always denotes a repository-read-only control plane. In delegated single-layer mode it coordinates leaf agents directly; in hierarchical mode the Main Router coordinates read-only Sub-routers, which coordinate leaf agents. A main agent that performs repository writes is operating in direct mode, not acting as a Router. Routers do not take over failed worker tasks.

Direct and delegated modes cannot have concurrent writers in the same checkout. A transition stops the outgoing writers, captures their base revision, diff, worktree status, and evidence, verifies that no writer remains active, and only then permits the incoming strategy to write.

## Consequences

Tool-use scope is advisory unless the active Codex host exposes an enforcement mechanism beyond the profile's sandbox mode. A task capsule contains only its work-package identifier, outcome, minimal context, and acceptance evidence, plus an edit scope or tool-use guidance when needed; dependency scheduling remains private to the Router. Writable work packages receive non-overlapping edit scopes, but the system creates no ownership registry. Every profile reports through a minimal proof, defect, blocker, or final evidence-event contract; intent-only progress does not count as evidence. The Router decides waiting, intervention, retry, and replacement from these events and may act before configured hard ceilings, but it cannot exceed them. Ineffective workers are replaced through central escalation guidance with a higher-capability profile, or orchestration stops for user input when no safe takeover exists.
