---
status: superseded
superseded_by: ADR-0003
---

# Use hooks for evidence collection, not routing decisions

The orchestration plugin will bundle a trusted `UserPromptSubmit` hook that gathers bounded, read-only session and workspace evidence on every turn and injects it as compact routing context. The deterministic hook will not choose or invoke a skill; the model will evaluate explicit `ORCHESTRATION.md` rules and select at most one primary skill. This gives every turn a reliable routing entry point without reducing semantic routing to brittle scripts or allowing the hook to perform expensive scans, tests, or writes.

ADR-0003 replaced both the primary-skill router and the mandatory evidence hook. The active design reconstructs bounded evidence directly from the current task and workspace and synthesizes an ephemeral workflow under `EXECUTION_POLICY.md`.

## Consequences

Users must review and trust the plugin hook, and updated hook definitions may require renewed trust. Hook failure must leave native Codex behavior available rather than blocking ordinary work.
