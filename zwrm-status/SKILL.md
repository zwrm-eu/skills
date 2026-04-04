---
name: zwrm-status
description: |
  Check the status of apps, machines, deployments, and the ZWRM system. Use this skill when the user asks about the state of their infrastructure, wants to see what's running, check deployment progress, view machine health, or get system statistics. Triggers on "status", "what's running", "check my app", "is it deployed", "system health", "how many machines", or "show me the state".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm status

Check the status of apps, machines, deployments, and the overall system.

## When to use

- You want to see what's currently running
- You need to check if a deployment succeeded
- You want machine IPs, resource allocation, or uptime

## Quick start

```bash
# Status of current app (reads zwrm.toml)
zwrm status

# Status of a specific app
zwrm status --app my-api

# System-wide status
zwrm status --system
```

## What it shows

- **App info**: name, ID, machine count
- **Machines**: ID, status, IP, size, region, uptime
- **Deployments**: latest deployment status, image, timestamp

## Tips

- Run `zwrm status` after `zwrm deploy` to confirm machines are running.
- Machine statuses: `created`, `starting`, `running`, `stopping`, `stopped`, `destroyed`.
- Use `--app` to check apps other than the one in your current `zwrm.toml`.
- For real-time output, combine with `zwrm logs --follow`.

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy or scale apps
- [zwrm-logs](../zwrm-logs/SKILL.md) — view VM console output
