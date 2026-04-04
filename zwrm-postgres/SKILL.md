---
name: zwrm-postgres
description: |
  Managed Postgres databases on ZWRM microVMs. Use this skill when the user wants to create a database, connect to Postgres, manage backups, restore from backup, enable WAL archiving, or do point-in-time recovery. Triggers on "create a database", "postgres", "database", "backup", "restore", "WAL", "PITR", "connect to db", or "managed database".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm postgres

Managed Postgres 16 databases running in Firecracker microVMs. Includes automated backups, WAL archiving, and point-in-time recovery.

## When to use

- You need a Postgres database for your app
- You want automated backups with S3 storage
- You need point-in-time recovery

## Quick start

```bash
# Create a database
zwrm postgres create my-db --size shared-cpu-1x

# Connect via psql
zwrm postgres connect my-db

# List databases
zwrm postgres list

# Get database details
zwrm postgres status my-db
```

## Lifecycle

```bash
# Stop a running database
zwrm postgres stop my-db

# Start a stopped database
zwrm postgres start my-db

# Scale replicas
zwrm postgres scale my-db --replicas 2

# Destroy database
zwrm postgres destroy my-db
```

## Backups

```bash
# Enable automated backups (WAL archiving + daily base backups)
zwrm postgres backup enable my-db

# Take a manual backup
zwrm postgres backup create my-db

# List backups
zwrm postgres backup list my-db

# Delete a backup
zwrm postgres backup delete <backup-id>

# Disable automated backups
zwrm postgres backup disable my-db
```

## Restore

```bash
# Restore from latest backup (creates new database)
zwrm postgres restore my-db --name my-db-restored

# Restore from specific backup
zwrm postgres restore my-db --name my-db-restored --backup <backup-id>

# Point-in-time recovery
zwrm postgres restore my-db --name my-db-restored --pitr "2026-03-10T14:30:00Z"

# Check restore status
zwrm postgres restore status <restore-id>
```

## Tips

- **Enable backups early.** `backup enable` turns on WAL archiving + scheduled base backups in one command.
- **PITR requires WAL archiving.** Enable backups before you need point-in-time recovery.
- Backups are streamed via SSH from the VM to the control plane, then uploaded to S3. VMs don't need direct internet access.
- Base backups run daily by default. Retention is configurable (default 7 days).
- **Use `connect`** for interactive psql sessions. It resolves the VM's IP automatically.

## See also

- [zwrm-secrets](../zwrm-secrets/SKILL.md) — set DATABASE_URL for your app
- [zwrm-deploy](../zwrm-deploy/SKILL.md) — deploy apps that connect to this database
- [zwrm-logs](../zwrm-logs/SKILL.md) — view Postgres VM logs
