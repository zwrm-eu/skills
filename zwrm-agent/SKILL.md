---
name: zwrm-agent
description: |
  Connect to and manage persistent coding agent VMs on ZWRM. Use this skill when the user wants to launch a coding agent, connect to a Claude or Codex agent VM, manage agent instances, resize agent volumes, or view agent activity logs. Triggers on "coding agent", "connect to agent", "agent VM", "claude agent", "codex agent", "launch agent", "agent instance", "remote dev environment", or "persistent dev VM".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm agent

Persistent coding agent VMs with dedicated volumes. Connect to Claude, Codex, or other AI coding agents running in isolated Firecracker microVMs with persistent home directories.

## When to use

- You want a persistent remote dev environment for an AI coding agent
- You need an isolated VM with a persistent volume that survives restarts
- You want to manage multiple named agent instances

## Quick start

```bash
# Connect to a Claude agent VM (creates if needed)
zwrm agent claude

# Connect to a Codex agent VM
zwrm agent codex

# Connect to a named instance (separate volume)
zwrm agent claude my-project
```

## Instance management

```bash
# List all agent instances
zwrm agent list

# Destroy an agent VM (preserves volume for next connect)
zwrm agent destroy my-project

# Delete completely (VM + volume)
zwrm agent delete my-project

# Resize persistent volume
zwrm agent resize my-project --volume 20GB
```

## Agent secrets

Secrets are injected into agent VMs at boot:

```bash
# Set secrets (prompted interactively on first connect)
zwrm agent secrets set GITHUB_TOKEN "ghp_..."
zwrm agent secrets set ANTHROPIC_API_KEY "sk-ant-..."

# List secrets
zwrm agent secrets list

# Bulk load
zwrm agent secrets add --from-file .env.agent

# Remove
zwrm agent secrets unset GITHUB_TOKEN
```

## Activity logs

```bash
# View recent agent activity
zwrm agent logs my-project

# Limit lines
zwrm agent logs my-project --limit 50
```

## Tips

- **Named instances** let you maintain separate volumes per project. `zwrm agent claude project-a` and `zwrm agent claude project-b` are independent.
- **`destroy` vs `delete`**: `destroy` stops the VM but keeps the volume — next `connect` boots a fresh VM with the same data. `delete` removes everything.
- **First-time setup** prompts interactively for GitHub token, API keys, and git author info if no agent secrets are set.
- **Persistent volumes** contain the home directory. Code, config, and shell history survive across reconnects.
- Agent VMs use templates from `zwrm templates list` and can be customized.

## See also

- [zwrm-sandbox](../zwrm-sandbox/SKILL.md) — ephemeral VMs for one-off execution (no persistence)
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — manage secrets for agents and apps
- [zwrm-logs](../zwrm-logs/SKILL.md) — view VM console output
