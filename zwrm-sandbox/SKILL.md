---
name: zwrm-sandbox
description: |
  Create and manage ephemeral or persistent sandbox microVMs for code execution. Use this skill when the user wants to run code in an isolated VM, create a disposable environment, execute commands in a sandbox, upload or download files from a sandbox, or needs a temporary compute environment. Triggers on "sandbox", "run this in a VM", "execute in isolation", "ephemeral environment", "disposable VM", "spin up a sandbox", "run code safely", or "isolated execution".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm sandbox

Disposable or persistent microVMs for code execution. Ephemeral sandboxes auto-destroy after timeout. Persistent sandboxes suspend when idle and wake on demand.

## When to use

- You need to execute untrusted code safely
- You want a temporary compute environment
- You need an isolated shell for testing
- Step up from local execution when isolation matters

## Quick start

```bash
# Create ephemeral sandbox (auto-destroys after timeout)
zwrm sandbox create --template python --timeout 10m

# Create persistent sandbox (suspends when idle, wakes on demand)
zwrm sandbox create --template python --persistent --idle-timeout 10m

# Execute a command
zwrm sandbox exec <id> -- python3 -c "print('hello')"

# Interactive shell
zwrm sandbox exec <id> -- /bin/bash

# Upload and download files
zwrm sandbox upload <id> ./script.py /home/user/script.py
zwrm sandbox download <id> /home/user/output.json ./output.json
```

## Templates

Available templates for sandbox creation:

| Template | Description                  |
| -------- | ---------------------------- |
| `base`   | Ubuntu + common dev tools    |
| `python` | Python 3.12 + pip            |
| `node`   | Node.js 22 + npm             |

List all templates: `zwrm templates list`

## Lifecycle management

```bash
# List running sandboxes
zwrm sandbox list
zwrm sandbox list --status running
zwrm sandbox list --status suspended

# Check sandbox details
zwrm sandbox status <id>

# Extend ephemeral timeout
zwrm sandbox keepalive <id> --timeout 15m

# Wake a suspended persistent sandbox
zwrm sandbox wake <id>

# Destroy sandbox
zwrm sandbox destroy <id>
```

## Warm pool

Pre-booted VMs for instant sandbox creation (sub-200ms). Check pool status:

```bash
zwrm sandbox pool
```

## Tips

- **Ephemeral sandboxes** auto-destroy when their timeout expires. Use `keepalive` to extend.
- **Persistent sandboxes** auto-suspend via Firecracker snapshots after idle timeout. Any API call wakes them.
- **CoW overlays** mean sandbox creation doesn't copy the full base image — just creates a thin writable layer.
- **Templates** define the base environment. Use `zwrm templates create` for custom templates.
- Sandboxes run in isolated Firecracker microVMs with dedicated TAP network interfaces.

## See also

- [zwrm-agent](../zwrm-agent/SKILL.md) — persistent coding agent VMs with volumes (longer-lived than sandboxes)
- [zwrm-logs](../zwrm-logs/SKILL.md) — view sandbox VM console output
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — inject secrets into sandboxes
