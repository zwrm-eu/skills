---
name: zwrm-secrets
description: |
  Manage encrypted secrets and environment variables for ZWRM apps, organizations, and agents. Use this skill when the user wants to set environment variables, configure API keys, manage secrets, load a .env file, or inject configuration into VMs. Triggers on "set secret", "environment variable", "env var", "API key", "configure secret", "load .env", "secrets", or "set DATABASE_URL".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm secrets

Encrypted secret management at three scopes: **app** (`zwrm secrets`), **organization** (`zwrm org secrets`, inherited by the org's apps), and **agent** (`zwrm agent secrets`). Values are encrypted at rest in the control plane and injected into VMs at boot.

## App secrets

```bash
zwrm secrets set DATABASE_URL "postgres://user:pass@host:5432/db"
zwrm secrets set API_KEY "sk-..." --app my-api
zwrm secrets add --from-file .env.production --app my-api
zwrm secrets list          # names + metadata only, never values
zwrm secrets unset API_KEY
```

## Organization secrets

Shared across the org — good for keys every app needs:

```bash
zwrm org secrets set SHARED_API_KEY "value"
zwrm org secrets add --from-file .env.shared
zwrm org secrets list
```

## Agent secrets

Coding agents have their own namespace (see [zwrm-agent](../zwrm-agent/SKILL.md)):

```bash
zwrm agent secrets set GITHUB_TOKEN "ghp_..." --instance work
zwrm agent secrets add --from-file .env.agent
```

## Command reference

```text
zwrm secrets — Manage application secrets

zwrm secrets add --from-file <path> — Add secrets from a file
      --app string         Application name
      --app-id string      Application ID
      --from-file string   Path to .env file

zwrm secrets list — List secrets
      --app string      Application name
      --app-id string   Application ID

zwrm secrets set <name> <value> — Set a secret
      --app string      Application name
      --app-id string   Application ID

zwrm secrets unset <name> — Remove a secret
      --app string      Application name
      --app-id string   Application ID
```

## Tips

- **`secrets list` never shows values** — by design. Don't try to print them from inside the VM either unless the user explicitly asks.
- **New values apply at next boot/deploy.** Redeploy the app (or restart the agent) after changing secrets.
- **Use `--from-file`** for bulk loads — it keeps values out of shell history.
- Non-secret config can live in `zwrm.toml` under `[env]`; anything sensitive belongs here instead.

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy after setting secrets
- [zwrm-agent](../zwrm-agent/SKILL.md) — agent-scoped secrets
