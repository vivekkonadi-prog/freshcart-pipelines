# Viva Voce Preparation Guide: Data Engineering Pipeline Orchestration
**Student:** Vivek Konadi &nbsp;|&nbsp; **Roll Number:** 23bcs294 &nbsp;|&nbsp; **Repository:** `vivekkonadi-prog/freshcart-pipelines`

---

## Topic A: Git, GitHub & Continuous Integration (CI/CD)

### Q1. What is the difference between the Working Directory, the Staging Area, the Local Repository, and the Remote Repository?
- **Short Interview-Ready Answer:**
  "The **Working Directory** is the sandbox of files you are currently modifying on disk. The **Staging Area (Index)** is the preparation buffer where you select specific changes via `git add`. The **Local Repository** (`.git/`) stores committed, immutable snapshots locally on your machine. The **Remote Repository** (GitHub) is the centralized, shared copy accessible by the entire team."
- **Deeper Explanation:**
  In a Distributed Version Control System (DVCS), every developer holds a complete history. Files transition through four states:
  1. *Untracked/Modified* in Working Tree &rarr; 2. *Staged (Index)* via `git add` &rarr; 3. *Committed (HEAD)* via `git commit` &rarr; 4. *Pushed (Remote)* via `git push`.
- **Implementation Example:**
  In `freshcart-pipelines`, running `git status` after editing `stream/dispatcher.py` showed it in red under "Changes not staged for commit". Running `git add stream/dispatcher.py` moved the blob into the index, and `git commit` sealed commit `c4bdb3c` locally without sending network packets.
- **Why Design Chosen:** Decoupling staging from committing allows atomic, well-scoped commits rather than bundling unrelated work together.
- **Likely Follow-up Question:** *"Can your teammate see your code after you run `git commit`?"*
  - **Answer:** *"No. `git commit` only updates the local object database in `.git/objects` and moves the local branch pointer `refs/heads/main`. No network traffic occurs until `git push origin main` uploads the packfiles to GitHub."*

---

### Q2. How does the GitHub Actions CI workflow detect pipeline failures?
- **Short Interview-Ready Answer:**
  "The CI runner monitors the shell exit status code of the test command. If `pytest` exits with code `0`, the job passes (green checkmark). If any assertion fails or an uncaught exception is raised, `pytest` returns an exit code of `1` (or higher), causing the GitHub Actions step to fail with a red cross."
- **Deeper Explanation:**
  In `.github/workflows/ci.yml`, the job runs on an `ubuntu-latest` container with Python 3.12. The runner executes `pytest tests/`. In Unix shells, step success depends on the process exit code (`$? == 0`).
- **Implementation Example (Red vs Fix Green Runs):**
  - In Run #1 (commit `fa158d2`, Run ID `43688840048`), `test_size=0.2` produced 120 train and 30 test records, passing all 3 assertions (Green).
  - In Run #2 (commit `6294a30`, Run ID `43690034500`), we intentionally set `test_size=0.3`. The assertion `assert (df['split'] == 'train').sum() == 120` raised `AssertionError: assert 105 == 120`, exiting with code `1` and failing the CI run (Red).
  - In Run #3 (commit `fa158d2`, Run ID `43690601305`), restoring `test_size=0.2` restored exit code `0` (Green).
- **Likely Follow-up Question:** *"Why is CI essential before merging into `main`?"*
  - **Answer:** *"It enforces branch protection gates automatically, preventing broken dependencies, syntax regressions, and schema mismatches from polluting the production codebase."*

---

## Topic B: Snowflake & Multi-Hop ELT (Bronze, Silver, Gold)

### Q1. What is the fundamental difference between ETL and ELT?
- **Short Interview-Ready Answer:**
  "In **ETL**, data is extracted, transformed on an external compute server (e.g., Python/Spark), and only the final cleaned data is loaded. In **ELT**, raw data is immediately loaded into scalable cloud storage (Snowflake Bronze) as raw JSON, and all cleaning, deduplication, and business logic are executed in-place using declarative SQL."
- **Deeper Explanation:**
  ELT separates ingestion compute from transformation compute and preserves raw history. If a business rule changes (like customer tier multipliers or currency validation), ELT allows re-processing the immutable Bronze layer without needing to re-extract from source APIs.
- **Implementation Example:**
  In `freshcart-pipelines`, `bronze.orders_raw` stores raw JSON payloads as `VARIANT`. When we added `unsupported_currency` in Part B3 or updated Arjun's SCD2 tier in Part B4, we did not call web/mobile APIs again; we simply re-ran `SQL_10` through `SQL_40` directly against Bronze.

---

### Q2. Walk me through the exact trace for ORD-1003. Why did the mobile record survive?
- **Short Interview-Ready Answer:**
  "ORD-1003 had two raw records: a web order with null amount (ingest sequence 2) and a mobile order with ₹2,100 (ingest sequence 8). Both had the same timestamp. In `silver.orders_dedup`, `QUALIFY ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY updated_at DESC, ingest_seq DESC) = 1` picked the mobile record because its higher `ingest_seq` (8 vs 2) broke the tie. In Gold, customer Meera Patel's Platinum tier (3×) yielded ₹2,100 revenue and 63 loyalty points."
- **Detailed Step-by-Step SQL Trace:**
  1. **Bronze Divergence:**
     - Web record: `raw:order_id = 'ORD-1003'`, `raw:amount = NULL`, `ingest_seq = 2`.
     - Mobile record: `raw:orderId = 'ORD-1003'`, `raw:total.value = '2100.00'`, `ingest_seq = 8`.
  2. **Normalization in `silver.orders_checked`:**
     - SQL unions both sources, mapping `raw:order_id` and `raw:orderId` to unified column `order_id`, and `raw:amount` and `raw:total.value` to decimal `amount`.
  3. **Deduplication in `silver.orders_dedup`:**
     - Both rows share `updated_at = '2026-09-02 11:45:00'`.
     - The `ORDER BY updated_at DESC, ingest_seq DESC` clause evaluates `ingest_seq=8` ahead of `ingest_seq=2`. The mobile record receives row number 1 and survives.
  4. **Gold Calculations in `SQL_40_GOLD`:**
     - Meera Patel (`meera@outlook.com`) is Platinum tier &rarr; Multiplier = **3**.
     - FX rate for INR on 2026-09-02 = **1.0**.
     - $\text{amount\_inr} = 2,100.00 \times 1.0 = \text{₹}2,100.00$.
     - $\text{loyalty\_points} = \lfloor 2,100 / 100 \rfloor \times 3 = 21 \times 3 = 63\text{ points}$.

---

### Q3. Why did ORD-1005 earn points using the Gold multiplier while ORD-1001 used Silver?
- **Short Interview-Ready Answer:**
  "Both orders belong to Priya Raman (`CUST-01`). In `dim.customers_scd2`, Priya was Silver until `2026-09-02`, and Gold from `2026-09-03` onward. Because ORD-1001 was placed on `2026-09-01`, it matched the Silver window (1× multiplier &rarr; 12 points). Because ORD-1005 was placed on `2026-09-05`, it matched the Gold window (2× multiplier &rarr; 16 points)."
- **SQL Logic:**
  ```sql
  LEFT JOIN dim.customers_scd2 c
    ON c.email = o.email
   AND o.order_date BETWEEN c.valid_from AND c.valid_to
  ```
- **Arithmetic:**
  - ORD-1001: Amount ₹1,250 &rarr; $\lfloor 1250/100 \rfloor \times 1 = 12$ points.
  - ORD-1005: Amount ₹899 &rarr; $\lfloor 899/100 \rfloor \times 2 = 8 \times 2 = 16$ points.

---

### Q4. How does the Gold layer achieve idempotency using `row_hash`?
- **Short Interview-Ready Answer:**
  "In `SQL_40_GOLD`, the MERGE statement hashes all 14 business attributes into a single integer `row_hash = HASH(...)`. When matching on `order_id`, Snowflake updates only `WHEN MATCHED AND t.row_hash <> s.row_hash`. If the incoming record has identical data, the hashes match, so 0 rows are updated and 0 rows are inserted, preventing duplicate revenue or points."
- **Implementation Proof (Part B4):**
  When Arjun's tier changed from Silver to Gold, re-running the ELT pipeline updated *only ORD-1002* because its tier, multiplier, and points changed (altering its hash). For ORD-1001, ORD-1003, and ORD-1005, hashes were unchanged, so their `published_at` timestamp was untouched.

---

## Topic C: ML Pipelines & In-DAG Test Gates (Mage AI)

### Q1. What is the architecture of the Mage Iris pipeline?
- **Short Interview-Ready Answer:**
  "It is an orchestrated Directed Acyclic Graph (DAG) defined in `iris_project/metadata.yaml`:
  `load_iris` (data loader) &rarr; `check_no_missing` (data quality transformer) &rarr; `prepare_data` (80/20 split transformer) &rarr; `train_model` (Decision Tree transformer). Downstream of `train_model`, the graph forks into two parallel blocks: `save_results` (exporter) and `evaluate_model` (metrics & confusion matrix transformer)."

---

### Q2. How does a failing `@test` stop downstream execution, and why is this superior to checking files afterwards?
- **Short Interview-Ready Answer:**
  "When `train_model` finishes, Mage immediately executes its attached `@test` function before resolving downstream dependencies. When `max_depth=1`, the model underfits (accuracy 63.3%), raising an `AssertionError: assert 0.633 >= 0.80`. Mage catches this exception, marks `train_model` as **FAILED**, and cancels downstream traversal. Consequently, `save_results` never executes, preventing `iris_predictions.csv` from ever being created."
- **Why This Is Superior to Post-Hoc File Auditing:**
  1. **Prevents Data Poisoning:** Bad predictions are never published to object storage, downstream warehouses, or live feature stores.
  2. **Eliminates Rollback Overhead:** No compensation scripts or deletes are needed because corrupt artifacts were never written.
  3. **Fail-Fast Observability:** The pipeline halts at the exact source of failure (`train_model`), surfacing clean stack traces immediately.

---

### Q3. What were the exact accuracy results across `max_depth = 1..5`?
- **Interview Table:**
  - `max_depth = 1`: Accuracy **0.633** &rarr; Gate <span class="badge badge-fail">FAIL</span> &rarr; `iris_predictions.csv` **NOT CREATED**.
  - `max_depth = 2`: Accuracy **0.967** &rarr; Gate <span class="badge badge-pass">PASS</span> &rarr; `iris_predictions.csv` **CREATED**.
  - `max_depth = 3`: Accuracy **1.000** &rarr; Gate <span class="badge badge-pass">PASS</span> &rarr; `iris_predictions.csv` **CREATED**.
  - `max_depth = 4`: Accuracy **1.000** &rarr; Gate <span class="badge badge-pass">PASS</span> &rarr; `iris_predictions.csv` **CREATED**.
  - `max_depth = 5`: Accuracy **1.000** &rarr; Gate <span class="badge badge-pass">PASS</span> &rarr; `iris_predictions.csv` **CREATED**.

---

## Topic D: Real-Time Streaming & RabbitMQ

### Q1. What is the difference between `*` and `#` in RabbitMQ topic routing?
- **Short Interview-Ready Answer:**
  "In an AMQP topic exchange, words are delimited by dots (`.`). The asterisk (`*`) matches **strictly one word**. The hash symbol (`#`) matches **zero or more words**."
- **Application to the 6th Routing Key (`bangalore.koramangala`):**
  - Why it failed `bangalore.koramangala.*`: The key has only 2 words. The pattern requires a 3rd word for `*`. Because no 3rd word exists, it fails.
  - Why it matched `bangalore.#`: The `#` symbol matches zero or more words. Here, `#` matches the single word `koramangala` (or zero remaining words), so the message was delivered exclusively to Control.

---

### Q2. Explain ACK and NACK behavior. What happens if a consumer crashes before sending an ACK?
- **Short Interview-Ready Answer:**
  "A consumer sends `basic_ack` to confirm successful processing, allowing RabbitMQ to delete the message from the queue. If a message is malformed (missing `zone`), the consumer sends `basic_nack(requeue=False)` to permanently discard it. If a consumer crashes before ACKing, RabbitMQ detects socket closure and re-queues the message at the head of the queue for another active worker."
- **The Poison Pill Scenario (`requeue=True`):**
  - If a malformed message causes an unhandled crash and is rejected with `requeue=True`, RabbitMQ immediately pushes the same malformed message back to the consumer, which crashes again. This creates an infinite, resource-exhausting crash loop called a **poison pill**. Setting `requeue=False` or utilizing a Dead Letter Exchange (DLX) is essential.

---

## Topic E: Lakehouse Stream Integration (Bonus)

### Q1. How does the RabbitMQ consumer connect to the Snowflake Lakehouse?
- **Short Interview-Ready Answer:**
  "`stream/freshcart_consumer.py` binds to `#.food` on `rapido-dispatch`. It buffers valid food delivery orders for a 60-second window, rejects malformed orders via NACK, and writes the batch using `write_landing(source='web')` as a compressed JSON Lines file in `freshcart_landing/web/`. It then uploads the file via Snowflake `PUT` and ingests via `COPY INTO bronze.orders_raw`, followed by executing the Silver and Gold SQL pipeline."

---

### Q2. How did you prove that replaying the stream batch does not double-count Gold totals?
- **Short Interview-Ready Answer:**
  "We ran an automated two-pass experiment in `stream/run_bonus_experiment.py`. In Run 1, 10 valid streamed orders were ingested: Gold orders rose from 4 to 14, and revenue rose from ₹7,785.00 to ₹26,185.00 (+₹18,400.00). In Run 2, we forced Snowflake to reload the exact same landing file into Bronze with `FORCE = TRUE` and re-ran the ELT pipeline. The results were identical: exactly 14 rows, ₹26,185.00 revenue, and 596 points. The delta was strictly **0 rows, ₹0.00 revenue, and 0 points**, proving complete idempotency."
- **Why It Did Not Double:**
  - `silver.orders_dedup` filtered the replayed records using `QUALIFY ROW_NUMBER()`.
  - `gold.fct_orders` evaluated `WHEN MATCHED AND t.row_hash <> s.row_hash`. Because incoming row hashes matched existing rows, 0 updates and 0 inserts occurred.

---

## Engineering Defense: Summary of Design Decisions & Limitations

| Component | What Was Implemented | Why Design Was Chosen | Limitations Remaining |
|:---|:---|:---|:---|
| **Git / CI** | 19+ atomic commits, pytest suite, GitHub Actions | Enforces automated test gates on Python 3.12 prior to merge | Requires active internet connection to GitHub runners |
| **Snowflake ELT** | Multi-hop Bronze &rarr; Silver &rarr; Dim/Ref &rarr; Gold | Preserves raw JSON data; in-place SQL transformations enable zero-reextraction schema changes | Dependent on cloud warehouse compute credits |
| **Mage ML** | 4-block DAG + parallel evaluation branch + custom test block | Prevents bad models from exporting predictions via in-DAG `@test` assertion | Evaluates Decision Trees on a single node (not distributed Spark) |
| **RabbitMQ** | Topic exchange (`rapido-dispatch`) + HTTP API fallback | Guarantees reliability when enterprise firewalls block raw AMQP ports 5671/5672 | HTTP polling has slightly higher latency (~100ms) than long-lived TCP socket streaming |
| **Bonus Lakehouse** | 60s micro-batching consumer into landing + COPY INTO | Demonstrates Lambda/Kappa micro-batch bridge from streaming to data warehouse | Micro-batch interval is 60 seconds rather than sub-second real-time streaming |
