---
name: zwrm
description: |
  MicroVM orchestration via the ZWRM CLI. Use this skill whenever the user wants to deploy an app, manage VMs, create sandboxes, manage Postgres databases, view logs, manage secrets, or connect to a coding agent. Also use when they say "deploy", "launch", "scale", "spin up a VM", "create a sandbox", "start a database", "check status", or reference managing infrastructure on ZWRM. This provides a complete control plane for deploying and managing lightweight Firecracker microVMs. Do NOT trigger for local file operations, git commands, or code editing tasks unrelated to ZWRM.
allowed-tools:
  - Bash(zwrm *)
---

# ZWRM CLI

MicroVM orchestration CLI. Deploy apps, manage sandboxes, Postgres databases, secrets, logs, and coding agents — all backed by Firecracker microVMs.

Run `zwrm --help` or `zwrm <command> --help` for full option details.

## Prerequisites

Must be installed and configured. Check with `zwrm status`.

If not ready, see [rules/install.md](rules/install.md). For security guidelines, see [rules/security.md](rules/security.md).

## Workflow

Follow this pattern based on what you need:

| Need                          | Command / Skill                                    | When                                           |
| ----------------------------- | -------------------------------------------------- | ---------------------------------------------- |
| Deploy an application         | `zwrm deploy`                                      | Have a Dockerfile or zwrm.toml                 |
| Run ephemeral code            | `zwrm sandbox create`                              | Need a disposable VM for execution             |
| Persistent coding environment | `zwrm agent <type>`                                | Need a long-lived dev VM with persistent state |
| Managed database              | `zwrm postgres create`                             | Need a Postgres instance                       |
| Environment variables         | `zwrm secrets set`                                 | Configure app or agent secrets                 |
| View output                   | `zwrm logs`                                        | Check VM console output                        |
| Check state                   | `zwrm status`                                      | See app, machine, or system status             |
| Scale up/down                 | `zwrm scale <count>`                               | Adjust number of running machines              |

## Command Overview

```bash
# App lifecycle
zwrm init                          # Generate zwrm.toml
zwrm deploy                        # Build and deploy
zwrm status                        # Show app status
zwrm scale <count>                 # Scale machines
zwrm destroy [--all]               # Tear down

# Sandboxes
zwrm sandbox create --template python --timeout 10m
zwrm sandbox exec <id> -- <cmd>
zwrm sandbox destroy <id>

# Coding agents
zwrm agent claude                  # Connect to Claude agent VM
zwrm agent codex                   # Connect to Codex agent VM
zwrm agent list                    # List running agents

# Postgres
zwrm postgres create <name> --size shared-cpu-1x
zwrm postgres connect <name>
zwrm postgres backup enable <name>

# Secrets
zwrm secrets set <name> <value>
zwrm secrets list
zwrm secrets unset <name>

# Logs
zwrm logs --app <name>
zwrm logs --follow
```

For detailed command reference, run `zwrm <command> --help` or see the individual skill files.
