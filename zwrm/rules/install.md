---
name: zwrm-installation
description: |
  Install and configure the ZWRM CLI for microVM orchestration.
---

# ZWRM CLI Installation

## Quick Setup

Download the latest binary for your platform:

```bash
# macOS (Apple Silicon)
curl -fsSL https://github.com/zwrm-eu/cli/releases/latest/download/zwrm-darwin-arm64 -o /usr/local/bin/zwrm
chmod +x /usr/local/bin/zwrm

# macOS (Intel)
curl -fsSL https://github.com/zwrm-eu/cli/releases/latest/download/zwrm-darwin-amd64 -o /usr/local/bin/zwrm
chmod +x /usr/local/bin/zwrm

# Linux (x86_64)
curl -fsSL https://github.com/zwrm-eu/cli/releases/latest/download/zwrm-linux-amd64 -o /usr/local/bin/zwrm
chmod +x /usr/local/bin/zwrm
```

## Configuration

```bash
# Set the control plane endpoint
zwrm config set api_url https://your-control-plane.example.com

# Verify connectivity
zwrm status
```

Configuration is stored at `~/.zwrm/config.toml`.

## Verify

```bash
zwrm --help
zwrm status
```

## If commands fail

- Ensure the binary is in your PATH
- Check that the control plane URL is configured: `cat ~/.zwrm/config.toml`
- Verify the control plane is reachable: `curl -s https://your-control-plane/v1/health`
