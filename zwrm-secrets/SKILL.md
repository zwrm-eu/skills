---
name: zwrm-secrets
description: |
  Manage encrypted secrets and environment variables for ZWRM apps and agents. Use this skill when the user wants to set environment variables, configure API keys, manage secrets, load a .env file, or inject configuration into running VMs. Triggers on "set secret", "environment variable", "env var", "API key", "configure secret", "load .env", "secrets", or "set DATABASE_URL".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm secrets

Encrypted secret and environment variable management for apps and coding agents. Secrets are stored encrypted in the control plane database and injected into VMs at boot.

## When to use

- You need to set API keys, database URLs, or other config for an app
- You want to load secrets from a .env file
- You need to manage secrets for coding agent VMs

## Quick start

```bash
# Set a secret for the current app
zwrm secrets set DATABASE_URL "postgres://user:pass@host:5432/db"

# Set a secret for a specific app
zwrm secrets set API_KEY "sk-..." --app my-api

# List secrets (shows names and metadata, never values)
zwrm secrets list

# Remove a secret
zwrm secrets unset DATABASE_URL
```

## Bulk loading

```bash
# Load all variables from a .env file
zwrm secrets add --from-file .env

# Load for a specific app
zwrm secrets add --from-file .env.production --app my-api
```

## Agent secrets

Coding agents have their own secret namespace:

```bash
# Set secrets for an agent
zwrm agent secrets set GITHUB_TOKEN "ghp_..."
zwrm agent secrets set ANTHROPIC_API_KEY "sk-ant-..."

# List agent secrets
zwrm agent secrets list

# Remove agent secret
zwrm agent secrets unset GITHUB_TOKEN

# Bulk load for agents
zwrm agent secrets add --from-file .env.agent
```

## Organization-wide secrets

```bash
# Set org-level secrets (inherited by all apps)
zwrm secrets set --org SHARED_API_KEY "key-value"

# List org secrets
zwrm secrets list --org
```

## Tips

- **`secrets list` never shows values** — only names, versions, and timestamps. This is by design.
- **Secrets are staged** until deployed. A new deployment picks up staged secrets automatically.
- **Use `--from-file`** for bulk loading instead of setting secrets one-by-one (avoids shell history exposure).
- **Agent secrets** are prompted interactively on first `zwrm agent` connect if not already set (GitHub token, API keys, git config).
- Secrets are encrypted at rest in the control plane database.

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy after setting secrets
- [zwrm-agent](../zwrm-agent/SKILL.md) — coding agents that consume these secrets
