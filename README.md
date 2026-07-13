# Oracle Notes (Seeomkus)

A collection of practical notes, guides, and scripts for Oracle Database administration — covering day-to-day DBA tasks, troubleshooting, and automation across multiple Oracle versions and operating systems.

---

## About This Repository

This repository is a personal reference library for Oracle Database administration tasks. Each document is written in structured English, designed to be self-contained, and includes:

- Step-by-step procedures with commands ready to copy and run
- Coverage for both **Oracle Linux** and **Windows Server**
- Version compatibility notes from **Oracle Database 11g through 26 AI**
- Mermaid diagrams where process flow benefits from visual explanation
- Troubleshooting sections for common issues

All sensitive information (schema names, hostnames, IP addresses, SIDs) has been replaced with generic placeholders suitable for public documentation.

---

## Platform and Version Coverage

| Category | Coverage |
|----------|----------|
| Oracle Database Versions | 11g, 12c, 18c, 19c, 21c, 23ai, 26 AI |
| Oracle Editions | Standard Edition 2 (SE2), Enterprise Edition (EE) |
| Operating Systems | Oracle Linux 6.x – 9.x, Windows Server 2012 – 2025 |
| Architecture | Single Instance, CDB/PDB Multitenant (12c+) |

---

## Topics Covered

| Category | Documents |
|----------|-----------|
| Backup & Restore | Data Pump Export, Data Pump Import, RMAN Archive Log |
| Storage Management | Move Tablespace Objects |
| Maintenance & Cleanup | Audit/Trace/Alert Cleanup, RMAN Archive Log Deletion |
| Security & User Management | Kill Sessions & Lock Users, Password Never Expires |
| Database Administration | Rename Pluggable Database, Shutdown Immediate Hang |
| Development & Testing | Generate Dummy Data |

---

## Notes

| No | File | Description |
|---|---|---|
| 1 | [cleanup-audit-trace-alert.md](cleanup-audit-trace-alert.md) | Guide to clean up Oracle diagnostic files (audit `.aud`, trace `.trc`, alert log, and incident files) using ADRCI and OS commands on Oracle Linux and Windows Server — covers ADRCI purge types, retention policies, combined shell/batch scripts, cron/Task Scheduler scheduling, unified audit cleanup (12c+), and troubleshooting (11g–26 AI). |
| 2 | [datapump-export.md](datapump-export.md) | Complete guide to backup using Oracle Data Pump Export (`expdp`) on Windows and Linux — covers directory setup, all parameters, 12c/19c/21c features, usage examples with sample output, version-by-version script differences (11g–26 AI), and troubleshooting. |
| 3 | [datapump-import.md](datapump-import.md) | Complete guide to restore using Oracle Data Pump Import (`impdp`) on Windows and Linux — covers directory setup, all parameters, remapping options, 12c/19c/21c features, usage examples with sample output, version-by-version script differences (11g–26 AI), and troubleshooting. |
| 4 | [generate-dummy-data.md](generate-dummy-data.md) | Guide to generate large volumes of dummy data in Oracle using `CONNECT BY LEVEL` and `DBMS_RANDOM` — covers 7 table variants from 50K to 1M rows, with verification, key technique explanations, and version compatibility (11g–26 AI, no script changes). |
| 5 | [kill-sessions-and-lock-users.md](kill-sessions-and-lock-users.md) | Guide to kill active Oracle sessions and lock user accounts — includes dry-run SQL script, execution SQL script, Windows `.bat` automation, Oracle Linux `.sh` equivalent, cron/Task Scheduler setup, and version compatibility notes (11g–26 AI). |
| 6 | [move-tablespace-objects.md](move-tablespace-objects.md) | Guide to move all tablespace objects (tables, partitions, sub-partitions, LOBs, and indexes) from one tablespace to another using a generate-then-execute DDL pattern (8i and above, `ONLINE` move requires Enterprise Edition 12.2+). |
| 7 | [oracle-no-expire-password.md](oracle-no-expire-password.md) | Complete guide to change the `SYSTEM` user profile so the password never expires — covers status check, creating a new profile, final verification, and version compatibility (11g–26 AI, no script changes). |
| 8 | [rename-pluggable-database.md](rename-pluggable-database.md) | Step-by-step guide to rename a Pluggable Database (PDB) in Oracle 19c — covers closing the PDB, opening in restricted mode, renaming via `RENAME GLOBAL_NAME TO`, verification, and version compatibility (12c–26 AI only; not applicable to 11g). |
| 9 | [rman-delete-archive-log.md](rman-delete-archive-log.md) | Guide to safely delete archive log files older than 1 month via RMAN — covers crosscheck, list preview, delete options, verification with sample output, and version compatibility (11g–26 AI, no script changes). |
| 10 | [shutdown-immediate-hang.md](shutdown-immediate-hang.md) | Step-by-step guide to resolve `SHUTDOWN IMMEDIATE` hang on Oracle Database — covers diagnosis, escalation to `SHUTDOWN ABORT`, OS-level force kill on Oracle Linux and Windows Server, IPC cleanup, crash recovery, and prevention tips, with Mermaid flow diagrams. |

---

## Version Compatibility by Document

Each document now includes its own **Version Compatibility** section. This table summarizes which documents share the exact same script across all supported versions, and which ones require version-specific script variants.

| Document | Supported Versions | Script Differs by Version? |
|---|---|---|
| [cleanup-audit-trace-alert.md](cleanup-audit-trace-alert.md) | 11g – 26 AI | Yes — Unified Audit cleanup (`DBMS_AUDIT_MGMT`) is 12c+ only |
| [datapump-export.md](datapump-export.md) | 11g – 26 AI (core parameters) | Yes — `COMPRESSION_ALGORITHM`/`CHECKSUM` require 19c+, blockchain table support requires 21c+ |
| [datapump-import.md](datapump-import.md) | 11g – 26 AI (core parameters) | Yes — `PARTITION_OPTIONS`/PDB import require 12c+, `DISABLE_ARCHIVE_LOGGING` requires 19c+ |
| [generate-dummy-data.md](generate-dummy-data.md) | 11g – 26 AI | No — identical script on every version |
| [kill-sessions-and-lock-users.md](kill-sessions-and-lock-users.md) | 11g – 26 AI | Yes — use `DBMS_LOCK.SLEEP` on 11g/12c, `DBMS_SESSION.SLEEP` on 18c+ |
| [move-tablespace-objects.md](move-tablespace-objects.md) | 8i and above | Yes — `ONLINE` move clause requires Enterprise Edition 12.2+ |
| [oracle-no-expire-password.md](oracle-no-expire-password.md) | 11g – 26 AI | No — identical script on every version |
| [rename-pluggable-database.md](rename-pluggable-database.md) | **12c – 26 AI only** (not applicable to 11g) | No — identical script across all supported (12c+) versions |
| [rman-delete-archive-log.md](rman-delete-archive-log.md) | 11g – 26 AI | No — identical script on every version |
| [shutdown-immediate-hang.md](shutdown-immediate-hang.md) | 11g – 26 AI | No — identical procedure on every version |

---

## How to Use

1. Browse the **Notes** table above and click the document that matches your task.
2. Each document starts with an explanation of the concept before the commands — read it first to understand the context.
3. Replace placeholder values with your actual environment details before running any command:

| Placeholder | Replace With |
|-------------|-------------|
| `ORCL` | Your Oracle SID or instance name |
| `YOUR_SCHEMA` | Your schema or user name |
| `SOURCE_TABLESPACE` | Your source tablespace name |
| `TARGET_DATA_TBS` | Your target data tablespace name |
| `TARGET_INDEX_TBS` | Your target index tablespace name |
| `OLD_PDB_NAME` | Your current PDB name |
| `NEW_PDB_NAME` | Your desired new PDB name |
| `APP_USER` | Your application user account name |
| `C:\app\oracle\` | Your actual Oracle Base path on Windows |
| `/u01/app/oracle/` | Your actual Oracle Base path on Linux |

4. For scripts that support dry-run mode, **always run the preview step first** before executing destructive or irreversible commands.

---

## Conventions Used in This Repository

| Convention | Meaning |
|-----------|---------|
| `-- Check ...` SQL comments | Read-only diagnostic query — safe to run anytime |
| `-- Execute ...` SQL comments | Modifies data or state — confirm before running |
| `[DRY RUN]` label | Script output only, no actual changes made |
| `[EXECUTE]` label | Script performs real operations |
| `ORCL` | Generic placeholder for Oracle SID |
| `YOUR_SCHEMA` | Generic placeholder for schema/user name |

---

## Prerequisites

Most scripts and commands in this repository require the following:

- Access to a user with **SYSDBA** privilege (for instance-level operations)
- The `oracle` OS user (Linux) or Administrator privilege (Windows)
- Environment variables set correctly:
  - Linux: `ORACLE_HOME`, `ORACLE_BASE`, `ORACLE_SID`, `PATH` including `$ORACLE_HOME/bin`
  - Windows: `ORACLE_HOME`, `ORACLE_SID` set, `%ORACLE_HOME%\bin` in system `PATH`
- For automation scripts: write permission to the log directory

---

## Quick Environment Check

### Oracle Linux

```bash
echo "ORACLE_HOME : $ORACLE_HOME"
echo "ORACLE_BASE : $ORACLE_BASE"
echo "ORACLE_SID  : $ORACLE_SID"
sqlplus / as sysdba <<EOF
SELECT instance_name, version, status FROM v\$instance;
EXIT
EOF
```

### Windows Server

```cmd
echo ORACLE_HOME: %ORACLE_HOME%
echo ORACLE_SID : %ORACLE_SID%
echo STARTUP | sqlplus -S / as sysdba
```

Or PowerShell:

```powershell
Write-Host "ORACLE_HOME : $env:ORACLE_HOME"
Write-Host "ORACLE_SID  : $env:ORACLE_SID"
```

---

## License

This repository is for personal reference and educational purposes.
All examples use generic placeholder values — no production credentials, hostnames, or schema names are included.
