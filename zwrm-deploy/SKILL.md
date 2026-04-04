---
name: zwrm-deploy
description: |
  Deploy applications to ZWRM microVMs. Use this skill when the user wants to deploy an app, initialize a project config, build and ship code, scale machines up or down, or tear down an application. Triggers on "deploy", "ship", "launch", "push to production", "scale up", "scale down", "destroy the app", "tear down", "initialize project", or "create zwrm.toml".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm deploy

Build Docker images and deploy them as Firecracker microVMs. Deployments are asynchronous — the CLI polls for status while background workers build and launch.

## When to use

- You have a Dockerfile and want to deploy it as a microVM
- You need to scale an existing app up or down
- You want to initialize a new project with `zwrm.toml`

## Quick start

```bash
# Initialize a new project (generates zwrm.toml)
zwrm init

# Deploy the app (builds Docker image, converts to ext4, launches VMs)
zwrm deploy

# Force rebuild (skip cache)
zwrm deploy --force-build

# Deploy from a specific directory
zwrm deploy --context ./my-app
```

## Scaling

```bash
# Scale to 3 machines
zwrm scale 3

# Scale a specific app
zwrm scale 5 --app my-api
```

## Teardown

```bash
# Destroy all machines for current app
zwrm destroy --all

# Destroy with force (skip confirmation)
zwrm destroy --all --force

# Destroy a single machine
zwrm destroy <machine-id>
```

## Configuration (zwrm.toml)

```toml
app = "my-api"
primary_region = "eu"

[build]
  dockerfile = "Dockerfile"

[vm]
  size = "shared-cpu-1x"    # or performance-4x, etc.
  count = 2
  internal_port = 8080

[env]
  NODE_ENV = "production"

[[volumes]]
  name = "data"
  size_mb = 10240
  mount_point = "/data"
```

## Tips

- **Deployments are async.** The CLI auto-polls status. If you disconnect, check with `zwrm status`.
- **Build caching** uses content hashing. Only rebuilds when Dockerfile or context changes.
- **Use `--force-build`** if you need to bypass the cache (e.g., after base image updates).
- VM presets: `shared-cpu-1x`, `shared-cpu-2x`, `shared-cpu-4x`, `performance-4x`.

## See also

- [zwrm-status](../zwrm-status/SKILL.md) — check deployment and machine status
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — set environment variables before deploying
- [zwrm-logs](../zwrm-logs/SKILL.md) — view VM output after deployment
