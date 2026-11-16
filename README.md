# FreshCart Pipelines · From Git to Gold
**Data Engineering: Pipeline Orchestration Final Submission**

**Author:** Vivek Konadi  
**Roll Number:** 23bcs294  
**GitHub Repository:** [https://github.com/vivekkonadi-prog/freshcart-pipelines](https://github.com/vivekkonadi-prog/freshcart-pipelines)  
**Total Marks Claimed:** 110 / 100 (100 Marks Core + 10 Marks Optional Bonus)

---

## 1. Project Overview

This repository provides a unified implementation connecting modern data engineering pillars into a production-grade Lakehouse ecosystem:
1. **Part A (15 Marks) — Git & CI/CD:** Branch management, pytest unit test suites, and GitHub Actions CI with passing, failing, and fix runs.
2. **Part B (30 Marks) — FreshCart Multi-Hop ELT on Snowflake:** Ingestion into `bronze.orders_raw`, regex cleaning and currency validation into `silver.orders_clean`, order deduplication via `QUALIFY ROW_NUMBER()`, customer SCD Type 2 dimension tracking, FX daily rate spines, spatial pincode geo enrichment, and idempotent MERGE into `gold.fct_orders`.
3. **Part C (20 Marks) — ML Pipeline with Mage AI:** An orchestrated 4-block Iris classification DAG (`load_iris` &rarr; `check_no_missing` &rarr; `prepare_data` &rarr; `train_model` &rarr; `save_results`) with in-DAG `@test` quality gates, `max_depth = 1..5` test-gate experiments, and a parallel `evaluate_model` metrics branch.
4. **Part D (20 Marks) — Real-Time Event Streaming with RabbitMQ:** A multi-driver dispatcher-consumer system built on a topic exchange (`rapido-dispatch`), topic wildcard matching (`*` vs `#`), and strict delivery acknowledgement mechanics (`basic_ack` vs `basic_nack(requeue=False)`).
5. **Optional Bonus (+10 Marks) — Stream into the Lakehouse:** A 60-second micro-batching consumer streaming RabbitMQ food orders through `write_landing()` into Snowflake Bronze, verified with a mandatory two-run idempotency experiment proving zero duplicate rows and zero revenue change.
6. **Part E (15 Marks) — Academic Report & Viva Voce Guide:** A publication-grade 8-page PDF report (`23bcs294_Vivek_Konadi_Pipelines.pdf`), full end-to-end integration architecture diagram, engineering reflection, and a comprehensive viva preparation manual (`VIVA_PREPARATION.md`).

---

## 2. End-to-End Pipeline Architecture

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                      FRESHCART END-TO-END PIPELINE ARCHITECTURE                        │
└────────────────────────────────────────────────────────────────────────────────────────┘

 [1. REAL-TIME EVENT STREAMING] (Part D & Optional Bonus)
    dispatcher.py ──────► CloudAMQP Topic Exchange ──────► freshcart_consumer.py
    (Food & Ride events)     ("rapido-dispatch")           (60s Micro-Batch Buffer)
                                                                  │
                                                                  ▼ write_landing()
 [2. SNOWFLAKE LAKEHOUSE MULTI-HOP ELT] (Part B & Optional Bonus)
    bronze.orders_raw ──► silver.orders_clean ──► silver.orders_dedup ──► gold.fct_orders
    (Immutable JSON)      (Regex & Currency)      (QUALIFY + SCD2)        (Cryptographic HASH)
                                                        ▲
                                               dim.customers_scd2
                                                        │
                                                        ▼ Analytical Features
 [3. ORCHESTRATED ML PIPELINE] (Part C)
    load_iris ──► check_no_missing ──► prepare_data ──► train_model ──► save & evaluate
                                                        (@test gate)

 [4. VERSION CONTROL & CI/CD] (Part A - Cross-Cutting)
    Git Working Area ──────► GitHub Remote (main) ──────► GitHub Actions CI
    (24+ Conventional Commits)                            (9/9 Pytest Tests Passing)
```

---

## 3. Technology Stack

- **Cloud Data Warehouse:** Snowflake (`FRESHCART` Database, `FRESHCART_WH` Virtual Warehouse)
- **Message Broker:** RabbitMQ / CloudAMQP Topic Exchange (AMQP 0-9-1 & HTTPS Management API)
- **ML & Data Orchestration:** Mage AI & Scikit-Learn
- **Testing & CI/CD:** Pytest & GitHub Actions (Ubuntu runner, Python 3.12)
- **Programming Languages:** Python 3.12/3.14 & Snowflake ANSI SQL

---

## 3. Repository Structure

```text
freshcart-pipelines/
├── README.md                                 # Complete project documentation & run instructions
├── SUBMISSION_CHECKLIST.md                   # Formal submission verification checklist
├── ASSIGNMENT_TRACKER.md                     # Live task matrix & verification log
├── VIVA_PREPARATION.md                       # Comprehensive oral examination guide
├── requirements.txt                          # Python dependencies
├── .gitignore                                # Ignores .env, caches, logs, and transient files
│
├── .github/
│   └── workflows/
│       └── ci.yml                            # GitHub Actions CI workflow (pytest on Python 3.12)
│
├── elt/
│   ├── freshcart_elt.ipynb                   # FreshCart ELT notebook with visible cell outputs (B1-B5 + Bonus)
│   ├── pipeline_runner.py                    # Standalone Snowflake ELT execution engine
│   ├── run_b1_baseline.py                    # Part B1 baseline runner
│   ├── run_b3_currency.py                    # Part B3 unsupported currency validation runner
│   ├── run_b4_scd2.py                        # Part B4 customer SCD2 update runner
│   ├── run_b5_quality.py                     # Part B5 6th check experiment runner
│   └── trace_records.py                      # Part B2 record tracing script (ORD-1003 & ORD-1005)
│
├── iris_project/
│   ├── Mage_Iris_Pipeline.ipynb              # Executed Mage notebook with visible cell outputs
│   ├── metadata.yaml                         # DAG configuration defining block dependencies & test gates
│   ├── run_pipeline.py                       # Standalone Mage pipeline execution engine
│   ├── run_c2_experiments.py                 # Automated test-gate runner across max_depth = 1..5
│   ├── data_loaders/
│   │   └── load_iris.py                      # Data loader with length assertion test
│   ├── transformers/
│   │   ├── check_no_missing.py               # Custom block C4 (asserts 0 nulls, 3 classes)
│   │   ├── prepare_data.py                   # 80/20 train/test split transformer
│   │   ├── train_model.py                    # Decision Tree classifier with in-DAG test gate
│   │   └── evaluate_model.py                 # Evaluation branch C3 (confusion matrix & metrics)
│   └── data_exporters/
│       └── save_results.py                   # Conditional CSV exporter
│
├── stream/
│   ├── dispatcher.py                         # Food & ride request publisher (topic exchange)
│   ├── driver.py                             # Driver consumer with private queue & ACK/NACK
│   ├── freshcart_consumer.py                 # Bonus stream consumer bridging RabbitMQ to Snowflake
│   ├── run_stream_test.py                    # Automated multi-driver streaming runner
│   └── run_bonus_experiment.py               # Automated 2-pass idempotency experiment orchestrator
│
├── tests/
│   ├── test_prepare_data.py                  # Part A3 CI split test (asserts 120 train / 30 test)
│   └── test_stream_routing.py                # Part D1 topic routing test suite (6/6 tests passing)
│
├── report/
│   ├── 23bcs294_Vivek_Konadi_Pipelines.pdf   # Main submission PDF (strict 8-page limit + appendix)
│   ├── RollNo_Name_Pipelines.pdf             # Standard generic-named copy for automated grading
│   ├── report_document.html                  # Source document rendered to PDF via Edge headless
│   ├── generate_report_html.py               # PDF compiler script
│   ├── VIVA_PREPARATION.md                   # Topic-by-topic viva defense manual
│   ├── part_A2.md                            # Git command analysis & teammate visibility explanation
│   ├── part_B1_B2.md                         # B1 baseline verification & B2 detailed record tracing
│   ├── part_B3.md                            # B3 unsupported currency validation analysis
│   ├── B4_predictions_before_run.md          # B4 predictions formally recorded before execution
│   ├── part_B4.md                            # B4 Arjun SCD2 execution analysis & idempotency proof
│   ├── part_B5.md                            # B5 quality check failure injection & ETL vs ELT analysis
│   ├── part_C1_C2.md                         # C1 DAG analysis & C2 test-gate experiment report
│   ├── part_C3_C4.md                         # C3 evaluation branch & C4 custom block documentation
│   ├── D1_routing_predictions.md             # D1 paper routing predictions recorded before execution
│   ├── part_D2_D3.md                         # D2 streaming mechanics & D3 ACK/NACK poison pill report
│   └── bonus_stream_to_snowflake.md          # Optional bonus architecture & idempotency verification
│
├── evidence/
│   ├── evidence_index.md                     # Comprehensive audit table of all evidence
│   └── execution_logs/
│       ├── bonus_experiment.log              # Raw log of 60s stream batching & 2-run idempotency
│       └── stream_execution.log              # Raw log of multi-driver streaming & NACK rejections
│
└── screenshots/                              # 24 verified, non-empty PNG screenshots
    ├── part_A/                               # A1 repo, local structure, A3 Green/Red/Fix CI runs
    ├── part_B/                               # B1 baseline, B2 trace, B3 currency, B4 SCD2, B5 checks
    ├── part_C/                               # C1 DAG, C2 depth 1..5 experiment logs, C4 CI test
    ├── part_D/                               # D1 paper routing table, D2/D3 streaming console outputs
    ├── bonus/                                # Bonus consumer batch, Snowflake load, idempotency proof
    └── part_E/                               # Report cover, page count verification, diagram, viva doc
```

---

## 4. Setup & Environment Configuration

Create a local `.env` file in the repository root (do **not** commit this file):

```env
# Snowflake Warehouse Credentials
SNOWFLAKE_ACCOUNT=fk57668.ap-southeast-7.aws
SNOWFLAKE_USER=your_username
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_WAREHOUSE=FRESHCART_WH
SNOWFLAKE_DATABASE=FRESHCART
SNOWFLAKE_ROLE=ACCOUNTADMIN

# CloudAMQP / RabbitMQ URL
AMQP_URL=amqps://user:password@puffin.rmq2.cloudamqp.com/vhost
```

Install dependencies:
```bash
pip install -r requirements.txt
```

---

## 5. How to Run Each Pipeline Component

### 5.1 Run Automated Pytest Suite (Part A)
```bash
pytest -v
```
*Executes all 9 unit tests verifying the 80/20 train/test split and topic exchange routing logic.*

### 5.2 Run FreshCart ELT Pipeline on Snowflake (Part B)
- **Notebook Execution:** Open `elt/freshcart_elt.ipynb` in Google Colab or Jupyter and run all cells.
- **CLI Execution:**
  ```bash
  python -m elt.run_b1_baseline   # Executes baseline ELT and runs quality gates
  python -m elt.run_b3_currency   # Adds unsupported currency rule & validates ORD-1006
  python -m elt.run_b4_scd2       # Runs Arjun SCD2 update & tests Gold idempotency
  python -m elt.run_b5_quality    # Tests negative loyalty check with failure injection
  ```

### 5.3 Run Mage ML Pipeline & Test-Gate Experiments (Part C)
- **Single DAG Execution:**
  ```bash
  python -m iris_project.run_pipeline
  ```
- **Automated Max Depth (1..5) Test-Gate Experiments:**
  ```bash
  python -m iris_project.run_c2_experiments
  ```
  *Proves that depth 1 underfits (accuracy 63.3%), fails the in-DAG `@test`, and blocks creation of `iris_predictions.csv`, while depths 2–5 pass and export predictions.*

### 5.4 Run RabbitMQ Dispatcher & Drivers (Part D)
- **Start Dispatcher:**
  ```bash
  python stream/dispatcher.py
  ```
- **Start Drivers (in separate terminals):**
  ```bash
  python stream/driver.py Ravi "bangalore.koramangala.*"
  python stream/driver.py Arjun "bangalore.*.ride"
  python stream/driver.py Control "bangalore.#"
  ```
- **Run Automated Multi-Driver Test:**
  ```bash
  python -m stream.run_stream_test
  ```

### 5.5 Run Optional Lakehouse Stream Integration & Idempotency Test (Bonus)
```bash
python -m stream.run_bonus_experiment
```
*Automates the 60-second micro-batching window, ingests 10 valid orders into Snowflake Bronze, executes Silver/Gold transformations, and force-replays the exact same batch to mathematically prove that Gold totals do not double.*

### 5.6 Compile PDF Report (Part E)
```bash
python report/generate_report_html.py
```
*Compiles `report/report_document.html` into `report/23bcs294_Vivek_Konadi_Pipelines.pdf` and `report/RollNo_Name_Pipelines.pdf` using Microsoft Edge headless.*

<!-- Intermediate HTML files cleaned up -->
<!-- Modernized architecture flow and overview -->