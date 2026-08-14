---
status: accepted
---

# Use Luna-first implementation escalation

Every writable implementation package starts with `executor_luna`, even when the Router anticipates difficult or cross-cutting work. `executor_sol` may take over only after Luna produces verifiable blocker or acceptance-failure evidence; this controls higher-capability-agent use without making difficulty estimates an implicit routing rule.

## Considered options

- Select Sol proactively for difficult or cross-cutting work.
- Require an attempted Luna assignment before any Sol assignment.

## Consequences

Escalation remains local to the failed package and explicitly dependent downstream packages. A Luna availability problem is not completion-failure evidence, so the Router stops for an explicit user exception instead of silently selecting Sol.
