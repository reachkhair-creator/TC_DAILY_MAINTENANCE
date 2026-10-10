# Teamcenter Daily Maintenance

A Windows batch script that performs the daily server-side health check and housekeeping for a **Siemens Teamcenter 2606** deployment (4-tier architecture, Windows Server).

All results are written to a dated log file so the output can be reviewed or collected by monitoring tools.

## What it does

| Step | Check | Action on failure |
|------|-------|-------------------|
| 0 | Loads `tc_profilevars.bat` | Logs CRITICAL and exits with code 1 |
| 1 | `tcserver.exe` processes in the pool | Starts `Teamcenter_Server_Manager` if it is not running |
| 2 | FSC status via `fscadmin` | Starts `Teamcenter_FSC_Service` |
| 3 | Purges `*.syslog`, `*.log`, `*.jnl` older than `RETENTION_DAYS` from `TC_ROOT\logs` | Warns if the directory is missing |
| 4 | FSC cache drive (C:) utilisation | Warns at 80% or more |
| 5 | Database connectivity through `sqlplus` | Logs the error output |
| 6 | Dispatcher Scheduler and Dispatcher Module services | Starts a stopped service |
| 7 | Free space on the volume drive (E:) | Logged for information |
| 8 | Latest `.bak`, `.dmp` or `.zip` in the backup folder | Logs ERROR or CRITICAL if none is found |

## Repository layout

```
tc-daily-maintenance/
├── src/TC_DAILY_MAINTENANCE.bat   # the script
├── config/settings.example.txt    # reference for the variables to edit
├── docs/SCHEDULING.md             # Task Scheduler setup
├── docs/KNOWN_ISSUES.md           # review items before production use
├── CHANGELOG.md
├── LICENSE
└── .gitignore
```

## Requirements

- Windows Server with Teamcenter 2606 installed
- Oracle client with `sqlplus` on `PATH`
- PowerShell (used for disk space checks)
- An account allowed to start and stop the Teamcenter services

## Configuration

The variables are defined at the top of the script. See `config/settings.example.txt` for the full list. The values in the script are generic placeholders and must be adapted to your environment:

- `TC_ROOT`, `TC_DATA`, `FSC_HOME`: Teamcenter installation paths
- `VOLUME_DIR`, `BACKUP_DIR`, `MAINT_LOG_DIR`: storage and log locations
- `FSC_URL`, `TC_DB_CONN_STR`: host and connection details
- `TC_DB_USER`: database account used for the connectivity test
- `RETENTION_DAYS`: log retention (default 2)

**Do not commit real passwords.** The script contains `[PLACEHOLDER_PASSWORD]` in the sqlplus call. Replace it at deployment time (see `docs/KNOWN_ISSUES.md` for safer options).

## Usage

```bat
src\TC_DAILY_MAINTENANCE.bat
```

Logs are written to `%MAINT_LOG_DIR%\daily_maintenance_<date>.log`. For scheduled runs see `docs/SCHEDULING.md`.

## Exit codes

- `0`: routine completed (individual check failures are recorded in the log)
- `1`: critical environment error (`tc_profilevars.bat` not found)

## Disclaimer

Test in a non-production environment first. The script deletes log files and starts services.

## License

MIT, see `LICENSE`.
