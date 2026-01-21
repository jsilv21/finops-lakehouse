# **FinOps Lakehouse (Local DuckDB) - Minimal Requirements**

### **Core Principles**

- Local-first, no required cloud compute (S3 reads for exports are standard)
- Minimal tooling: **Python + DuckDB + dbt + Streamlit/Evidence**
- Drop-in replaceable data (demo -> user billing)
- Open-source "template" usable by small teams
- Clear senior-level engineering patterns (idempotency, failure modes, tests, lineage)

# **Requirements for Template for end users**

- Must guide user on exports (e.g. FOCUS exports from https://docs.aws.amazon.com/cur/latest/userguide/dataexports-create-standard.html). Default guidance: FOCUS 1.2, Parquet, Daily cadence. Hourly/Monthly exports are supported but not the default.
- Also should make system easily reproducible for any users, no local version dependency hell if possible.
- Retrieval from AWS S3 export is the default path; local `input/` drop-in remains supported.

---

# **Functional Requirements**

### **1) Ingestion (Python)**

- Single CLI command (`costlake run`)
- Accepts **FOCUS 1.2 (Parquet, default)** or **AWS CUR** -> normalizes into FOCUS
- Retrieves from AWS S3 export by default; local `input/` is optional
- Handles "no data yet" when exports are newly configured (graceful no-op + manifest)
- Writes **immutable Bronze** with manifest (idempotent, replayable)

### **2) Bronze Layer**

- Raw files stored per `snapshot_date` (Parquet)
- Manifest includes row counts, hashes, schema version, status
- Skip re-ingestion if manifest exists (`--force` overrides)

### **3) Silver Layer (FOCUS Canonical)**

- One standardized table: `silver_focus_cost_usage`
- Required fields validated (BilledCost >= 0, UsageDate valid, etc.)
- CUR -> FOCUS adapter provided
- Demo data included; user can drop in real billing files with no code changes

### **4) Gold Layer (Analytics Marts)**

- `mart_daily_cost_by_team` (safe-to-sum daily cost)
- `mart_unallocated_cost` (missing tag mapping)
- Optional: `mart_daily_cost_drivers` (top deltas)

### **5) dbt Transform + Tests**

- Unique grain tests for marts
- Required column tests
- Sanity checks (unallocated_cost <= total_cost)
- CI: `dbt build` + tests on every commit (GitHub Actions)

---

# **Non-Functional Requirements**

### **6) Failure Mode Handling**

- Schema drift detection (warn/fail on missing FOCUS columns)
- Partial ingestion recovery via manifest
- Quarantine invalid rows (bad dates, invalid costs, missing keys)
- Versioned adapters for CUR/FOCUS changes

### **7) Front-End**

- Built-in UI (Streamlit or Evidence.dev)
- Shows:
  - Daily spend
  - Team allocation
  - Unallocated spend
  - Cost drivers (deltas)
  - Audit trace (manifest -> silver -> gold)

### **8) Plug-and-Play UX**

- One folder: `input/`
- One config: `project.yml`
- One command to run: `costlake run`
- One command to demo: `make demo`

### **9) Distribution**

- Optional Docker Compose wrapper
- Works natively on any machine (Python + DuckDB)
- Sample dataset included for instant demo

---

# **Deliverables**

- GitHub repo with clear **README + architecture diagram**
- Demo dataset + demo command
- Python CLI + adapters
- dbt project (silver + gold)
- Streamlit/Evidence dashboard
- CI/CD pipeline
- Failure mode documentation (what breaks, why, how handled)
