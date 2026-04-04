---
name: zwrm-logs
description: |
  View and stream logs from ZWRM microVMs. Use this skill when the user wants to see VM console output, tail logs, stream logs in real-time, debug a running machine, or check what happened during a deployment. Triggers on "logs", "show me the output", "what's happening", "tail the logs", "stream logs", "debug the VM", "console output", or "check the error".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm logs

View VM console output. Supports historical log retrieval, real-time streaming, and filtering by app or machine.

## When to use

- You want to see what a VM is printing to its console
- You need to debug a failed deployment or crashed process
- You want to stream logs in real-time

## Quick start

```bash
# Logs for current app (reads zwrm.toml)
zwrm logs

# Logs for a specific app
zwrm logs --app my-api

# Logs for a specific machine
zwrm logs --machine <machine-id>

# Stream logs in real-time (WebSocket)
zwrm logs --follow

# Last 500 lines
zwrm logs --lines 500
```

## Filtering

```bash
# Search for specific text
zwrm logs --app my-api --search "error"

# Logs since a specific time
zwrm logs --app my-api --since "2026-04-01T00:00:00Z"

# Combine: stream + filter
zwrm logs --follow --app my-api --search "panic"
```

## Tips

- **`--follow` uses WebSocket** for real-time streaming. Press Ctrl+C to stop.
- **App-level logs** interleave output from all machines in the app.
- **Short machine IDs** work — you don't need the full UUID.
- Logs come from local files (`/var/lib/zwrm/logs/`), remote agents via gRPC, or Loki if configured.
- Internal Firecracker health check noise is auto-filtered.
- Log lines are timestamped when possible (RFC3339, syslog, and Postgres formats are recognized).

## See also

- [zwrm-status](../zwrm-status/SKILL.md) — check machine status alongside logs
- [zwrm-deploy](../zwrm-deploy/SKILL.md) — redeploy if you spot issues in logs
