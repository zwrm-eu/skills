---
name: zwrm
description: |
  MicroVM orchestration via the ZWRM CLI. Use this skill whenever the user wants to deploy an app, manage VMs, create sandboxes, manage Postgres databases, view logs, manage secrets, run coding agents, schedule agent runs, or connect MCP tools on ZWRM. Also use when they say "deploy", "launch", "scale", "spin up a VM", "create a sandbox", "start a database", "check status", or reference managing infrastructure on ZWRM. Do NOT trigger for local file operations, git commands, or code editing tasks unrelated to ZWRM.
allowed-tools:
  - Bash(zwrm *)
---

# ZWRM CLI

Deploy and manage lightweight Firecracker microVMs: apps, sandboxes, managed Postgres, persistent volumes, secrets, coding agents, and an MCP gateway — all through one CLI.

Generated from zwrm `v0.27.13`. `zwrm <command> --help` is always authoritative.

## Prerequisites

The `zwrm` binary must be installed and authenticated. Check with:

```bash
zwrm auth whoami
```

If not installed or not logged in, see [rules/install.md](rules/install.md). For security guidelines, see [rules/security.md](rules/security.md).

## Picking the right command

| Need | Command | Skill |
| --- | --- | --- |
| Deploy an application | `zwrm deploy` | [zwrm-deploy](../zwrm-deploy/SKILL.md) |
| Run ephemeral code in isolation | `zwrm sandbox create` / `zwrm sandbox run` | [zwrm-sandbox](../zwrm-sandbox/SKILL.md) |
| Managed database | `zwrm postgres create` | [zwrm-postgres](../zwrm-postgres/SKILL.md) |
| Persistent storage | `zwrm volumes create` | [zwrm-volumes](../zwrm-volumes/SKILL.md) |
| Environment variables / API keys | `zwrm secrets set` | [zwrm-secrets](../zwrm-secrets/SKILL.md) |
| See VM output, get a shell | `zwrm logs` / `zwrm ssh` | [zwrm-logs](../zwrm-logs/SKILL.md) |
| Check what's running | `zwrm status` | [zwrm-status](../zwrm-status/SKILL.md) |
| Autonomous coding agent | `zwrm agent run` | [zwrm-agent](../zwrm-agent/SKILL.md) |
| Cron / event-driven agent runs | `zwrm schedules` / `zwrm triggers` | [zwrm-agent](../zwrm-agent/SKILL.md) |
| MCP tools for agents | `zwrm mcp` | [zwrm-mcp](../zwrm-mcp/SKILL.md) |

## Command map

```text
zwrm admin      Admin operations for the control plane
zwrm agent      Manage agents and their workspaces
zwrm auth       Manage authentication
zwrm credits    Prepaid credits: balance, history, usage
zwrm dashboard  Show monitoring dashboard URLs
zwrm deploy     Deploy an application
zwrm destroy    Destroy machines or entire application
zwrm host       Manage hosts in the cluster
zwrm init       Initialize a new zwrm.toml configuration
zwrm logs       View VM console logs
zwrm mcp        Run an MCP server exposing autonomous agent runs (stdio)
zwrm org        Manage organizations and org resources
zwrm postgres   Manage PostgreSQL databases
zwrm routes     Manage application routes
zwrm sandbox    Manage sandboxes
zwrm scale      Scale application to desired number of machines
zwrm schedules  Manage scheduled agent runs (cron → agent runs)
zwrm secrets    Manage application secrets
zwrm sftp       Transfer files to/from a running VM via SFTP
zwrm skills     Manage the skill library (validated bundles agents can enable)
zwrm ssh        SSH into a running VM or run a remote command on it
zwrm status     Show application and machine status
zwrm templates  Manage agent templates
zwrm transfer   Move an app to another organization
zwrm triggers   Manage inbound triggers (external events → agent runs)
zwrm volumes    Manage persistent volumes
```

## Global flags

Available on every command:

```text
      --api-url string   Control plane API URL (default: https://zwrm.io)
      --config string    config file (default is $HOME/.zwrm/config.toml)
      --org string       Organization slug or ID (overrides env/config)
  -v, --verbose          Enable verbose output
```

## Authentication and orgs

```text
zwrm auth — Manage authentication

zwrm auth login — Login via browser or API token
      --token string   API token (for headless/SSH environments)

zwrm auth logout — Clear saved credentials

zwrm auth token — Show current token info

zwrm auth tokens — Manage API tokens

zwrm auth tokens create — Create a new API token (optionally capability-scoped)
      --expiry string    Token expiry duration (e.g. '8760h' for 1 year, empty for no expiry)
      --name string      Token name (e.g. 'github-actions')
      --org string       Organization id or slug the scoped token is bound to (default: your target org)
      --scopes strings   Capability set (comma-separated resource:action, e.g. apps:read,deploy,logs:read); empty = unrestricted token

zwrm auth tokens list — List API tokens

zwrm auth tokens revoke <token-id> — Revoke an API token

zwrm auth whoami — Show current authenticated user

zwrm org — Manage organizations and org resources

zwrm org create <name> — Create a new organization

zwrm org list — List organizations

zwrm org llm-provider — Manage external OpenAI-compatible model providers

zwrm org llm-provider add <name> — Register an external provider
      --allow-private        permit a private/internal address (host-admin only)
      --api-key string       bearer API key; "-" reads it from stdin (omit for keyless endpoints)
      --base-url string      OpenAI-compatible API root (required)
      --eu-resident          self-declare the endpoint as EU-resident (label only)
      --header stringArray   custom auth header "Name: value" (repeatable; e.g. Azure's api-key). Mutually exclusive with --api-key
      --model stringArray    model id served by the endpoint (repeatable; omit to discover)

zwrm org llm-provider disable <name> — Disable a provider's models

zwrm org llm-provider discover [name] — List the models an endpoint advertises, without registering it
      --api-key string       bearer API key; "-" reads it from stdin
      --base-url string      endpoint to probe (omit when naming a registered provider)
      --header stringArray   custom auth header "Name: value" (repeatable)

zwrm org llm-provider enable <name> — Enable a provider's models

zwrm org llm-provider list — List registered external providers

zwrm org llm-provider remove <name> — Remove a provider registration

zwrm org llm-provider show <name> — Show one provider and its models

zwrm org llm-provider sync <name> — Refresh the model list from the endpoint's /models listing

zwrm org llm-provider update <name> — Update a provider's URL, API key, or EU label
      --api-key string       new bearer API key; "-" reads it from stdin
      --base-url string      new OpenAI-compatible API root
      --clear-api-key        remove the stored credential (endpoint becomes keyless)
      --eu-resident          self-declared EU-resident label
      --header stringArray   replace the custom auth headers with "Name: value" (repeatable)

zwrm org secrets — Manage organization-wide secrets

zwrm org secrets add --from-file <path> — Add organization secrets from a file
      --from-file string   Path to .env file

zwrm org secrets list — List organization secrets

zwrm org secrets set <name> [value] — Set an organization secret
      --stdin   Read the secret value from standard input instead of argv

zwrm org secrets unset <name> — Remove an organization secret
```

Resources belong to organizations. The org scope for an invocation resolves in this order: `--org` flag → `ZWRM_ORG_ID` env var → `org` in `zwrm.toml`. With no scope, reads span all orgs you belong to and new resources land in your personal org.

API tokens can be capability-scoped (`zwrm auth tokens create --scopes apps:read,deploy,logs:read`) — prefer scoped tokens for CI and automation.

## Operator commands

`zwrm host` (host listing, drain/undrain for maintenance) and `zwrm admin` (control-plane backups, secrets-keyring rotation) are cluster-operator commands. They require host-admin access and are not needed for normal app work; see `zwrm host --help` and `zwrm admin --help`.
