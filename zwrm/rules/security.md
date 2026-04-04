---
name: zwrm-security
description: |
  Security guidelines for working with ZWRM microVMs, secrets, and infrastructure.
---

# Security Guidelines

## Secrets

- **Never echo or log secret values.** Use `zwrm secrets list` which only shows metadata, never values.
- **Never hardcode secrets** in zwrm.toml, Dockerfiles, or source code. Always use `zwrm secrets set`.
- **Use `--from-file`** for bulk secret loading from .env files rather than setting secrets one-by-one in shell history.

## Sandbox Execution

- Sandboxes run untrusted code in isolated Firecracker microVMs. They are ephemeral by default.
- **Do not expose sandbox IPs** outside the host network without explicit firewall rules.
- **Set appropriate timeouts** — ephemeral sandboxes auto-destroy after their TTL expires.

## Agent VMs

- Agent VMs have persistent volumes. Secrets set via `zwrm agent secrets set` are injected at boot.
- **Rotate secrets regularly** — use `zwrm secrets set` to update, then reconnect the agent.

## General

- The control plane API has no built-in authentication. Ensure it runs behind a reverse proxy with auth in production.
- VM images are stored world-readable at `/var/lib/zwrm/images/`. Do not store sensitive data in base images.
- All VM networking uses isolated TAP devices with /30 subnets. Cross-app traffic is blocked by default when private networking is enabled.
