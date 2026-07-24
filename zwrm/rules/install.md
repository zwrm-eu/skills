---
name: zwrm-installation
description: |
  Install and authenticate the ZWRM CLI.
---

# ZWRM CLI installation

## Install

```bash
# Linux / macOS (installs to /usr/local/bin or ~/.local/bin)
curl -fsSL https://releases.zwrm.eu/zwrm/install.sh | bash

# macOS via Homebrew
brew tap zwrm-eu/tap
brew install zwrm

# Pin a version
curl -fsSL https://releases.zwrm.eu/zwrm/install.sh | bash -s -- --version=v1.0.0
```

## Authenticate

```bash
# Browser login flow
zwrm auth login

# Headless environments (SSH sessions, CI): use an API token
zwrm auth login --token <api-token>

# Verify
zwrm auth whoami
```

Credentials and the control-plane URL live in `~/.zwrm/config.toml`. `ZWRM_API_TOKEN` in the environment also works for one-off invocations.

## If commands fail

- Ensure the binary is on PATH: `zwrm --help`
- Not logged in / token expired: `zwrm auth login`
- Wrong control plane: pass `--api-url` or check `~/.zwrm/config.toml`
- Wrong org: pass `--org <slug>` (see `zwrm org list`)
