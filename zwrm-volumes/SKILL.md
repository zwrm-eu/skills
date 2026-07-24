---
name: zwrm-volumes
description: |
  Persistent volumes for ZWRM microVMs. Use this skill when the user wants persistent storage for an app, needs to create, resize, or destroy a volume, or wants volume snapshots and restores. Triggers on "volume", "persistent storage", "disk", "mount", "snapshot", "restore volume", or "resize disk".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm volumes

Persistent volumes that survive VM restarts and redeploys, with on-demand and scheduled snapshots.

## Quick start

```bash
zwrm volumes create data --size 10240 --mount /data   # size in MB; updates zwrm.toml unless --no-sync
zwrm volumes list
zwrm volumes resize data --size 20480                 # grow only; app must be stopped
zwrm volumes destroy data                             # -f to skip confirmation
```

Volumes can also be declared in `zwrm.toml` (`[[volumes]]` with `name`, `size_mb`, `mount_point`) and are created on deploy.

## Snapshots

```bash
zwrm volumes snapshot create data          # manual snapshot
zwrm volumes snapshot list data
zwrm volumes snapshot enable data          # automatic scheduled snapshots
zwrm volumes restore data                  # latest snapshot; --snapshot <id> for a specific one
```

## Command reference

```text
zwrm volumes — Manage persistent volumes

zwrm volumes create <name> — Create a new volume
      --app string      Application name
      --app-id string   Application ID
      --host string     Agent host to create the volume on (default: scheduler picks one)
      --mount string    Mount point inside VM (default: /data) (default "/data")
      --no-sync         Don't update zwrm.toml
      --region string   Constrain scheduler placement to this region (ignored if --host is set)
      --size int        Volume size in MB (default: 1024) (default 1024)

zwrm volumes destroy <name> — Delete a volume
      --app string      Application name
      --app-id string   Application ID
  -f, --force           Skip confirmation prompt
      --no-sync         Don't update zwrm.toml

zwrm volumes limits — Show volume limits

zwrm volumes list — List volumes
      --app string      Application name
      --app-id string   Application ID

zwrm volumes resize <name> — Resize a volume (app must be stopped)
      --app string      Application name
      --app-id string   Application ID
      --size int        New size in MB (must be larger than current)

zwrm volumes restore <volume-name> — Restore a volume from a snapshot
      --app string        Application name
      --app-id string     Application ID
      --snapshot string   Specific snapshot ID (default: latest)

zwrm volumes snapshot — Manage volume snapshots

zwrm volumes snapshot create <volume-name> — Create a snapshot of a volume
      --app string      Application name
      --app-id string   Application ID

zwrm volumes snapshot delete <snapshot-id> — Delete a snapshot

zwrm volumes snapshot disable <volume-name> — Disable automatic snapshots for a volume
      --app string      Application name
      --app-id string   Application ID

zwrm volumes snapshot enable <volume-name> — Enable automatic snapshots for a volume
      --app string      Application name
      --app-id string   Application ID

zwrm volumes snapshot list <volume-name> — List snapshots for a volume
      --app string      Application name
      --app-id string   Application ID
```

## Tips

- **Resize requires the app stopped** and only grows — plan initial sizes accordingly.
- **Restore overwrites the volume's current contents** — snapshot first if in doubt, and confirm with the user.
- A volume pins its machine to the host holding the data; `--host`/`--region` on create control placement.
- Postgres data management belongs to [zwrm-postgres](../zwrm-postgres/SKILL.md) (its backups, not raw volume snapshots).

## See also

- [zwrm-deploy](../zwrm-deploy/SKILL.md) — declare volumes in zwrm.toml
- [zwrm-status](../zwrm-status/SKILL.md) — see which machines mount what
