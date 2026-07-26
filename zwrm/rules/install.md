---
name: zwrm-installation
description: |
  Install and authenticate the ZWRM CLI.
---

# ZWRM CLI installation

## Install

`zwrm` is a single static binary. Every release publishes a `checksums.txt` alongside it, and
each method below verifies the download against it.

**Homebrew** (macOS/Linux) — the formula pins a SHA-256 per release and Homebrew checks it:

```bash
brew tap zwrm-eu/tap
brew install zwrm
```

**Install script** — downloads the binary, verifies its SHA-256 against the release's
`checksums.txt`, and aborts without installing if the checksum is missing or doesn't match:

```bash
curl -fsSL https://releases.zwrm.eu/zwrm/install.sh | bash

# Pin a version
curl -fsSL https://releases.zwrm.eu/zwrm/install.sh | bash -s -- --version=v1.0.0
```

If you'd rather read the script before running it — reasonable for anything piped to a
shell — download it first:

```bash
curl -fsSL -o zwrm-install.sh https://releases.zwrm.eu/zwrm/install.sh
less zwrm-install.sh
sh zwrm-install.sh
```

**Manual** — fetch the binary and check it yourself. Every step is chained with
`&&`, so nothing gets installed if the checksum fails:

```bash
VERSION=v1.0.0
OS=$(uname -s | tr '[:upper:]' '[:lower:]')      # linux | darwin
ARCH=$(uname -m | sed 's/x86_64/amd64/; s/aarch64/arm64/')
BASE="https://releases.zwrm.eu/zwrmd/${VERSION}"

# sha256sum on Linux, shasum on macOS
SHA=sha256sum; command -v sha256sum >/dev/null 2>&1 || SHA="shasum -a 256"

curl -fsSL -O "${BASE}/zwrm-${OS}-${ARCH}" &&
curl -fsSL -O "${BASE}/checksums.txt" &&
grep " [*]\{0,1\}zwrm-${OS}-${ARCH}$" checksums.txt | $SHA -c - &&   # must print: OK
sudo install -m 0755 "zwrm-${OS}-${ARCH}" /usr/local/bin/zwrm
```

Drop the `sudo` and use `install -m 0755 "zwrm-${OS}-${ARCH}" ~/.local/bin/zwrm`
if you'd rather not install system-wide.

The script installs to `/usr/local/bin` when run as root and `~/.local/bin` otherwise; pass
`--dir=<path>` to override. It warns if the chosen directory isn't on your `PATH`.

## Authenticate

```bash
# Browser login flow
zwrm auth login

# Headless environments (SSH sessions, CI): export a token rather than passing
# it as an argument, so it stays out of argv and shell history
export ZWRM_API_TOKEN=...   # from `zwrm auth tokens create`, or CI secret store
zwrm auth whoami            # verify

# To persist it to the config file instead of exporting it every session,
# reference the variable — don't type the token into the command
zwrm auth login --token "$ZWRM_API_TOKEN"
```

`ZWRM_API_TOKEN` is checked before the credentials file, so exporting it is enough — there's no
need to run a login command at all in CI. Otherwise credentials and the control-plane URL live
in `~/.zwrm/config.toml`.

Don't paste a token into a command for someone else to run, and don't echo one back in your
output. If a token is needed and you don't have it, ask the user to export it.

## If commands fail

- Ensure the binary is on PATH: `zwrm --help`
- Not logged in / token expired: `zwrm auth login`
- Wrong control plane: pass `--api-url` or check `~/.zwrm/config.toml`
- Wrong org: pass `--org <slug>` (see `zwrm org list`)
