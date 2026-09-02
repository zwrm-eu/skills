---
name: zwrm-secrets
description: |
  Manage encrypted secrets and environment variables for ZWRM apps, organizations, and agents. Use this skill when the user wants to set environment variables, configure API keys, manage secrets, load a .env file, or inject configuration into VMs. Triggers on "set secret", "environment variable", "env var", "API key", "configure secret", "load .env", "secrets", or "set DATABASE_URL".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm secrets

Encrypted secret management at three scopes: **app** (`zwrm secrets`), **organization** (`zwrm org secrets`, inherited by the org's apps), and **agent** (`zwrm agent secrets`). Values are encrypted at rest in the control plane and injected into VMs at boot.

## Never put a secret value in the command

This is the rule that matters most in this skill. Do **not** write a literal
secret into a `zwrm` command — not even a redacted-looking one. Anything in argv
lands in the shell history and is readable by any process on the machine via the
process table. Use one of these three forms instead:

```bash
# 1. Pipe the value in — nothing sensitive in argv at all (best)
printf '%s' "$API_KEY" | zwrm secrets set API_KEY --stdin --app my-api

# 2. Reference an environment variable the user already exported
zwrm secrets set API_KEY "$API_KEY" --app my-api

# 3. Bulk-load from a .env file
zwrm secrets add --from-file .env.production --app my-api
```

If you don't have the value, **ask the user to set it** — either by exporting it
and letting you reference `"$VAR"`, by putting it in a `.env` file, or by running
the `--stdin` form themselves. Never invent, guess, or echo a secret value.

`--stdin` needs zwrm `v0.27.8` or newer. If it fails with `unknown flag:
--stdin`, the installed binary is older — fall back to form 2 or 3 above (both
work on every version), and suggest `zwrm auth whoami` / an upgrade. Don't fall
back to a literal value.

## App secrets

```bash
printf '%s' "$DATABASE_URL" | zwrm secrets set DATABASE_URL --stdin
zwrm secrets add --from-file .env.production --app my-api
zwrm secrets list          # names + metadata only, never values
zwrm secrets unset API_KEY
```

## Organization secrets

Shared across the org — good for keys every app needs:

```bash
printf '%s' "$SHARED_API_KEY" | zwrm org secrets set SHARED_API_KEY --stdin
zwrm org secrets add --from-file .env.shared
zwrm org secrets list
```

## Agent secrets

Coding agents have their own namespace (see [zwrm-agent](../zwrm-agent/SKILL.md)):

```bash
printf '%s' "$GITHUB_TOKEN" | zwrm agent secrets set GITHUB_TOKEN --stdin --instance work
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

zwrm secrets set <name> [value] — Set a secret
      --app string      Application name
      --app-id string   Application ID
      --stdin           Read the secret value from standard input instead of argv

zwrm secrets unset <name> — Remove a secret
      --app string      Application name
      --app-id string   Application ID
```

## Where these values go

Worth being explicit, because this skill handles credentials:

- `zwrm` sends secret values over HTTPS to **one destination**: the ZWRM control plane
  configured in `~/.zwrm/config.toml` (or `--api-url`), which is `https://zwrm.io` for the
  managed service and the org's own host for a self-hosted install. Nothing is sent anywhere else.
- Values are encrypted at rest there and served to the org's VMs at boot via an
  internal metadata service. The API never returns a stored value.
- `--from-file` uploads the parsed key/value pairs from the `.env` file you name — that file
  only. Confirm the path with the user before running it, so a broader `.env` than they
  intended doesn't get uploaded.
- `zwrm` is the platform's own CLI, published from the same source tree as this skill. Install
  it from the official location (see [rules/install.md](../zwrm/rules/install.md)) so the binary
  handling these values is the verified one.

## Tips

- **`secrets list` never shows values** — by design. Don't try to print them from inside the VM either unless the user explicitly asks.
- **New values apply at next boot/deploy.** Redeploy the app (or restart the agent) after changing secrets.
- **Use `--stdin` for single values and `--from-file` for bulk loads** — both keep values out of argv and shell history.
- Non-secret config can live in `zwrm.toml` under `[env]`; anything sensitive belongs here instead.

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy after setting secrets
- [zwrm-agent](../zwrm-agent/SKILL.md) — agent-scoped secrets
