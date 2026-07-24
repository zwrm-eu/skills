---
name: zwrm-status
description: |
  Check the status of apps, machines, and deployments on ZWRM. Use this skill when the user asks about the state of their infrastructure, wants to see what's running, check deployment progress, view machine health, or find monitoring dashboards. Triggers on "status", "what's running", "check my app", "is it deployed", "how many machines", "show me the state", or "dashboard".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm status

Check the status of apps, machines, and deployments.

## Quick start

```bash
zwrm status                 # current app (reads zwrm.toml)
zwrm status --app my-api    # a specific app
zwrm dashboard              # monitoring dashboard URLs (--open to launch)
```

Shows app info (name, ID, machine count), machines (ID, status, IP, size, region, uptime), and the latest deployment.

## Command reference

```text
zwrm status — Show application and machine status
      --app string      Application name
      --app-id string   Application ID

zwrm dashboard — Show monitoring dashboard URLs
      --open   Open the system overview dashboard in a browser
```

## Tips

- Run `zwrm status` after `zwrm deploy` to confirm machines are running.
- Machine statuses: `created`, `starting`, `running`, `stopping`, `stopped`, `destroyed`. A `stopped` machine under `[autostop]` is normal — it wakes on the next request.
- For live output, pair with `zwrm logs --follow`.

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy or scale apps
- [zwrm-logs](../zwrm-logs/SKILL.md) — view VM console output
