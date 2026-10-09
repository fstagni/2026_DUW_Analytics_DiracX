# DIRAC Database Schemas

## Overview

DIRAC uses **MySQL** as its primary OLTP database engine. Each subsystem has its own database with a corresponding `.sql` schema file and a Python DB class that extends `DIRAC.Core.Base.DB` (which wraps MySQL). Some components also use **OpenSearch/Elasticsearch** for specific use cases (e.g., JobParametersDB).

**Base class**: `DB` → `DIRACDB` + `MySQL` (`src/DIRAC/Core/Base/DB.py:9`)

---

## JobDB — Main Workload Management Database

**Schema file**: `src/DIRAC/WorkloadManagementSystem/DB/JobDB.sql`
**Python class**: `src/DIRAC/WorkloadManagementSystem/DB/JobDB.py`

### Tables

#### `JobJDLs` (JobJDLs.sql:24-31)
Stores job definition language (JDL) documents.

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | AUTO_INCREMENT, PRIMARY KEY |
| `JDL` | MEDIUMTEXT | NOT NULL |
| `JobRequirements` | TEXT | NOT NULL |
| `OriginalJDL` | MEDIUMTEXT | NOT NULL |

#### `Jobs` (JobJDLs.sql:34-70)
Main job status and metadata table. Foreign key to `JobJDLs(JobID)`.

| Column | Type | Default | Description |
|--------|------|---------|-------------|
| `JobID` | INT(11) UNSIGNED | 0 | PRIMARY KEY, FK → JobJDLs |
| `JobType` | VARCHAR(32) | 'user' | e.g., "user", "test" |
| `JobGroup` | VARCHAR(32) | '00000000' | Grouping identifier |
| `Site` | VARCHAR(100) | 'ANY' | Target execution site |
| `JobName` | VARCHAR(128) | 'Unknown' | Human-readable name |
| `Owner` | VARCHAR(64) | 'Unknown' | User name |
| `OwnerGroup` | VARCHAR(128) | 'Unknown' | VOMS group |
| `VO` | VARCHAR(32) | 'Unknown' | Virtual Organization |
| `SubmissionTime` | DATETIME | NULL | When job was submitted |
| `RescheduleTime` | DATETIME | NULL | Last reschedule timestamp |
| `LastUpdateTime` | DATETIME | NULL | Last status update |
| `StartExecTime` | DATETIME | NULL | When execution started |
| `HeartBeatTime` | DATETIME | NULL | Last heartbeat |
| `EndExecTime` | DATETIME | NULL | When execution ended |
| `Status` | VARCHAR(32) | 'Received' | Major status |
| `MinorStatus` | VARCHAR(128) | 'Unknown' | Minor status |
| `ApplicationStatus` | VARCHAR(255) | 'Unknown' | Application-level status |
| `UserPriority` | INT(11) | 0 | User-defined priority |
| `RescheduleCounter` | INT(11) | 0 | Number of reschedules |
| `VerifiedFlag` | ENUM('True','False') | 'False' | Verification flag |
| `AccountedFlag` | ENUM('True','False','Failed') | 'False' | Accounting flag |

**Indexes**: `JobType`, `JobGroup`, `Site`, `Owner`, `OwnerGroup`, `Status`, `MinorStatus`, `ApplicationStatus`, `StatusSite` (composite), `LastUpdateTime`, `JobName`

#### `InputData` (JobDB.sql:73-80)
Job input data files (LFNs).

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `LFN` | VARCHAR(255) | PRIMARY KEY (with JobID) |
| `Status` | VARCHAR(32) | DEFAULT 'AprioriGood' |

#### `JobParameters` (JobDB.sql:83-90)
Runtime job parameters (being migrated to OpenSearch — see `JobParametersDB.py`).

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `Name` | VARCHAR(100) | PRIMARY KEY (with JobID) |
| `Value` | TEXT | NOT NULL |

#### `OptimizerParameters` (JobDB.sql:93-100)
Optimizer chain parameters.

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `Name` | VARCHAR(100) | PRIMARY KEY (with JobID) |
| `Value` | MEDIUMTEXT | NOT NULL |

#### `AtticJobParameters` (JobDB.sql:103-111)
Archived parameters from previous reschedule cycles.

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `Name` | VARCHAR(100) | PRIMARY KEY (with JobID, RescheduleCycle) |
| `Value` | TEXT | NOT NULL |
| `RescheduleCycle` | INT(11) UNSIGNED | NOT NULL |

#### `HeartBeatLoggingInfo` (JobDB.sql:114-122)
Heartbeat data from running jobs.

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `Name` | VARCHAR(100) | PRIMARY KEY (with JobID, HeartBeatTime) |
| `Value` | TEXT | NOT NULL |
| `HeartBeatTime` | DATETIME | NOT NULL |

#### `JobCommands` (JobDB.sql:125-135)
Commands to be sent to running jobs (e.g., Kill).

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INT(11) UNSIGNED | FK → Jobs(JobID) |
| `Command` | VARCHAR(100) | PRIMARY KEY (with JobID, Arguments, ReceptionTime) |
| `Arguments` | VARCHAR(100) | NOT NULL |
| `Status` | VARCHAR(64) | DEFAULT 'Received' |
| `ReceptionTime` | DATETIME | NOT NULL |
| `ExecutionTime` | DATETIME | NULL |

---

## JobLoggingDB — Job Status History

**Schema file**: `src/DIRAC/WorkloadManagementSystem/DB/JobLoggingDB.sql`
**Python class**: `src/DIRAC/WorkloadManagementSystem/DB/JobLoggingDB.py`

### `LoggingInfo` (JobLoggingDB.sql:25-36)

| Column | Type | Constraints |
|--------|------|-------------|
| `JobID` | INTEGER | NOT NULL, PRIMARY KEY (with SeqNum) |
| `SeqNum` | INTEGER | AUTO-generated by trigger, PRIMARY KEY (with JobID) |
| `Status` | VARCHAR(32) | NOT NULL, DEFAULT '' |
| `MinorStatus` | VARCHAR(128) | NOT NULL, DEFAULT '' |
| `ApplicationStatus` | VARCHAR(255) | NOT NULL, DEFAULT '' |
| `StatusTime` | DATETIME | NOT NULL |
| `StatusTimeOrder` | DOUBLE(12,3) | NOT NULL — used for ordering |
| `StatusSource` | VARCHAR(32) | NOT NULL, DEFAULT 'Unknown' |

**Trigger**: `SeqNumGenerator` — auto-generates sequential `SeqNum` per `JobID` on insert (JobLoggingDB.sql:44-45).

---

## TaskQueueDB — Job Matching Queue

**Schema file**: `src/DIRAC/WorkloadManagementSystem/DB/TaskQueueDB.sql` (nearly empty — schema created programmatically)
**Python class**: `src/DIRAC/WorkloadManagementSystem/DB/TaskQueueDB.py`

Tables are created dynamically in `__initializeDB()` (TaskQueueDB.py:96-157):

### `tq_TaskQueues` (TaskQueueDB.py:107-119)

| Column | Type | Constraints |
|--------|------|-------------|
| `TQId` | INTEGER(11) UNSIGNED | AUTO_INCREMENT, PRIMARY KEY |
| `Owner` | VARCHAR(255) | NOT NULL |
| `OwnerGroup` | VARCHAR(32) | NOT NULL |
| `VO` | VARCHAR(32) | NOT NULL |
| `CPUTime` | BIGINT(20) UNSIGNED | NOT NULL |
| `Priority` | FLOAT | NOT NULL |
| `Enabled` | TINYINT(1) | NOT NULL, DEFAULT 0 |

**Index**: `TQOwner` on (Owner, OwnerGroup, CPUTime)

### `tq_Jobs` (TaskQueueDB.py:121-131)

| Column | Type | Constraints |
|--------|------|-------------|
| `TQId` | INTEGER(11) UNSIGNED | NOT NULL, FK → tq_TaskQueues.TQId |
| `JobId` | INTEGER(11) UNSIGNED | NOT NULL, PRIMARY KEY |
| `Priority` | INTEGER UNSIGNED | NOT NULL |
| `RealPriority` | FLOAT | NOT NULL |

**Index**: `TaskIndex` on (TQId)

### `tq_RAM_requirements` (TaskQueueDB.py:133-141)

| Column | Type | Constraints |
|--------|------|-------------|
| `TQId` | INTEGER(11) UNSIGNED | NOT NULL, PRIMARY KEY, FK → tq_TaskQueues.TQId |
| `MinRAM` | INTEGER UNSIGNED | NOT NULL, DEFAULT 0 |
| `MaxRAM` | INTEGER UNSIGNED | NOT NULL, DEFAULT 0 |

### `tq_TQTo{Field}` tables (TaskQueueDB.py:143-150)

Dynamic tables for multi-value fields: `tq_TQToSites`, `tq_TQToGridCEs`, `tq_TQToBannedSites`, `tq_TQToPlatforms`, `tq_TQToJobTypes`, `tq_TQToTags`.

| Column | Type | Constraints |
|--------|------|-------------|
| `TQId` | INTEGER(11) UNSIGNED | NOT NULL, FK → tq_TaskQueues.TQId |
| `Value` | VARCHAR(64) | NOT NULL, PRIMARY KEY (with TQId) |

**Indexes**: `TaskIndex` on (TQId, Value), `{Field}Index` on (Value)

---

## PilotAgentsDB — Pilot Job Tracking

**Schema file**: `src/DIRAC/WorkloadManagementSystem/DB/PilotAgentsDB.sql`
**Python class**: `src/DIRAC/WorkloadManagementSystem/DB/PilotAgentsDB.py`

### `PilotAgents` (PilotAgentsDB.sql:25-48)

| Column | Type | Default | Description |
|--------|------|---------|-------------|
| `PilotID` | INT(11) UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | |
| `InitialJobID` | INT(11) UNSIGNED | 0 | First job assigned |
| `CurrentJobID` | INT(11) UNSIGNED | 0 | Currently running job |
| `PilotJobReference` | VARCHAR(255) | 'Unknown' | External pilot reference |
| `PilotStamp` | VARCHAR(32) | '' | Unique pilot stamp |
| `DestinationSite` | VARCHAR(128) | 'NotAssigned' | CE/Site |
| `Queue` | VARCHAR(128) | 'Unknown' | Queue name |
| `GridSite` | VARCHAR(128) | 'Unknown' | Grid site |
| `VO` | VARCHAR(128) | — | Virtual Organization |
| `GridType` | VARCHAR(32) | 'LCG' | Grid middleware type |
| `BenchMark` | DOUBLE | 0.0 | Benchmark score |
| `SubmissionTime` | DATETIME | NULL | |
| `LastUpdateTime` | DATETIME | NULL | |
| `Status` | VARCHAR(32) | 'Unknown' | Pilot status |
| `StatusReason` | VARCHAR(255) | 'Unknown' | Reason for status |
| `AccountingSent` | ENUM('True','False') | 'False' | Accounting flag |

**Indexes**: `PilotJobReference`, `Status`, `Statuskey` (GridSite, DestinationSite, Status), `idx_dest_queue_status` (DestinationSite, Queue, Status)

### `JobToPilotMapping` (PilotAgentsDB.sql:51-58)

| Column | Type | Constraints |
|--------|------|-------------|
| `PilotID` | INT(11) UNSIGNED | NOT NULL, KEY |
| `JobID` | INT(11) UNSIGNED | NOT NULL, KEY |
| `StartTime` | DATETIME | NOT NULL |

### `PilotOutput` (PilotAgentsDB.sql:60-66)

| Column | Type | Constraints |
|--------|------|-------------|
| `PilotID` | INT(11) UNSIGNED | NOT NULL, PRIMARY KEY |
| `StdOutput` | MEDIUMTEXT | NULL |
| `StdError` | MEDIUMTEXT | NULL |

---

## Job Status Values

Defined in `dirac-common/src/DIRACCommon/WorkloadManagementSystem/Client/JobStatus.py:8-59`:

| Status | Description |
|--------|-------------|
| `Submitting` | Initial state |
| `Received` | Received by WMS |
| `Checking` | Being validated |
| `Staging` | Input data staging |
| `Scouting` | Scout job mode |
| `Waiting` | Waiting for resources |
| `Matched` | Matched to a pilot |
| `Rescheduled` | Virtual status (not stored) |
| `Running` | Executing on a worker node |
| `Stalled` | No heartbeat received |
| `Completing` | Finalizing execution |
| `Done` | Completed successfully |
| `Completed` | Completed (alternative) |
| `Failed` | Execution failed |
| `Deleted` | Removed from system |
| `Killed` | User/admin killed |

**Final states**: `Done`, `Completed`, `Failed`, `Killed`

---

## Job Minor Status Values

Defined in `src/DIRAC/WorkloadManagementSystem/Client/JobMinorStatus.py:5-79`. Key values include:

- `Application`, `Application Finished Successfully`, `Application Finished With Errors`
- `Job Initialization`, `Job Wrapper Initialization`, `JobWrapper execution`
- `Downloading InputSandbox`, `Uploading Output Sandbox`, `Output Sandbox Uploaded`
- `Job Rescheduled`, `Going to reschedule job`
- `Job stalled: pilot not running`, `Watchdog identified this job as stalled`
- `No candidate sites available`, `Input Data Not Available`

---

## AccountingDB — Analytics Data

**Schema file**: `src/DIRAC/AccountingSystem/DB/AccountingDB.sql` (nearly empty — schema created programmatically)
**Python class**: `src/DIRAC/AccountingSystem/DB/AccountingDB.py`

Uses a dynamic schema with a catalog table and per-type tables:

### Catalog table (AccountingDB.py:48-59)

| Column | Type | Constraints |
|--------|------|-------------|
| `name` | VARCHAR(64) | UNIQUE, NOT NULL, PRIMARY KEY |
| `keyFields` | VARCHAR(255) | NOT NULL — dimension fields |
| `valueFields` | VARCHAR(255) | NOT NULL — metric fields |
| `bucketsLength` | VARCHAR(255) | NOT NULL — bucketing intervals |

Per-type tables are created dynamically with `key` (dimension) columns and `value` (metric) columns, plus bucketing for time aggregation.

---

## TransformationDB — Data Transformation Engine

**Schema file**: `src/DIRAC/TransformationSystem/DB/TransformationDB.sql`

### Key Tables

| Table | Purpose |
|-------|---------|
| `Transformations` | Transformation definitions |
| `DataFiles` | LFN registry |
| `TransformationTasks` | Task instances |
| `TransformationFiles` | File-to-transformation mapping |
| `TransformationFileTasks` | File-to-task mapping |
| `TaskInputs` | Input vectors per task |
| `TransformationLog` | Audit log |
| `AdditionalParameters` | Extra parameters |
| `TransformationMetaQueries` | Metadata queries |

---

## Other DBs

| Database | Schema File | Purpose |
|----------|-------------|---------|
| `FileCatalogDB` | `DataManagementSystem/DB/FileCatalogDB.sql` | LFN metadata catalog |
| `RequestManagementDB` | `RequestManagementSystem/DB/ReqDB.sql` | Request execution |
| `ResourceStatusDB` | `ResourceStatusSystem/DB/ResourceStatusDB.sql` | Site/CE status |
| `ResourceManagementDB` | `ResourceStatusSystem/DB/ResourceManagementDB.sql` | Resource metrics |
| `StorageManagementDB` | `StorageManagementSystem/DB/StorageManagementDB.sql` | Storage staging |
| `ProductionDB` | `ProductionSystem/DB/ProductionDB.sql` | Production workflows |
| `UserProfileDB` | `FrameworkSystem/DB/UserProfileDB.sql` | User preferences |
| `ProxyDB` | `FrameworkSystem/DB/ProxyDB.sql` | Proxy management |
| `TokenDB` | `FrameworkSystem/DB/TokenDB.sql` | Auth tokens |
| `AuthDB` | `FrameworkSystem/DB/AuthDB.sql` | Authentication |
| `InstalledComponentsDB` | `FrameworkSystem/DB/InstalledComponentsDB.sql` | Component registry |
| `FTS3DB` | `DataManagementSystem/DB/FTS3DB.sql` | File transfer service |
| `DataIntegrityDB` | `DataManagementSystem/DB/DataIntegrityDB.sql` | Data integrity checks |
| `MonitoringDB` | `MonitoringSystem/DB/MonitoringDB.py` | System metrics (no .sql) |

---

## JobParametersDB (OpenSearch/Elasticsearch)

**Python class**: `src/DIRAC/WorkloadManagementSystem/DB/JobParametersDB.py`

Migrated from MySQL to OpenSearch for scalability. Index name: `job_parameters` (configurable).

### OpenSearch Mapping (JobParametersDB.py:14-33)

| Field | Type |
|-------|------|
| `JobID` | long |
| `timestamp` | date |
| `PilotAgent` | keyword |
| `Pilot_Reference` | keyword |
| `JobGroup` | keyword |
| `CPUNormalizationFactor` | long |
| `NormCPUTime(s)` | long |
| `Memory(MB)` | long |
| `LocalAccount` | keyword |
| `TotalCPUTime(s)` | long |
| `PayloadPID` | long |
| `HostName` | text |
| `GridCE` | keyword |
| `CEQueue` | keyword |
| `BatchSystem` | keyword |
| `ModelName` | keyword |
| `Status` | keyword |
| `JobType` | keyword |

Index naming convention: `job_parameters_{vo}_{N}m` where N = JobID // 1,000,000.

---

## Example Data for Presentations

### Sample Job Record

```json
{
  "JobID": 12345,
  "JobType": "user",
  "JobGroup": "prod-sim-2026",
  "Site": "LCG.CERN.ch",
  "JobName": "sim_prod_001",
  "Owner": "jdoe",
  "OwnerGroup": "lhcb_user",
  "VO": "lhcb",
  "SubmissionTime": "2026-10-01 10:00:00",
  "RescheduleTime": null,
  "LastUpdateTime": "2026-10-01 10:05:30",
  "StartExecTime": "2026-10-01 10:02:00",
  "HeartBeatTime": "2026-10-01 10:05:00",
  "EndExecTime": null,
  "Status": "Running",
  "MinorStatus": "Application",
  "ApplicationStatus": "Unknown",
  "UserPriority": 1,
  "RescheduleCounter": 0,
  "VerifiedFlag": "True",
  "AccountedFlag": "False"
}
```

### Sample JobLogging Record

```json
{
  "JobID": 12345,
  "SeqNum": 1,
  "Status": "Received",
  "MinorStatus": "Job accepted",
  "ApplicationStatus": "Unknown",
  "StatusTime": "2026-10-01 10:00:00",
  "StatusTimeOrder": 1270000000.000,
  "StatusSource": "JobManager"
}
```

### Sample Pilot Record

```json
{
  "PilotID": 98765,
  "InitialJobID": 12345,
  "CurrentJobID": 12345,
  "PilotJobReference": "https://ce.example.org:8443/cream-pilot-001",
  "PilotStamp": "abc123def",
  "DestinationSite": "LCG.CERN.ch",
  "Queue": "cloud-lx",
  "GridSite": "CERN-PROD",
  "VO": "lhcb",
  "GridType": "ARC",
  "BenchMark": 250.5,
  "SubmissionTime": "2026-10-01 09:55:00",
  "LastUpdateTime": "2026-10-01 10:05:00",
  "Status": "Running",
  "StatusReason": "Report from job 12345",
  "AccountingSent": "False"
}
```

### Sample TaskQueue Record

```json
{
  "TQId": 42,
  "Owner": "jdoe",
  "OwnerGroup": "lhcb_user",
  "VO": "lhcb",
  "CPUTime": 86400,
  "Priority": 1.0,
  "Enabled": 1
}
```

---

## Data Flow for Analytics (ELT)

As described in the presentation slides (`slides.md`):

1. **Extract**: Incremental queries from OLTP (`SELECT * WHERE LastUpdateTime > ?`)
2. **Load**: Into DuckLake (Parquet files on S3 + PostgreSQL catalog)
3. **Transform**: Bucketing (hourly/daily/monthly) via DuckDB

Key extraction points:
- `Jobs` table: filter by `LastUpdateTime`
- `LoggingInfo` table: filter by `StatusTimeOrder`
- `PilotAgents` table: filter by `LastUpdateTime`
- AccountingDB: use pending records mechanism (`loadPendingRecords()`)
