# Oracle Data Pump Export (expdp) — Backup Guide

Oracle Data Pump Export (`expdp`) is a server-side utility for exporting database objects and data into a binary dump file. It supports full database, schema, tablespace, and table-level exports. This guide covers directory setup, all available options, version-specific features (12c, 19c, 21c), usage examples, and troubleshooting.

---

## Table of Contents

1. [Directory Setup](#1-directory-setup)
2. [Export Modes](#2-export-modes)
3. [All Export Parameters](#3-all-export-parameters)
4. [Version-Specific Features](#4-version-specific-features)
5. [Usage Examples](#5-usage-examples)
6. [Monitoring a Running Job](#6-monitoring-a-running-job)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. Directory Setup

Data Pump requires an Oracle **DIRECTORY object** that points to a physical path on the server. The dump files are written to and read from that location — not from the client machine.

### 1.1 Create the Physical Directory

**Windows:**
```cmd
mkdir C:\oracle\datapump
```

**Linux:**
```bash
mkdir -p /oracle/datapump
chown oracle:oinstall /oracle/datapump
chmod 750 /oracle/datapump
```

### 1.2 Create the Oracle Directory Object

Connect as `SYS AS SYSDBA` and run:

**Windows:**
```sql
CREATE OR REPLACE DIRECTORY DATAPUMP_DIR AS 'C:\oracle\datapump';
```

**Linux:**
```sql
CREATE OR REPLACE DIRECTORY DATAPUMP_DIR AS '/oracle/datapump';
```

### 1.3 Grant Privileges to the User

```sql
GRANT READ, WRITE ON DIRECTORY DATAPUMP_DIR TO scott;
```

### 1.4 Verify the Directory

```sql
SELECT directory_name, directory_path
FROM dba_directories
WHERE directory_name = 'DATAPUMP_DIR';
```

**Example output:**
```
DIRECTORY_NAME     DIRECTORY_PATH
------------------ ----------------------
DATAPUMP_DIR       /oracle/datapump
```

---

## 2. Export Modes

| Mode | Parameter | Description |
|---|---|---|
| Full Database | `FULL=Y` | Exports the entire database |
| Schema | `SCHEMAS=schema1,schema2` | Exports one or more schemas |
| Tablespace | `TABLESPACES=tbs1,tbs2` | Exports all objects in specified tablespaces |
| Table | `TABLES=schema.table1,schema.table2` | Exports specific tables |
| Transportable Tablespace | `TRANSPORT_TABLESPACES=tbs1` | Exports metadata for transportable tablespace |

---

## 3. All Export Parameters

### 3.1 Connection & Authentication

| Parameter | Description | Example |
|---|---|---|
| `USERID` | Connection string | `USERID=system/password` |
| `ATTACH` | Attach to an existing export job | `ATTACH=job_name` |

### 3.2 Output Files

| Parameter | Description | Example |
|---|---|---|
| `DIRECTORY` | Oracle directory object for dump/log files | `DIRECTORY=DATAPUMP_DIR` |
| `DUMPFILE` | Name of the dump file(s). Use `%U` for multiple files | `DUMPFILE=exp_%U.dmp` |
| `LOGFILE` | Name of the log file | `LOGFILE=export.log` |
| `NOLOGFILE` | Suppress log file creation | `NOLOGFILE=Y` |
| `FILESIZE` | Maximum size per dump file | `FILESIZE=2G` |
| `REUSE_DUMPFILES` | Overwrite existing dump files | `REUSE_DUMPFILES=Y` |

### 3.3 Scope & Content

| Parameter | Description | Example |
|---|---|---|
| `FULL` | Full database export | `FULL=Y` |
| `SCHEMAS` | Schema-level export | `SCHEMAS=HR,SCOTT` |
| `TABLES` | Table-level export | `TABLES=HR.EMPLOYEES,HR.DEPARTMENTS` |
| `TABLESPACES` | Tablespace-level export | `TABLESPACES=USERS,DATA_TBS` |
| `CONTENT` | What to export: `ALL`, `DATA_ONLY`, `METADATA_ONLY` | `CONTENT=METADATA_ONLY` |
| `QUERY` | Filter rows during export | `QUERY=HR.EMPLOYEES:"WHERE dept_id=10"` |
| `SAMPLE` | Export a percentage sample of data | `SAMPLE=50` |

### 3.4 Filtering Objects

| Parameter | Description | Example |
|---|---|---|
| `INCLUDE` | Include only specific object types | `INCLUDE=TABLE,VIEW` |
| `EXCLUDE` | Exclude specific object types | `EXCLUDE=STATISTICS,INDEX` |

**INCLUDE/EXCLUDE syntax:**
```bash
INCLUDE=object_type[:name_clause]
EXCLUDE=object_type[:name_clause]
```

**Example — exclude specific tables:**
```bash
EXCLUDE=TABLE:"IN ('LOG_TABLE','TEMP_TABLE')"
```

**Example — include only specific tables:**
```bash
INCLUDE=TABLE:"IN ('EMPLOYEES','DEPARTMENTS')"
```

### 3.5 Performance

| Parameter | Description | Example |
|---|---|---|
| `PARALLEL` | Number of parallel worker processes | `PARALLEL=4` |
| `ESTIMATE` | Estimate method: `BLOCKS` or `STATISTICS` | `ESTIMATE=STATISTICS` |
| `ESTIMATE_ONLY` | Estimate size without exporting | `ESTIMATE_ONLY=Y` |

### 3.6 Compression & Encryption

| Parameter | Values | Description |
|---|---|---|
| `COMPRESSION` | `ALL`, `DATA_ONLY`, `METADATA_ONLY`, `NONE` | Compress dump file content (requires Advanced Compression option) |
| `COMPRESSION_ALGORITHM` | `BASIC`, `LOW`, `MEDIUM`, `HIGH` | Compression algorithm (19c+) |
| `ENCRYPTION` | `ALL`, `DATA_ONLY`, `METADATA_ONLY`, `ENCRYPTED_COLUMNS_ONLY`, `NONE` | Encrypt dump file |
| `ENCRYPTION_ALGORITHM` | `AES128`, `AES192`, `AES256` | Encryption algorithm |
| `ENCRYPTION_MODE` | `PASSWORD`, `TRANSPARENT`, `DUAL` | Encryption mode |
| `ENCRYPTION_PASSWORD` | Password string | Used with `ENCRYPTION_MODE=PASSWORD` |

### 3.7 Consistency & Flashback

| Parameter | Description | Example |
|---|---|---|
| `FLASHBACK_TIME` | Export data as of a specific time | `FLASHBACK_TIME="TO_TIMESTAMP('2025-05-01 08:00:00','YYYY-MM-DD HH24:MI:SS')"` |
| `FLASHBACK_SCN` | Export data as of a specific SCN | `FLASHBACK_SCN=123456` |

### 3.8 Job Control

| Parameter | Description | Example |
|---|---|---|
| `JOB_NAME` | Assign a name to the export job | `JOB_NAME=FULL_EXPORT_20250505` |
| `STATUS` | Interval (seconds) to display job status | `STATUS=30` |
| `VERSION` | Target DB version for compatibility | `VERSION=12.2` |
| `CLUSTER` | Use RAC cluster resources | `CLUSTER=Y` |

### 3.9 Transportable Tablespace

| Parameter | Description | Example |
|---|---|---|
| `TRANSPORT_TABLESPACES` | Tablespaces to transport | `TRANSPORT_TABLESPACES=DATA_TBS` |
| `TRANSPORT_FULL_CHECK` | Check for dependencies outside the tablespace set | `TRANSPORT_FULL_CHECK=Y` |
| `TRANSPORTABLE` | Use transportable method for table export | `TRANSPORTABLE=ALWAYS` |

---

## 4. Version-Specific Features

### 4.1 Oracle 12c

| Feature | Parameter | Description |
|---|---|---|
| Views as Tables | `VIEWS_AS_TABLES` | Export a view as if it were a table | 
| Transportable | `TRANSPORTABLE=ALWAYS` | Force transportable export for tables |
| Exclude Pluggable DB | `EXCLUDE=PLUGGABLE_DATABASE` | Exclude PDB metadata in CDB full export |

**12c — Export from PDB:**
```bash
expdp system/password@pdb1 FULL=Y DIRECTORY=DATAPUMP_DIR \
DUMPFILE=pdb1_full.dmp LOGFILE=pdb1_full.log
```

### 4.2 Oracle 19c

| Feature | Parameter | Description |
|---|---|---|
| Checksum | `CHECKSUM=SHA256` | Add checksum to dump file for integrity verification |
| Disable Archive Logging | `DISABLE_ARCHIVE_LOGGING=Y` | Reduce redo generation during import (import-side) |
| Compression Algorithm | `COMPRESSION_ALGORITHM=HIGH` | Fine-grained compression control |

**19c — Export with checksum and compression:**
```bash
expdp system/password SCHEMAS=HR DIRECTORY=DATAPUMP_DIR \
DUMPFILE=hr_19c.dmp LOGFILE=hr_19c.log \
COMPRESSION=ALL COMPRESSION_ALGORITHM=MEDIUM \
CHECKSUM=SHA256
```

### 4.3 Oracle 21c

| Feature | Description |
|---|---|
| Blockchain Table Support | Data Pump now supports export/import of blockchain tables |
| Mandatory Column Support | Export/import tables with mandatory columns |
| Improved Parallel Export | Enhanced parallelism for large tables |

**21c — Full export with parallel:**
```bash
expdp system/password FULL=Y DIRECTORY=DATAPUMP_DIR \
DUMPFILE=full_21c_%U.dmp LOGFILE=full_21c.log \
PARALLEL=8 COMPRESSION=ALL COMPRESSION_ALGORITHM=HIGH \
JOB_NAME=FULL_EXPORT_21C
```

---

## 5. Usage Examples

### 5.1 Full Database Export

```bash
expdp system/password FULL=Y \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=full_db_%U.dmp \
LOGFILE=full_db.log \
PARALLEL=4 \
COMPRESSION=METADATA_ONLY \
JOB_NAME=FULL_DB_EXPORT
```

**Example output:**
```
Export: Release 19.0.0.0.0 - Production on Mon May 5 09:00:00 2025

Copyright (c) 1982, 2019, Oracle and/or its affiliates. All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0

Starting "SYSTEM"."FULL_DB_EXPORT":  system/******** FULL=Y DIRECTORY=DATAPUMP_DIR ...
Estimate in progress using BLOCKS method...
Processing object type DATABASE_EXPORT/EARLY_OPTIONS/VIEWS_AS_TABLES/TABLE_DATA
Processing object type DATABASE_EXPORT/NORMAL_OPTIONS/TABLE_DATA
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 2.5 GB
Processing object type DATABASE_EXPORT/PRE_SYSTEM_IMPCALLOUT/MARKER
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/TABLE
...
. . exported "HR"."EMPLOYEES"                          1.2 MB   107 rows
. . exported "HR"."DEPARTMENTS"                        22 KB    27 rows
. . exported "SCOTT"."EMP"                             8 KB     14 rows
...
Master table "SYSTEM"."FULL_DB_EXPORT" successfully loaded/unloaded
******************************************************************************
Dump file set for SYSTEM.FULL_DB_EXPORT is:
  /oracle/datapump/full_db_01.dmp
  /oracle/datapump/full_db_02.dmp
Job "SYSTEM"."FULL_DB_EXPORT" successfully completed at Mon May 5 09:12:34 2025 elapsed 0 00:12:34
```

---

### 5.2 Schema Export

```bash
expdp system/password \
SCHEMAS=HR,SCOTT \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=schema_hr_scott.dmp \
LOGFILE=schema_hr_scott.log \
COMPRESSION=ALL
```

**Example output:**
```
Starting "SYSTEM"."SYS_EXPORT_SCHEMA_01":  system/******** SCHEMAS=HR,SCOTT ...
Estimate in progress using BLOCKS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 15 MB
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
. . exported "HR"."EMPLOYEES"                          1.2 MB   107 rows
. . exported "HR"."DEPARTMENTS"                        22 KB    27 rows
. . exported "SCOTT"."EMP"                             8 KB     14 rows
. . exported "SCOTT"."DEPT"                            5 KB     4 rows
Master table "SYSTEM"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
Dump file set: /oracle/datapump/schema_hr_scott.dmp
Job "SYSTEM"."SYS_EXPORT_SCHEMA_01" successfully completed at ... elapsed 0 00:01:45
```

---

### 5.3 Table Export with Row Filter

```bash
expdp system/password \
TABLES=HR.EMPLOYEES \
QUERY=HR.EMPLOYEES:'"WHERE department_id=60"' \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=emp_dept60.dmp \
LOGFILE=emp_dept60.log
```

**Example output:**
```
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . exported "HR"."EMPLOYEES"                          5 KB    5 rows
Job "SYSTEM"."SYS_EXPORT_TABLE_01" successfully completed ...
```

---

### 5.4 Metadata Only Export (No Data)

```bash
expdp system/password \
SCHEMAS=HR \
CONTENT=METADATA_ONLY \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=hr_metadata.dmp \
LOGFILE=hr_metadata.log
```

---

### 5.5 Estimate Size Without Exporting

```bash
expdp system/password \
SCHEMAS=HR \
ESTIMATE_ONLY=Y \
ESTIMATE=STATISTICS \
DIRECTORY=DATAPUMP_DIR \
LOGFILE=hr_estimate.log
```

**Example output:**
```
Estimate in progress using STATISTICS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
.  estimated "HR"."EMPLOYEES"                          2 MB
.  estimated "HR"."DEPARTMENTS"                        512 KB
Total estimation using STATISTICS method: 2.5 MB
Job "SYSTEM"."SYS_EXPORT_SCHEMA_01" successfully completed ...
```

---

### 5.6 Encrypted Export

```bash
expdp system/password \
SCHEMAS=HR \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=hr_encrypted.dmp \
LOGFILE=hr_encrypted.log \
ENCRYPTION=ALL \
ENCRYPTION_ALGORITHM=AES256 \
ENCRYPTION_MODE=PASSWORD \
ENCRYPTION_PASSWORD=MySecurePass123
```

---

### 5.7 Exclude Statistics and Grants

```bash
expdp system/password \
SCHEMAS=HR \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=hr_clean.dmp \
LOGFILE=hr_clean.log \
EXCLUDE=STATISTICS,GRANT,OBJECT_GRANT
```

---

### 5.8 Export from PDB (12c and above)

```bash
# Connect using PDB service name
expdp system/password@ORCLPDB1 \
SCHEMAS=HR \
DIRECTORY=DATAPUMP_DIR \
DUMPFILE=pdb_hr.dmp \
LOGFILE=pdb_hr.log
```

---

## 6. Monitoring a Running Job

### 6.1 Attach to a Running Job

```bash
expdp system/password ATTACH=FULL_DB_EXPORT
```

**Interactive commands inside the job monitor:**
```
Export> STATUS          -- show current status
Export> PARALLEL=8      -- increase parallel workers
Export> KILL_JOB        -- terminate the job
Export> STOP_JOB        -- suspend the job
Export> CONTINUE_CLIENT -- resume display
```

### 6.2 Check Active Jobs from SQL

```sql
SELECT job_name, operation, job_mode, state, degree
FROM dba_datapump_jobs
WHERE state = 'EXECUTING';
```

**Example output:**
```
JOB_NAME                  OPERATION  JOB_MODE  STATE      DEGREE
------------------------- ---------- --------- ---------- ------
FULL_DB_EXPORT            EXPORT     FULL      EXECUTING  4
```

---

## 7. Troubleshooting

### Error: ORA-39002: invalid operation / ORA-39070: Unable to open the log file

**Cause:** The directory object does not exist or the OS path is invalid.

**Solution:**
```sql
-- Verify directory exists
SELECT directory_name, directory_path FROM dba_directories WHERE directory_name = 'DATAPUMP_DIR';

-- Re-create if missing
CREATE OR REPLACE DIRECTORY DATAPUMP_DIR AS '/oracle/datapump';
GRANT READ, WRITE ON DIRECTORY DATAPUMP_DIR TO system;
```

---

### Error: ORA-39006: internal error / ORA-39213: Metadata processing is not available

**Cause:** The `SYS.METASTYLESHEET` table or XSL stylesheets are invalid.

**Solution:**
```sql
-- Run as SYS
@$ORACLE_HOME/rdbms/admin/catmeta.sql
@$ORACLE_HOME/rdbms/admin/utlrp.sql
```

---

### Error: ORA-31634: job already exists

**Cause:** A job with the same `JOB_NAME` already exists in the database.

**Solution:**
```sql
-- Drop the old job
SELECT job_name FROM dba_datapump_jobs WHERE job_name = 'FULL_DB_EXPORT';
DROP TABLE system.FULL_DB_EXPORT;
```

Or use a different job name:
```bash
JOB_NAME=FULL_DB_EXPORT_2
```

---

### Error: ORA-39095: Dump file space has been exhausted

**Cause:** The dump file(s) reached the size limit or the disk is full.

**Solution:**
```bash
# Use multiple files with %U wildcard and set file size limit
DUMPFILE=full_db_%U.dmp
FILESIZE=4G
```

Or add more dump files while the job is running:
```bash
expdp system/password ATTACH=FULL_DB_EXPORT
Export> ADD_FILE=DATAPUMP_DIR:full_db_03.dmp
```

---

### Error: ORA-01555: snapshot too old

**Cause:** The undo tablespace ran out of space during a long export.

**Solution:**
```sql
-- Increase undo retention
ALTER SYSTEM SET UNDO_RETENTION=10800;  -- 3 hours

-- Or use FLASHBACK_SCN to export a consistent snapshot
expdp ... FLASHBACK_SCN=<current_scn>
```

Get current SCN:
```sql
SELECT current_scn FROM v$database;
```

---

### Error: ORA-39126: Worker unexpected fatal error in KUPW$WORKER

**Cause:** A corrupted object or unsupported object type.

**Solution:**
Use `EXCLUDE` to skip the problematic object:
```bash
EXCLUDE=TABLE:"IN ('PROBLEM_TABLE')"
```

---

### Export Runs Slowly

**Tips to improve performance:**

1. Increase parallel workers:
```bash
PARALLEL=4
```

2. Use `COMPRESSION=METADATA_ONLY` to reduce I/O without CPU overhead.

3. Disable redo logging (19c+):
```bash
DISABLE_ARCHIVE_LOGGING=Y
```

4. Use `ESTIMATE=BLOCKS` for faster estimation.

5. Place dump files on fast storage (SSD or separate disk from data files).

---

### Check Export Log for Errors

Always review the log file after export:

```bash
# Linux
grep -i "ORA-\|error\|warning" /oracle/datapump/export.log

# Windows
findstr /I "ORA- error warning" C:\oracle\datapump\export.log
```
