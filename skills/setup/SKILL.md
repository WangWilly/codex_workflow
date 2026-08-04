---
name: setup
description: Install, verify, upgrade, or uninstall the Codex Workflow user-global agent profiles and required multi-agent v2 configuration. Use only when the user explicitly invokes setup and names one of those lifecycle operations; never invoke it during ordinary routing, plugin activation, or session startup.
---

# Setup

This skill is explicit-only. Never run it from the Router, another skill, plugin
activation, or a session-start hook.

Resolve paths before acting:

- Plugin root: two directories above this `SKILL.md`.
- Source profiles: every `*.toml` directly under `<plugin-root>/agents/`.
- Codex home: `$CODEX_HOME` when set, otherwise the current user's `.codex`
  directory.
- Installed profiles: `<codex-home>/agents/`.
- Host configuration: `<codex-home>/config.toml`.

Support exactly one requested mode: `install`, `verify`, `upgrade`, or
`uninstall`. If the user did not name a mode, ask for it.

## Safety invariants

- Treat source profiles and host configuration as one bounded setup scope.
- `verify` is read-only.
- Obtain explicit authorization before any write outside the current workspace.
- Never overwrite or remove an unmanaged profile.
- Never modify a user-edited managed profile.
- Never create a central receipt or transaction journal.
- Never use symlinks.
- Preserve comments, ordering, unrelated tables, and explicit existing values
  in `config.toml`.
- Do not modify `config.toml` during uninstall.
- Do not install a routing hook or another compatibility workaround.

## Managed-profile header

Prepend this comment-only header to every installed profile:

```text
# codex-workflow-managed-profile: true
# manager: codex-workflow
# source-profile: agents/<source-filename>
# source-version: <plugin-version>
# installed-at: <UTC RFC 3339 timestamp>
# payload-sha256: <SHA-256 of the exact source TOML bytes>
```

The payload begins after the header's final newline. Ownership requires every
header field to be present and `manager` to equal `codex-workflow`. Integrity
requires the checksum of the installed payload to match `payload-sha256`.

## Preflight

Complete preflight before requesting write authorization:

1. Read the plugin version from `.codex-plugin/plugin.json`.
2. Read and validate every source profile as TOML. Require a non-empty unique
   `name`, a model, reasoning effort, sandbox mode, and developer instructions.
3. Inspect every existing personal-agent TOML. Detect collisions by destination
   filename and by profile `name`.
4. Classify a destination as:
   - absent;
   - managed and unchanged;
   - managed but user-modified;
   - unmanaged.
5. Abort the entire operation on an unmanaged collision, duplicate identity,
   invalid source, malformed managed header, or user-modified managed profile.
6. Read `config.toml` if present and verify that it can be edited without
   replacing explicit values.

Do not perform a partial install or upgrade after a preflight conflict.

## Install

After preflight and authorization:

1. Create the personal-agent directory if necessary.
2. Stage every generated managed profile on the destination filesystem.
3. Add missing user-global configuration:

   ```toml
   [features.multi_agent_v2]
   enabled = true
   hide_spawn_agent_metadata = false
   tool_namespace = "agents"
   ```

   If the section exists, add only missing keys and preserve all existing
   values, including an explicit `enabled = false`.
4. Validate staged profiles and the complete resulting configuration.
5. Keep temporary rollback copies of destinations changed by this operation,
   then replace files atomically where the host supports it.
6. On a caught failure, restore the original files before removing temporary
   data. A later run must be able to converge after an ungraceful termination.
7. Report installed and preserved files, preserved configuration values,
   conflicts, and anything that still prevents orchestration.

Do not display or persist a separate activation-status marker.

## Verify

Perform no writes. Verify:

1. Every source profile has one installed counterpart with a valid management
   header, matching source identity, plugin version, and payload checksum.
2. No duplicate installed profile identity shadows a managed profile.
3. The effective multi-agent v2 configuration is enabled, exposes spawn
   metadata, and uses the expected active namespace.
4. The current host exposes direct concrete-profile selection plus the expected
   model and reasoning metadata.
5. No reserved-schema or profile-resolution incompatibility is observable.

Report each check as pass, fail, or unverified with evidence. Do not claim that
file deployment alone proves runtime compatibility.

## Upgrade

Use the install preflight and transactional procedure. Replace only unchanged
managed profiles. Preserve explicit configuration values and add only missing
compatibility keys. Abort before writes if any managed profile was edited by the
user or any destination now conflicts.

## Uninstall

After preflight and authorization, remove only profiles that have a valid
management header and a payload checksum matching their installed content.
Preserve user-modified managed profiles and report them. Do not modify any
`config.toml` key; explicitly report that the shared user-global configuration
remains.
