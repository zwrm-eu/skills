---
name: zwrm-deploy
description: |
  Deploy applications to ZWRM microVMs. Use this skill when the user wants to deploy an app, initialize a project config, build and ship code, scale machines up or down, expose an app on a hostname, move an app between orgs, or tear down an application. Triggers on "deploy", "ship", "launch", "push to production", "scale up", "scale down", "destroy the app", "tear down", "custom domain", "initialize project", or "create zwrm.toml".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm deploy

Build a Docker image and run it as Firecracker microVMs. Deployments are asynchronous — the CLI polls status while background workers build the image, convert it to ext4, and launch VMs behind the platform's reverse proxy.

## Quick start

```bash
# In the project directory (next to the Dockerfile)
zwrm init --name my-app     # writes zwrm.toml (in a git repo, also a GitHub Actions deploy workflow unless --no-workflow)
zwrm deploy                 # build + launch; streams progress, prints the app URL
zwrm status                 # confirm machines are running
```

Useful variants:

```bash
zwrm deploy --force-build            # bypass the build cache
zwrm deploy --context ./services/api # build from another directory
zwrm deploy --replicas 3             # deploy N machines at once
zwrm scale 3                         # scale an already-deployed app
zwrm destroy --machine <id>          # remove one machine
zwrm destroy --all                   # remove app + all machines (add -f to skip confirm)
```

## Configuration (zwrm.toml)

```toml
[app]
  name = "my-api"

[build]
  dockerfile = "Dockerfile"      # or: image = "ghcr.io/acme/api:latest"

[vm]
  size = "shared-cpu-1x"         # shared-cpu-1x/2x/4x, performance-1x/2x/4x/8x

[[services]]
  internal_port = 8080           # port your app listens on; the proxy routes to it

[env]
  NODE_ENV = "production"        # non-secret env; secrets go through `zwrm secrets`

[[volumes]]
  name = "data"
  size_mb = 10240
  mount_point = "/data"

[autostop]
  auto_stop = true               # scale-to-zero when idle
  idle_timeout = "5m"
  min_machines_running = 0       # 0 allows full scale-to-zero; auto-starts on request
```

## Routes (hostnames)

Apps get a default hostname via the platform proxy. Extra hostnames:

```bash
zwrm routes create --hostname api.example.com --port 8080
zwrm routes list
```

## Command reference

```text
zwrm init — Initialize a new zwrm.toml configuration
      --dockerfile string   Dockerfile path (default: Dockerfile)
      --name string         Application name
      --no-prompt           Don't prompt for input, use defaults
      --no-workflow         Don't create a GitHub Actions deploy workflow in a git repository

zwrm deploy — Deploy an application
      --app string       Application name (default: from zwrm.toml)
      --context string   Build context directory (default: current directory)
      --force-build      Force rebuild, bypass cache
      --ref string       Branch, tag, or commit for --repo (alternative to an @ref suffix; don't use both)
      --replicas int     Number of instances to deploy (default 1)
      --repo string      Deploy from a GitHub repo (github.com/owner/repo[@ref]); server-side clone, requires --app

zwrm scale <count> — Scale application to desired number of machines
      --app string      Application name
      --app-id string   Application ID

zwrm destroy — Destroy machines or entire application
      --all              Also destroy the app itself
      --app string       Application name
      --app-id string    Application ID
  -f, --force            Skip confirmation prompt
      --machine string   Destroy specific machine by ID

zwrm transfer — Move an app to another organization
      --app string      App name (defaults to zwrm.toml in the current directory)
      --app-id string   App ID (alternative to --app)
      --to-org string   Destination organization slug or ID

zwrm routes — Manage application routes

zwrm routes create — Create a route for an application
      --app string        Application name (default: from zwrm.toml)
      --hostname string   Hostname for the route (required)
      --port int          Target port (default: from app config or 8080)

zwrm routes delete <route-id> — Delete a route
      --app string   Application name (default: from zwrm.toml)
  -f, --force        Skip confirmation

zwrm routes list — List routes for an application
      --app string   Application name (default: from zwrm.toml)
```

## Tips

- **Deployments are async.** If you disconnect mid-deploy, `zwrm status` shows where it landed.
- **Build caching is content-addressed** — unchanged Dockerfile + context = no rebuild. `--force-build` bypasses it.
- **Scale-to-zero**: with `[autostop]` configured, idle machines stop and the proxy wakes them on the next request.
- **CI deploys**: `zwrm init` scaffolds a GitHub Actions workflow using the `zwrm-eu/deploy-action` action; give it a scoped token (`zwrm auth tokens create --scopes apps:read,deploy`).
- `zwrm transfer --to-org <org>` moves an app (machines, volumes, secrets) to another organization.

## See also

- [zwrm-status](../zwrm-status/SKILL.md) — check deployment and machine status
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — set secrets before deploying
- [zwrm-logs](../zwrm-logs/SKILL.md) — view VM output after deployment
- [zwrm-volumes](../zwrm-volumes/SKILL.md) — persistent storage for apps
