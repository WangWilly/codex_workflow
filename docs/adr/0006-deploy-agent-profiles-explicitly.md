---
status: accepted
---

# Deploy bundled agent profiles explicitly

The repository root will be the Codex plugin root. Its `agents/` directory will contain the canonical source bundle for all concrete custom-agent profiles.

Current Codex plugin packaging does not define `agents/` as an automatically installed resource. Personal custom agents are loaded from `~/.codex/agents/`, while project-scoped custom agents are loaded from `.codex/agents/`. Installing the plugin therefore cannot be described as sufficient to activate its bundled profiles.

The supported user-global installation flow has two explicit stages:

1. Install the plugin.
2. Invoke the plugin's bundled setup skill to copy the bundled TOML files into `~/.codex/agents/`.

The setup skill will use copies rather than symlinks because local, cached, and versioned plugin paths may change. It must detect name and destination conflicts and must not silently overwrite an existing profile.

The plugin will expose install, verify, upgrade, and uninstall behavior through the setup skill without requiring Node.js or another external installer runtime. The English `README.md` will document plugin installation, setup-skill invocation, a manual copy fallback, verification, restart or new-task requirements, upgrades, and uninstallation as one complete workflow.

The setup skill is explicit-only and cannot be selected through automatic model invocation, normal Router planning, plugin enablement, or a session-start hook. Users invoke its `install`, `verify`, `upgrade`, or `uninstall` mode directly. `verify` is read-only; every mode that changes `~/.codex/agents/` requires explicit user-global filesystem authorization before writes begin.

Install and upgrade also manage the minimum user-global multi-agent v2 compatibility configuration:

```toml
[features.multi_agent_v2]
enabled = true
hide_spawn_agent_metadata = false
tool_namespace = "collaboration"
```

If `[features.multi_agent_v2]` already exists, setup adds only missing keys and preserves existing values, including an explicit `enabled = false`. The setup result must distinguish successful file deployment from active orchestration: preserved values can leave the profiles installed while multi-agent v2 or direct profile selection remains unavailable.

This configuration is based on the typed configuration and tests in the pinned Codex source revision `5af85998c24fb3353ddd8164c3ed472057b03cb3`. That revision accepts a custom namespace but defaults to `collaboration`, marks multi-agent v2 stable while leaving it disabled by default, and defaults to hiding spawn metadata. The plugin uses `collaboration` because a namespace labels the host tool surface; it is unrelated to the `~/.codex/agents/` profile directory.

Because provider-visible spawn schemas can vary by Codex build and model, a parse check is insufficient. Verification requires a newly started task that confirms multi-agent v2 starts and exposes the expected profile, model, and reasoning metadata. A reserved-schema error or missing metadata is reported as an incompatible host configuration. Setup does not silently change the namespace, re-hide the metadata, or introduce hook-based role injection.

The multi-agent v2 section is shared, user-owned Codex host configuration after it is written. Uninstall removes only eligible managed profiles. It does not delete, revert, or otherwise mutate any `config.toml` key, because the plugin does not maintain a receipt that could prove ownership and other plugins or user workflows may depend on the same settings. The uninstall result explicitly reports that the shared configuration remains.

Installed profiles will be self-describing. The installer prepends a TOML comment header containing the managing plugin identity, source profile, source version, installation timestamp, and checksum of the TOML payload. No central installation receipt is created.

Upgrade and uninstall scan personal-agent TOML files and act only on files with a valid `codex-workflow` management header. Ownership does not depend on a filename prefix. If the payload checksum no longer matches, the setup skill treats the file as user-modified, preserves it, and reports the conflict. Files without the management header are never modified or removed.

Deployment preserves each source filename, such as `agents/executor_sol.toml` to `~/.codex/agents/executor_sol.toml`. Preflight checks both the destination filename and the TOML `name` identity across existing personal agents. Any unmanaged path or identity collision aborts the entire operation before writes begin; partial deployment is not allowed.

Install and upgrade use best-effort transactional deployment without a recovery journal. The setup skill completes every schema, filename, agent-name, checksum, and permission check before writing, stages all generated managed profiles on the same filesystem as `~/.codex/agents/`, validates the staged set, and keeps temporary rollback copies of managed destinations. It then replaces files through atomic renames. A runtime failure that the active tool environment can handle restores the original set before temporary data is removed.

An ungraceful process or machine termination can leave a mixture of individually complete old and new managed profiles. A later setup-skill run uses the self-describing headers and current source bundle to detect conflicts and converge the managed set to one version. The setup skill does not create an ephemeral or persistent transaction journal.

## Consequences

The plugin remains the canonical distribution unit, but Codex-native plugin installation and custom-agent deployment are separate operations. Users receive an accurate installation contract instead of an unsupported claim that the plugin manifest installs personal agent profiles automatically.

Copied profiles require explicit upgrade and uninstall behavior. Self-describing headers allow those operations to distinguish installer-managed files from unrelated or user-modified profiles without maintaining a second state file.

Successful deployment does not by itself prove runtime compatibility. Users must restart Codex or open a new task for the compatibility attestation, and an explicit incompatible value remains under user control rather than being overwritten.

Uninstallation is intentionally asymmetric: managed profiles can be attributed through their embedded headers, while shared host configuration cannot. Users who want to disable multi-agent v2 after uninstall must make that separate configuration decision themselves.
