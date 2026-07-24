---
name: zwrm-sandbox
description: |
  Create and manage ephemeral or persistent sandbox microVMs for code execution. Use this skill when the user wants to run code in an isolated VM, create a disposable environment, execute commands in a sandbox, upload or download files from a sandbox, restrict a workload's network egress, or needs a temporary compute environment. Triggers on "sandbox", "run this in a VM", "execute in isolation", "ephemeral environment", "disposable VM", "spin up a sandbox", "run code safely", or "isolated execution".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm sandbox

Disposable or persistent microVMs for code execution. Ephemeral sandboxes auto-destroy at their timeout; persistent ones suspend when idle (Firecracker snapshot) and wake on demand.

## Quick start

```bash
# One-shot: create → run → destroy
zwrm sandbox run -t python -- python3 -c "print('hello')"

# Ephemeral sandbox you interact with
zwrm sandbox create -t python --timeout 10m
zwrm sandbox exec <id> -- python3 script.py
zwrm sandbox upload <id> ./script.py /root/script.py
zwrm sandbox download <id> /root/output.json ./output.json

# Persistent sandbox (suspends when idle, wakes on use)
zwrm sandbox create -t base --persistent --idle-timeout 10m
```

Built-in templates: `base` (Ubuntu + dev tools), `python`, `node`. Custom ones via `zwrm templates create` (own Dockerfile, default env, egress policy, disk quota).

## Lifecycle

```bash
zwrm sandbox list --status running
zwrm sandbox status <id>
zwrm sandbox keepalive <id> --timeout 15m   # extend before the TTL hits
zwrm sandbox wake <id>                      # resume a suspended persistent sandbox
zwrm sandbox destroy <id>
```

## Egress control

For untrusted code, deny by default and allowlist what's needed:

```bash
zwrm sandbox create -t python \
  --egress-mode deny_all \
  --egress-allow-cidr 140.82.112.0/20 \
  --egress-allow-port 443
```

Templates can bake in a default egress policy so every sandbox created from them inherits it.

## Command reference

```text
zwrm sandbox — Manage sandboxes

zwrm sandbox create — Create a new sandbox
      --bandwidth-mbps int              Override per-drive bandwidth cap (MiB/s)
      --burst-mbps int                  Override one-time bandwidth burst (MiB)
      --burst-ops int                   Override one-time ops burst
      --disk-size-mb int                Override the template's writable disk quota (MiB)
      --egress-allow-cidr stringArray   Allow outbound traffic to this CIDR (can be repeated; implies --egress-mode deny_all)
      --egress-allow-port stringArray   Restrict allow_cidrs to this destination TCP port (can be repeated)
      --egress-deny-cidr stringArray    Explicitly deny outbound traffic to this CIDR (can be repeated)
      --egress-mode string              Egress firewall mode: 'allow_all' or 'deny_all' (default: cluster setting)
  -e, --env stringArray                 Environment variable in KEY=VALUE format (can be repeated)
      --idle-timeout string             Idle timeout before auto-suspend (persistent mode only) (default "10m")
      --ops-per-sec int                 Override per-drive IOPS cap
      --persistent                      Create a persistent sandbox (auto-suspends when idle)
  -s, --size string                     VM size preset (default "shared-cpu-1x")
  -t, --template string                 Template name or ID (required)
      --timeout string                  Sandbox lifetime duration (e.g. 5m, 1h) (default "5m")

zwrm sandbox destroy <id> — Destroy a sandbox
  -f, --force   Skip confirmation prompt

zwrm sandbox download <id> <remote-path> <local-path> — Download a file from a sandbox

zwrm sandbox exec <id> -- <command...> — Execute a command inside a sandbox
      --timeout string   Command timeout (e.g. '30s', '2m') (default "30s")
      --workdir string   Working directory inside the sandbox

zwrm sandbox keepalive <id> — Extend a sandbox's lifetime
      --timeout string   New timeout duration (e.g. '10m', '1h'); defaults to sandbox's current timeout

zwrm sandbox list — List sandboxes
      --status string   Filter by status (running, creating, suspended, destroyed)

zwrm sandbox run -- <command...> — Create a sandbox, run a command, and destroy it
      --egress-allow-cidr stringArray   Allow outbound traffic to this CIDR (can be repeated; implies --egress-mode deny_all)
      --egress-allow-port stringArray   Restrict allow_cidrs to this destination TCP port (can be repeated)
      --egress-deny-cidr stringArray    Explicitly deny outbound traffic to this CIDR (can be repeated)
      --egress-mode string              Egress firewall mode: 'allow_all' or 'deny_all' (default: cluster setting)
  -e, --env stringArray                 Environment variable in KEY=VALUE format (can be repeated)
      --exec-timeout string             Command execution timeout (default "30s")
  -s, --size string                     VM size preset (default "shared-cpu-1x")
  -t, --template string                 Template name or ID (required)
      --timeout string                  Sandbox lifetime duration (default "5m")
      --wait-timeout string             Max time to wait for sandbox readiness (default "2m")
      --workdir string                  Working directory inside the sandbox

zwrm sandbox status <id> — Show sandbox details

zwrm sandbox upload <id> <local-path> <remote-path> — Upload a file to a sandbox

zwrm sandbox wake <id> — Wake a suspended sandbox

zwrm templates — Manage agent templates

zwrm templates create <name> — Create a new agent template
      --bandwidth-mbps int              Default per-drive bandwidth cap (MiB/s)
      --burst-mbps int                  Default one-time bandwidth burst (MiB)
      --burst-ops int                   Default one-time ops burst
      --description string              Human-readable description of the template
      --disk-size-mb int                Writable disk quota for sandboxes from this template (MiB)
      --dockerfile string               Path to the Dockerfile for the template
      --egress-allow-cidr stringArray   Default allowed outbound CIDR (can be repeated; implies --egress-mode deny_all)
      --egress-allow-port stringArray   Default destination TCP port restriction for allow_cidrs (can be repeated)
      --egress-deny-cidr stringArray    Default denied outbound CIDR (can be repeated)
      --egress-mode string              Default egress firewall mode for sandboxes from this template: 'allow_all' or 'deny_all'
  -e, --env stringArray                 Default environment variable for sandboxes from this template, in KEY=VALUE format (can be repeated)
      --ops-per-sec int                 Default per-drive IOPS cap

zwrm templates delete <name> — Delete a template
  -f, --force   Skip confirmation prompt

zwrm templates list — List available agent templates

zwrm templates rebuild <name> — Rebuild a template image

zwrm templates update-quota <name-or-id> — Update a template's disk size and/or drive rate limit
      --bandwidth-mbps int   New per-drive bandwidth cap (MiB/s)
      --burst-mbps int       New one-time bandwidth burst (MiB)
      --burst-ops int        New one-time ops burst
      --clear-rate-limit     Drop the per-template rate limit (fall back to preset default)
      --disk-size-mb int     New writable disk quota (MiB)
      --ops-per-sec int      New per-drive IOPS cap
```

## Tips

- **`sandbox run` is the fastest path** for one-off commands — no cleanup to remember.
- **Ephemeral sandboxes hard-stop at the timeout.** Extend with `keepalive` *before* it expires.
- **Persistent sandboxes** suspend via Firecracker memory snapshots; wake is sub-second-to-seconds, and any exec/upload also wakes them.
- CoW overlays make creation cheap — the base image isn't copied, so short-lived sandboxes are the norm, not a waste.
- Size presets: `shared-cpu-1x` (default) through `performance-8x` via `-s`.

## See also

- [zwrm-agent](../zwrm-agent/SKILL.md) — long-lived coding agents with persistent volumes
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — inject env via `-e` only for non-secrets; use secrets for keys
