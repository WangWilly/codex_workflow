---
status: accepted
partially_superseded_by: ADR-0003
---

# Package orchestration as a user-global stateless plugin

The workflow will become a user-global Codex plugin rather than a project-local installer. It will separate development state from execution strategy, select one primary skill through evidence-first explicit routing rules, and enforce a user-global permission ceiling that project configuration may only narrow. The orchestrator will reconstruct state from the current session, subagent reports, and existing workspace or tracker evidence instead of creating workflow-owned checkpoints; selected skills may still produce their declared artifacts. This avoids repeated project installation and state-file pollution while preserving explicit user control and Codex-native skill discovery.

ADR-0003 supersedes the primary-skill selection and explicit skill-routing parts of this decision. The user-global plugin, permission-ceiling, and stateless-orchestration decisions remain accepted.

## Consequences

Fresh sessions cannot recover plans that exist only in an earlier conversation. Existing project artifacts may be read as evidence, but the plugin will not require or initialize the legacy six-file `agent_docs` framework.
