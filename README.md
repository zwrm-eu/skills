# ZWRM skills

Skills that teach AI coding agents (Claude Code, Cursor, Codex, …) the `zwrm` CLI: deploying apps, sandboxes, managed Postgres, volumes, secrets, logs, coding agents, and the MCP gateway on ZWRM's Firecracker microVM platform.

## Install

Install all ten skills into one agent — Claude Code here:

```bash
npx -y skills add zwrm-eu/skills --agent claude-code --skill '*' --global --yes
```

Other common targets:

```bash
# Several agents at once
npx -y skills add zwrm-eu/skills --agent claude-code cursor codex --skill '*' --global --yes

# Just the hub skill, which links to the rest on demand
npx -y skills add zwrm-eu/skills --agent claude-code --skill zwrm --global --yes

# This project only, rather than your user directory (drop --global)
npx -y skills add zwrm-eu/skills --agent claude-code --skill '*' --yes

# Pick interactively
npx -y skills add zwrm-eu/skills
```

Prefer `--agent` over `--all`. `--all` is shorthand for `--agent '*' --skill '*' -y`, which
fans every skill out to every agent the installer knows about — dozens of tools you likely
don't have, and a wall of "does not support global skill installation" errors from the few
that have no global scope. Naming your agents avoids all of it.

## Skills

| Skill | Covers |
| --- | --- |
| [zwrm](zwrm/SKILL.md) | Hub: install, auth, org scoping, full command map |
| [zwrm-deploy](zwrm-deploy/SKILL.md) | Init, deploy, scale, destroy, transfer, routes |
| [zwrm-status](zwrm-status/SKILL.md) | App/machine status, dashboards |
| [zwrm-logs](zwrm-logs/SKILL.md) | Logs, SSH, SFTP into VMs |
| [zwrm-secrets](zwrm-secrets/SKILL.md) | App, org, and agent secrets |
| [zwrm-postgres](zwrm-postgres/SKILL.md) | Managed Postgres, backups, PITR |
| [zwrm-volumes](zwrm-volumes/SKILL.md) | Persistent volumes and snapshots |
| [zwrm-sandbox](zwrm-sandbox/SKILL.md) | Ephemeral/persistent sandboxes, templates |
| [zwrm-agent](zwrm-agent/SKILL.md) | Coding agents, runs, schedules, triggers, memory, skills |
| [zwrm-mcp](zwrm-mcp/SKILL.md) | MCP gateway: upstreams, virtual servers, OAuth |

## Generated — do not edit here

This repository is generated from the [zwrm-eu/zwrm](https://github.com/zwrm-eu/zwrm) source tree (`cmd/gen-skills`) and synced on every release. Command references are rendered from the CLI's actual Cobra command tree, so they match the shipped binary.

To change these skills, edit `cmd/gen-skills/src/` in the main repo. Pull requests against this repository will be overwritten by the next release.

Last generated from zwrm `v0.21.4`.
