# Backups and restore

A backup plan is complete when a restore has succeeded. Docker images and Compose files can be recreated; personal files, app configuration, databases, and Caddy CA state cannot be assumed recoverable.

## Inventory

For each service, record:

| Item | Record |
| --- | --- |
| Data location | Named volume or host directory |
| Database | Product and consistent export method |
| Backup frequency | Based on acceptable data loss |
| Retention | Number of local and off-host copies |
| Restore steps | Tested commands and order |
| Encryption | Key storage and recovery owner |

Do not copy a live database file and assume it is consistent. Use the database's supported dump mechanism or a coordinated snapshot. Encrypt an off-host copy and keep the recovery key outside the server being backed up.

## Restore drill

1. Choose one noncritical service and a spare host or isolated directory.
2. Restore its configuration, database, and files from a selected backup.
3. Start the service without replacing production data.
4. Verify application login and a representative file or record.
5. Record duration, missing dependencies, and the last recoverable point.

Repeat after major upgrades or storage changes. Alert when a backup is missing or older than its planned interval.
