---
name: zwrm-postgres
description: |
  Managed Postgres databases on ZWRM microVMs. Use this skill when the user wants to create a database, connect to Postgres, link a database to an app, manage backups, restore from backup, enable WAL archiving, or do point-in-time recovery. Triggers on "create a database", "postgres", "database", "backup", "restore", "WAL", "PITR", "connect to db", or "managed database".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm postgres

Managed Postgres in Firecracker microVMs: connection pooling, read replicas, automated base backups, WAL archiving, and point-in-time recovery.

## Quick start

```bash
zwrm postgres create my-db --size small   # presets: small, medium, large (see `zwrm postgres presets`)
zwrm postgres connect my-db               # connection info (pooled); --direct for raw :5432
zwrm postgres list
```

## Link to an app

Linking opens network access: the app's VMs can then reach the database on port 5432 via its private IP. It does **not** set any env var — set `DATABASE_URL` yourself via `zwrm secrets set`, using the connection info from `zwrm postgres connect`. The app needs private networking enabled (`[network] private = true` in zwrm.toml).

```bash
zwrm postgres link my-db --app my-api
zwrm postgres links --app my-api
zwrm postgres unlink my-db --app my-api
```

## Lifecycle

```bash
zwrm postgres stop my-db
zwrm postgres start my-db
zwrm postgres scale my-db 2      # 2 read replicas
zwrm postgres destroy my-db      # -f to skip confirmation
```

## Backups and restore

```bash
zwrm postgres backup enable my-db            # WAL archiving + scheduled base backups
zwrm postgres backup create my-db            # manual base backup (--type logical for pg_dump-style)
zwrm postgres backup list my-db

# Restore into a NEW database (recommended)
zwrm postgres restore my-db --name my-db-restored
zwrm postgres restore my-db --name my-db-restored --backup <backup-id>
zwrm postgres restore my-db --name my-db-restored --pitr "2026-07-01T14:30:00Z"

# Omitting --name restores IN PLACE — destructive, confirm with the user first
```

## Command reference

```text
zwrm postgres — Manage PostgreSQL databases

zwrm postgres backup — Manage PostgreSQL backups

zwrm postgres backup create <name> — Create a backup of a database
      --type string   Backup type (base, logical) (default "base")

zwrm postgres backup delete <backup-id> — Delete a backup

zwrm postgres backup disable <name> — Disable WAL archiving for a database

zwrm postgres backup enable <name> — Enable WAL archiving for a database

zwrm postgres backup list <name> — List backups for a database

zwrm postgres connect <name> — Get connection info for a PostgreSQL database
      --direct   Return the raw Postgres connection (:5432), bypassing the pooler

zwrm postgres create <name> — Create a new PostgreSQL database
      --database string   Database name (defaults to instance name)
      --size string       Size preset (small, medium, large) (default "small")

zwrm postgres destroy <name> — Delete a PostgreSQL database
  -f, --force   Skip confirmation prompt

zwrm postgres link <database-name> — Link a database to an app
      --app string   App name to link the database to (default: from zwrm.toml)

zwrm postgres links — List database links
      --app string        App name to list linked databases for
      --database string   Database name to list linked apps for

zwrm postgres list — List PostgreSQL databases

zwrm postgres presets — List available size presets

zwrm postgres restore <name> — Restore a database from backup
      --backup string   Specific backup ID to restore from
      --name string     Name for restored database (omit for in-place restore)
      --pitr string     Point-in-time recovery target (RFC3339)
      --size string     Size preset override (small, medium, large)

zwrm postgres scale <name> <count> — Scale PostgreSQL read replicas

zwrm postgres start <name> — Start a stopped PostgreSQL database

zwrm postgres stop <name> — Stop a PostgreSQL database

zwrm postgres unlink <database-name> — Unlink a database from an app
      --app string   App name to unlink the database from (default: from zwrm.toml)
```

## Tips

- **Enable backups early** — PITR needs WAL archiving running *before* the moment you want to recover to.
- **Prefer restore-into-new-name** and cut over after verifying; in-place restore replaces the live database.
- `connect` returns the pooled endpoint by default; use `--direct` for tools that need a raw Postgres connection (e.g. some migration runners).
- Backups stream from the VM through the control plane to S3 — database VMs don't need S3 access for base backups.

## See also

- [zwrm-secrets](../zwrm-secrets/SKILL.md) — set DATABASE_URL for the app (`link` opens the network path only)
- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy apps that use the database
