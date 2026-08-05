# Codex Workflow

Codex Workflow is a user-global, skills-only Codex plugin for adaptive
development orchestration. It reconstructs state from the active task and
workspace, synthesizes a proportionate workflow, and delegates to concrete
agent profiles under explicit concurrency, verification, fallback, and
worktree rules.

It does not install project-specific routes, create an `agent_docs/` framework,
or require a primary skill to define the workflow.

## What it provides

- Dynamic selection between direct execution, single-layer delegation, and
  two-layer worktree orchestration.
- A read-only Router control plane with writable leaf agents.
- Five concrete, model-bound profiles:

  | Profile | Purpose | Sandbox |
  | --- | --- | --- |
  | `explorer` | Bounded investigation and evidence gathering | Read-only |
  | `executor_luna` | Default implementation work | Workspace write |
  | `executor_sol` | Difficult or cross-cutting implementation | Workspace write |
  | `tester` | Independent verification and test work | Workspace write |
  | `doc-writer` | Evidence-backed durable documentation | Workspace write |

- Stateless orchestration: session and workspace evidence replace
  workflow-owned checkpoint files.
- At most three concurrent leaf agents per Router.
- Optional hierarchical execution with at most three concurrent Sub-routers,
  each in a separate Codex session and managed Git worktree.
- An explicit-only setup skill for profile installation, verification,
  upgrades, and uninstallation.

The canonical runtime rules are in
[`EXECUTION_POLICY.md`](EXECUTION_POLICY.md). Design rationale is recorded under
[`docs/adr/`](docs/adr/).

## Requirements

- A Codex surface with plugin and multi-agent v2 support.
- A Codex build compatible with the structured
  `[features.multi_agent_v2]` configuration used below.
- Git worktrees when hierarchical orchestration is requested.
- Permission to write to the user's Codex home when running setup lifecycle
  operations.

The bundled profiles use GPT-5.6 Luna and GPT-5.6 Sol model identifiers. Those
models must be available in the user's Codex environment.

## Install

Plugin installation and personal-agent deployment are separate operations.
Codex plugin packaging does not automatically copy the plugin's `agents/`
directory into `~/.codex/agents/`.

### 1. Place the plugin in a stable local path

```bash
mkdir -p ~/plugins
git clone https://github.com/WangWilly/codex_workflow.git \
  ~/plugins/codex-workflow
```

If that directory already contains a clone, update it instead:

```bash
git -C ~/plugins/codex-workflow pull --ff-only
```

### 2. Add it to the personal marketplace

Use Codex's built-in `$plugin-creator` once:

```text
Use $plugin-creator to add the existing plugin at
~/plugins/codex-workflow to my default personal marketplace.
Preserve the plugin source and do not scaffold or overwrite it.
```

This creates or updates the default personal marketplace entry. The entry must
be named `codex-workflow` and point to `./plugins/codex-workflow`.

### 3. Install the plugin

```bash
codex plugin add codex-workflow@personal
```

The plugin can also be installed from the **Personal** source in the Codex
plugin browser. Start a new Codex task after installation so its bundled skills
are discovered.

### 4. Deploy the user-global agent profiles

Explicitly invoke:

```text
Use $codex-workflow:setup in install mode.
```

The setup skill requests authorization before writing outside the workspace. It
copies managed profile files into `~/.codex/agents/` (or
`$CODEX_HOME/agents/`) and adds missing keys to the user-global Codex
configuration:

```toml
[features.multi_agent_v2]
hide_spawn_agent_metadata = false
tool_namespace = "agents"
```

If the section already exists, setup preserves every explicit value and adds
only missing keys. It aborts before writing when a destination filename or
agent identity conflicts with an unmanaged or user-modified profile.

#### Manual profile fallback

If the setup skill cannot be invoked, copy only missing files from the plugin's
`agents/` directory into `~/.codex/agents/`, then add the configuration block
above by hand. Never overwrite an existing filename or duplicate an existing
profile `name`.

Manual copies do not contain the setup skill's management header. They are
therefore unmanaged: later `upgrade` and `uninstall` operations will preserve
them, and the user must maintain or remove them manually.

### 5. Restart and verify

Restart Codex or open a new task, then invoke:

```text
Use $codex-workflow:setup in verify mode.
```

Verification checks profile identity and checksums, effective configuration,
direct profile selection, model and reasoning metadata, and reserved-schema
compatibility. The installer does not display or persist a separate activation
status marker.

## Use

For normal development work, describe the outcome directly. The
`orchestrate` skill may activate when delegation or structured verification is
useful, but it can choose direct execution when delegation would add needless
cost.

Invoke it explicitly when desired:

```text
Use $codex-workflow:orchestrate to implement and verify this feature.
```

For multiple independently deliverable feature- or pull-request-sized tracks:

```text
Use $codex-workflow:orchestrate. Plan these features as isolated parallel
tracks and use hierarchical worktree orchestration only if the policy permits.
```

The Router synthesizes the workflow from the current task and workspace. There
are no light, medium, or heavy route presets.

## Project-local limits

A repository can narrow the user-global ceilings without installing another
copy of the plugin. Add this optional section to the repository-root
`AGENTS.md`:

````markdown
## Orchestration policy override

```yaml
allow_hierarchical_router: false
max_concurrent_subrouters: 2
max_leaf_agents_per_router: 2
```
````

Only those three keys are accepted. Project values may disable or lower global
capabilities but cannot broaden them.

## Upgrade

Update the stable source clone and reinstall the marketplace plugin:

```bash
git -C ~/plugins/codex-workflow pull --ff-only
codex plugin add codex-workflow@personal
```

Start a new task, then invoke:

```text
Use $codex-workflow:setup in upgrade mode.
```

Open another new task and run `verify` after the upgrade.

## Uninstall

Run profile cleanup while the setup skill is still available:

```text
Use $codex-workflow:setup in uninstall mode.
```

Then remove the plugin:

```bash
codex plugin remove codex-workflow@personal
```

Setup removes only unchanged profiles carrying a valid Codex Workflow
management header. It preserves user-modified or unmanaged profiles.

The shared `[features.multi_agent_v2]` configuration remains after uninstall
because it is user-owned and may be used by other plugins. Disable or edit it
separately if it is no longer wanted.

## Compatibility note

The structured configuration above is present in the pinned Codex source
revision
[`5af85998c24fb3353ddd8164c3ed472057b03cb3`](https://github.com/openai/codex/tree/5af85998c24fb3353ddd8164c3ed472057b03cb3/codex-rs/config/src).
That revision defaults the tool namespace to `collaboration`, keeps
multi-agent v2 disabled unless explicitly enabled, and supports controlling
spawn-metadata visibility. Codex Workflow explicitly selects the `agents`
namespace to avoid the provider-reserved `collaboration.spawn_agent` schema.

Provider-visible spawn schemas can still vary by Codex build and model. If a
fresh-task verification reports a reserved-schema or missing-metadata
incompatibility, setup stops and reports it. This plugin does not silently
install a hook-based routing workaround.

## Development validation

From the plugin root:

```bash
uv run --with pyyaml python \
  ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/orchestrate

uv run --with pyyaml python \
  ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/setup

uv run --with pyyaml python \
  ~/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

These commands validate the two skill packages and the plugin manifest. Profile
TOML files should also be parsed before release.
