# Oracle Notes (Seeomkus)

A collection of notes and guides about Oracle Database.

---

## Notes

| No | File | Description |
|---|---|---|
| 1 | [datapump-export.md](datapump-export.md) | Complete guide to backup using Oracle Data Pump Export (`expdp`) on Windows and Linux — covers directory setup, all parameters, 12c/19c/21c features, usage examples with sample output, and troubleshooting. |
| 2 | [datapump-import.md](datapump-import.md) | Complete guide to restore using Oracle Data Pump Import (`impdp`) on Windows and Linux — covers directory setup, all parameters, remapping options, 12c/19c/21c features, usage examples with sample output, and troubleshooting. |
| 3 | [generate-dummy-data.md](generate-dummy-data.md) | Guide to generate large volumes of dummy data in Oracle using `CONNECT BY LEVEL` and `DBMS_RANDOM` — covers 7 table variants from 50K to 1M rows, with verification and key technique explanations. |
| 4 | [kill-sessions-and-lock-users.md](kill-sessions-and-lock-users.md) | Guide to kill active Oracle sessions and lock user accounts — includes dry-run SQL script, execution SQL script, Windows `.bat` automation, Oracle Linux `.sh` equivalent, cron/Task Scheduler setup, and version compatibility notes (11g–26 AI). |
| 5 | [move-tablespace-objects.md](move-tablespace-objects.md) | Guide to move all tablespace objects (tables, partitions, sub-partitions, LOBs, and indexes) from one tablespace to another using a generate-then-execute DDL pattern. |
| 6 | [oracle-no-expire-password.md](oracle-no-expire-password.md) | Complete guide to change the `SYSTEM` user profile so the password never expires — covers status check, creating a new profile, and final verification. |
| 7 | [rename-pluggable-database.md](rename-pluggable-database.md) | Step-by-step guide to rename a Pluggable Database (PDB) in Oracle 19c — covers closing the PDB, opening in restricted mode, renaming via `RENAME GLOBAL_NAME TO`, and verification. |
| 8 | [rman-delete-archive-log.md](rman-delete-archive-log.md) | Guide to safely delete archive log files older than 1 month via RMAN — covers crosscheck, list preview, delete options, and verification with sample output. |
| 9 | [shutdown-immediate-hang.md](shutdown-immediate-hang.md) | Step-by-step guide to resolve `SHUTDOWN IMMEDIATE` hang on Oracle Database — covers diagnosis, escalation to `SHUTDOWN ABORT`, OS-level force kill on Oracle Linux and Windows Server, IPC cleanup, crash recovery, and prevention tips, with Mermaid flow diagrams. |
