---
name: zwrm-security
description: |
  Security guidelines for working with ZWRM microVMs, secrets, and tokens.
---

# Security guidelines

## Secrets

- **Never echo or log secret values.** `zwrm secrets list` shows only names and metadata, never values — keep it that way in your own output too.
- **Never hardcode secrets** in `zwrm.toml`, Dockerfiles, or source code. Use `zwrm secrets set` (or `zwrm org secrets` / `zwrm agent secrets`).
- **Prefer `--from-file`** for bulk loading from `.env` files over setting values one-by-one on the command line (keeps values out of shell history).
- Secrets are encrypted at rest in the control plane and injected into VMs at boot via a metadata service.

## Tokens

- For CI and automation, mint **capability-scoped tokens**: `zwrm auth tokens create --name ci --scopes apps:read,deploy,logs:read --expiry 8760h`. An empty scope list creates an unrestricted token — avoid that outside interactive use.
- Revoke unused tokens: `zwrm auth tokens list` / `zwrm auth tokens revoke <id>`.

## Sandboxes

- Sandboxes run untrusted code in isolated Firecracker microVMs; ephemeral ones auto-destroy at their timeout.
- For untrusted workloads, lock down egress: `--egress-mode deny_all` plus explicit `--egress-allow-cidr` (optionally `--egress-allow-port`) allowlists. Templates can set egress defaults for every sandbox created from them.
- Set the shortest timeout that fits the job; use `keepalive` to extend rather than starting long.

## Agent VMs

- Agent secrets (`zwrm agent secrets set`) are injected at boot; rotate by setting the new value and restarting the agent VM.
- Constrain what an agent's boot token may do with `zwrm agent update --allowed-scopes …`, and cap spend with `zwrm agent budget`.
