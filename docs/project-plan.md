# Hospital OLTP — Project Plan

## Project Overview

| Item | Detail |
|---|---|
| **Project Name** | Hospital Patient Records — OLTP Learning Project |
| **Owner** | Atul Kamble |
| **Start Date** | 2026-06-02 |
| **Goal** | Learn transactional database design, normalisation, ACID transactions, CDC, data synchronisation, and OLTP-to-OLAP migration |
| **GitHub Repo** | [github.com/AtulKamble03/hospital-oltp-project](https://github.com/AtulKamble03/hospital-oltp-project) |
| **Tech Stack** | SQL Server + SSMS + Synthea (synthetic data) |
| **Reference Project** | [COVID-19 ETL Data Warehouse](https://github.com/AtulKamble03/covid-etl-project) — OLAP counterpart |

---

## What We Are Building

A **normalised OLTP database** for a hospital — representing the kind of transactional system
that feeds an ETL pipeline like the COVID-19 warehouse we already built.

```
Synthea CSV files (synthetic patients)
         ↓
Legacy flat table (denormalised — as-is from Synthea)
         ↓  Phase 2: Migration
3NF normalised hospital database (8 tables)
         ↓  Phase 5: CDC
Change log (every insert/update/delete captured)
         ↓  Phase 6: ETL
Mini star schema (dim_patient, dim_doctor, dim_date, fact_visits)
```

---

## What We Will Learn

| # | Concept | Why It Matters |
|---|---|---|
| 1 | **Normalisation (1NF → 3NF)** | Every OLTP system is built on normalised schemas — understanding anomalies and how to fix them |
| 2 | **ACID Transactions** | Guarantees that clinical operations (admit patient, prescribe, discharge) are atomic |
| 3 | **Concurrency & Locking** | Multiple nurses/doctors accessing the same patient record simultaneously |
| 4 | **Data Migration** | Moving from a legacy flat table to a proper normalised schema — the most common real-world project |
| 5 | **CDC (Change Data Capture)** | How OLTP systems feed ETL pipelines — the source side of what we built in the warehouse |
| 6 | **OLTP vs OLAP** | Same data, two schemas, different purposes — why both exist and when to use which |
| 7 | **DB-to-DB Sync** | How EHR, lab, pharmacy systems stay in sync — message queues, replication, API sync |

---

## Data Source

**Synthea** — Synthetic Patient Data Generator by MITRE Corporation

| | Detail |
|---|---|
| **License** | Apache 2.0 — completely free, no restrictions |
| **PHI/HIPAA** | Zero — 100% synthetic patients, no real people |
| **Download** | synthea.mitre.org OR github.com/synthetichealth/synthea |
| **Format** | CSV (we use this) + JSON + FHIR |
| **What it generates** | Patients, encounters, conditions, medications, providers, organisations |
| **Why Synthea** | Same spirit as OWID for COVID project — open data, real medical codes, fake people |

---

## Tech Stack

| Tool | Purpose | Cost |
|---|---|---|
| SQL Server Developer Edition | OLTP + CDC | Free |
| SSMS | Query, manage, test | Free |
| Synthea | Synthetic patient data | Free (Apache 2.0) |
| Python (optional) | Additional data generation | Free |
| GitHub Desktop | Version control | Free |

---

## Project Phases

---

### Phase 1 — Setup and Schema Design
**Status: 🔲 Not Started**

**Goal:** Understand normalisation by designing a proper 3NF hospital schema from scratch.

**What we build:** 8 interconnected tables representing a hospital OLTP system.

#### 1.1 — Theory: Understand Normalisation
| Task | Done? |
|---|---|
| Learn what anomalies are (update, insert, delete) | 🔲 |
| Understand 1NF — atomic values, no repeating groups | 🔲 |
| Understand 2NF — no partial dependencies | 🔲 |
| Understand 3NF — no transitive dependencies | 🔲 |
| Read through `docs/learning-plan.md` — Phase 1 section | 🔲 |

#### 1.2 — Practice: Design the Schema
| Task | Done? |
|---|---|
| Create `hospital_db` database in SSMS | 🔲 |
| Write `sql/schema/create_tables.sql` — all 8 tables in 3NF | 🔲 |
| Run DDL in SSMS — verify all tables, FKs, constraints created | 🔲 |
| Draw ERD (Entity Relationship Diagram) showing all relationships | 🔲 |
| Save ERD to `docs/architecture/erd.md` | 🔲 |

**Tables to create:**
```
zip_codes          (reference table)
patients           → references zip_codes
doctors
diagnoses          (ICD-10 reference table)
visits             → references patients + doctors
visit_diagnoses    → references visits + diagnoses
medications
prescriptions      → references visits + medications
```

**Next action:** Create `hospital_db` in SSMS and run `sql/schema/create_tables.sql`

---

### Phase 2 — Load Synthea Data (Data Migration)
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 1 complete — all 8 tables exist in hospital_db.

**Goal:** Understand data migration by loading messy Synthea CSV data into the normalised schema.

#### 2.1 — Download Synthea Data
| Task | Done? |
|---|---|
| Download pre-generated Synthea dataset (100 patients) from synthea.mitre.org | 🔲 |
| Extract CSV files to `data/synthea/` folder | 🔲 |
| Open each CSV in Excel — understand what columns exist | 🔲 |
| Map Synthea columns to our 3NF tables (write mapping document) | 🔲 |

**Synthea files we use:**
```
patients.csv    → patients + zip_codes
providers.csv   → doctors
encounters.csv  → visits
conditions.csv  → visit_diagnoses + diagnoses
medications.csv → prescriptions + medications
```

#### 2.2 — Create Legacy Flat Table
| Task | Done? |
|---|---|
| Write `sql/migration/legacy_table.sql` — one big denormalised table | 🔲 |
| Load raw Synthea data into the legacy table | 🔲 |
| Profile the data — nulls, duplicates, formats, anomalies | 🔲 |
| Document findings in `docs/architecture/data-profiling.md` | 🔲 |

#### 2.3 — Migrate to 3NF
| Task | Done? |
|---|---|
| Write `sql/migration/01_load_zip_codes.sql` | 🔲 |
| Write `sql/migration/02_load_patients.sql` | 🔲 |
| Write `sql/migration/03_load_doctors.sql` | 🔲 |
| Write `sql/migration/04_load_diagnoses.sql` | 🔲 |
| Write `sql/migration/05_load_visits.sql` | 🔲 |
| Write `sql/migration/06_load_medications.sql` | 🔲 |
| Write `sql/migration/07_load_prescriptions.sql` | 🔲 |
| Write `tests/validate_migration.sql` — row counts, FK checks | 🔲 |
| Run validation — all checks pass | 🔲 |

**Key learning:** How the same data looks in a flat table vs 3NF.
Compare query complexity for "find all diagnoses for a patient" — flat vs 3NF.

---

### Phase 3 — ACID Transactions
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 2 complete — data loaded into 3NF tables.

**Goal:** Write SQL transactions that simulate real hospital operations. Understand commit and rollback.

#### 3.1 — Theory: ACID Properties
| Task | Done? |
|---|---|
| Understand Atomicity — all or nothing | 🔲 |
| Understand Consistency — data always valid | 🔲 |
| Understand Isolation — concurrent users | 🔲 |
| Understand Durability — survives crashes | 🔲 |

#### 3.2 — Practice: Write Transactions
| Task | Done? |
|---|---|
| Write `sql/transactions/patient_admission.sql` — create patient + first visit + diagnosis in one transaction | 🔲 |
| Write `sql/transactions/prescribe_medication.sql` — add prescription (rolls back if medication not found) | 🔲 |
| Write `sql/transactions/transfer_patient.sql` — change doctor, update visit record | 🔲 |
| Test rollback — introduce an error mid-transaction, confirm nothing committed | 🔲 |
| Write `sql/transactions/isolation_levels.sql` — demonstrate READ COMMITTED vs REPEATABLE READ | 🔲 |

**Key learning:** How BEGIN TRANSACTION / COMMIT / ROLLBACK works.
What happens when two nurses update the same patient record simultaneously.

---

### Phase 4 — Concurrency, Locking and Deadlocks
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 3 complete.

**Goal:** Understand how SQL Server manages multiple concurrent users — the hidden layer that makes OLTP reliable.

| Task | Done? |
|---|---|
| Run two concurrent sessions updating the same patient — observe blocking | 🔲 |
| Use `sys.dm_exec_requests` to see live blocking sessions | 🔲 |
| Create a deliberate deadlock (two sessions locking each other) | 🔲 |
| Use SQL Server Profiler or Extended Events to detect the deadlock | 🔲 |
| Write `sql/transactions/deadlock_demo.sql` | 🔲 |
| Document: what locking strategy is right for a hospital system? | 🔲 |

**Key learning:** Why READ COMMITTED is the default. When to use NOLOCK (and why it's dangerous). How deadlocks are resolved.

---

### Phase 5 — Change Data Capture (CDC)
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 3 complete — data is being inserted/updated by transactions.

**Goal:** Capture every change to patient and visit records. This is the source-side of ETL.

**Real-world connection:** In the COVID-19 project, we read CSV files as the source.
In real systems, the source is a CDC log from an OLTP database like this one.

| Task | Done? |
|---|---|
| Enable CDC on `hospital_db` | 🔲 |
| Enable CDC on `patients` table | 🔲 |
| Enable CDC on `visits` table | 🔲 |
| Enable CDC on `prescriptions` table | 🔲 |
| Insert a new patient — query CDC log to see the INSERT | 🔲 |
| Update a patient's zip code — query CDC log to see before/after values | 🔲 |
| Delete a visit — query CDC log to see the DELETE | 🔲 |
| Write `sql/cdc/setup_cdc.sql` — all CDC enable commands | 🔲 |
| Write `sql/cdc/query_changes.sql` — how to read changes since last checkpoint | 🔲 |

**Key learning:** How ETL pipelines consume CDC logs instead of full table scans.
Why this is more efficient than SELECT * every hour.

---

### Phase 6 — OLTP to OLAP (Mini Data Warehouse)
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 5 complete.

**Goal:** Build a simple star schema FROM the OLTP tables. Connect both projects.

**What we build:**
```
dim_patient    (from patients table)
dim_doctor     (from doctors table)
dim_date       (generated, same as COVID project)
fact_visits    (from visits + visit_diagnoses)
```

| Task | Done? |
|---|---|
| Create `hospital_dw` database in SSMS | 🔲 |
| Write `sql/analytical/create_star_schema.sql` | 🔲 |
| Write ETL SQL to populate star schema from OLTP tables | 🔲 |
| Write `sql/analytical/report_queries.sql` — 5 analytical questions | 🔲 |
| Compare: same question in 3NF (how many joins?) vs star schema | 🔲 |
| Document: why the warehouse schema is simpler for analytics | 🔲 |

**Key learning:** Why the same data needs two different schemas.
The star schema you build here is the same Kimball pattern as the COVID-19 project.

---

### Phase 7 — Database Migration Scenarios
**Status: 🔲 Not Started**

**Pre-requisite:** Phase 6 complete — full hospital system working end-to-end.

**Goal:** Simulate a real migration scenario — moving data between two databases.

#### Scenario A — Schema Migration (add columns)
| Task | Done? |
|---|---|
| Add `blood_type` column to patients — without breaking existing data | 🔲 |
| Add `is_emergency` flag to visits | 🔲 |
| Write migration script with rollback capability | 🔲 |

#### Scenario B — Platform Migration (SQL Server → SQL Server)
| Task | Done? |
|---|---|
| Create `hospital_db_v2` (simulates new environment) | 🔲 |
| Script entire schema with data | 🔲 |
| Validate: row counts match, FKs intact, constraints preserved | 🔲 |

#### Scenario C — Sync Between Two OLTP Databases
| Task | Done? |
|---|---|
| Create `hospital_db_branch` (simulates a second hospital branch) | 🔲 |
| Write SQL replication script — sync new patients from branch to main | 🔲 |
| Handle conflicts: same patient in both databases | 🔲 |

---

## Skills You Will Learn

| Skill | Phase |
|---|---|
| 1NF, 2NF, 3NF normalisation — identify and fix anomalies | Phase 1 |
| SQL DDL — CREATE TABLE with FK, CHECK, UNIQUE constraints | Phase 1 |
| Data profiling — understand source data before migration | Phase 2 |
| Data migration SQL — INSERT INTO ... SELECT with transformations | Phase 2 |
| BEGIN TRANSACTION / COMMIT / ROLLBACK | Phase 3 |
| ACID properties — practical understanding | Phase 3 |
| Transaction isolation levels — READ COMMITTED, SERIALIZABLE | Phase 4 |
| Locking — shared vs exclusive, blocking detection | Phase 4 |
| Deadlock — detection and resolution | Phase 4 |
| CDC — enable, query change log, understand LSN | Phase 5 |
| OLTP → OLAP ETL via SQL (no SSIS needed) | Phase 6 |
| Star schema from OLTP — Kimball pattern revisited | Phase 6 |
| Schema migration with rollback | Phase 7 |
| Database synchronisation between OLTP instances | Phase 7 |

---

## How This Connects to the COVID-19 Project

```
COVID-19 project (what we built):
  Synthea/OWID CSV → SSIS ETL → Star Schema (OLAP) → Power BI

Hospital project (what we're building):
  Synthea CSV → Legacy Flat Table → 3NF OLTP → CDC → Mini Star Schema
                                                  ↑
                             This is the SOURCE SIDE of the ETL pipeline
```

When you work on an ETL project at Veradigm, the data starts in a system
like the hospital database you're building here. Understanding both sides
makes you a complete data engineer — not just someone who moves data,
but someone who understands where it comes from and why it's structured that way.
