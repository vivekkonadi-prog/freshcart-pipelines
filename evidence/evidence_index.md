# Evidence Index: Data Engineering Pipeline Orchestration
**Student:** Vivek Konadi &nbsp;|&nbsp; **Roll Number:** 23bcs294 &nbsp;|&nbsp; **Repository:** `vivekkonadi-prog/freshcart-pipelines`

---

## 1. Master Evidence Verification Index

| Requirement | Evidence Filename | What It Proves | Verification Status |
|:---|:---|:---|:---:|
| **A1: Repo Structure** | `screenshots/part_A/A1_local_structure.png` & `screenshots/part_A/A1_github_repo_page.png` | Complete repository structure on disk and public GitHub repository (`vivekkonadi-prog/freshcart-pipelines`). | ✅ **VERIFIED** |
| **A2: Git Commands** | `report/part_A2.md` & `report/RollNo_Name_Pipelines.pdf` | Comprehensive 6-command Git table (Working Folder, Staging, Local Repo, Remote Repo) and teammate visibility explanation. | ✅ **VERIFIED** |
| **A3: Green CI Run** | `screenshots/part_A/A3_github_actions_green.png` & `screenshots/part_A/A3_github_actions_fixed_green.png` | Passing GitHub Actions CI run (#1 `43688840048` and #3 `43690601305`) asserting 80/20 train/test split. | ✅ **VERIFIED** |
| **A3: Red CI Run** | `screenshots/part_A/A3_github_actions_red.png` & `screenshots/part_A/A3_github_actions_history_overview.png` | Intentionally failing CI run (#2 `43690034500`) triggered by `test_size=0.3`, producing an exit code 1 assertion failure. | ✅ **VERIFIED** |
| **B1: Baseline ELT** | `screenshots/part_B/B1_baseline_tables_verification.png` & `report/part_B1_B2.md` | Actual Snowflake query outputs: 8 raw, 6 clean, 2 quarantined, 4 dedup, 4 gold, ₹7,785 revenue, 126 loyalty points, 25% rejected. | ✅ **VERIFIED** |
| **B2: ORD-1003 Trace** | `screenshots/part_B/B2_record_trace_ord1003_ord1005.png` & `report/part_B1_B2.md` | Field name divergence (camelCase vs snake_case), SQL normalization, `QUALIFY ROW_NUMBER()` mobile survival, and ₹2,100 × 3 = 63 points proof. | ✅ **VERIFIED** |
| **B2: ORD-1005 Trace** | `screenshots/part_B/B2_record_trace_ord1003_ord1005.png` & `report/part_B1_B2.md` | Point-in-time customer SCD2 tier matching: ORD-1001 placed 2026-09-01 earns Silver 1× (12 pts); ORD-1005 placed 2026-09-05 earns Gold 2× (16 pts). | ✅ **VERIFIED** |
| **B3: Currency Rule** | `screenshots/part_B/B3_unsupported_currency_before_after.png` & `report/part_B3.md` | Addition of `unsupported_currency` to `ARRAY_CONSTRUCT_COMPACT`; ORD-1006 reasons array expanded; quarantine and gold totals unchanged. | ✅ **VERIFIED** |
| **B4: Arjun SCD2** | `report/B4_predictions_before_run.md` & `screenshots/part_B/B4_scd2_arjun_gold_update.png` | 5 predictions committed *before* run (commit `7fedb30`); all 5 matched actual Snowflake output (ORD-1002 tier=Gold mult=2 pts=70, total=161, 1 timestamp updated). | ✅ **VERIFIED** |
| **B5: 6th Quality Check** | `screenshots/part_B/B5_quality_check_normal_and_failure.png` & `report/part_B5.md` | 6th check (`loyalty_points < 0`): Normal PASS (0), injected `MOCK-FAIL-999` with -10 points PROBLEM (1), cleaned PASS (0), zero mock records remain. | ✅ **VERIFIED** |
| **C1: Mage DAG** | `screenshots/part_C/C1_C3_mage_pipeline_evidence.png` & `report/part_C1_C2.md` | Orchestrated 4-block Iris classifier DAG (`load_iris` &rarr; `check_no_missing` &rarr; `prepare_data` &rarr; `train_model`) with automated `@test` execution. | ✅ **VERIFIED** |
| **C2: Test-Gate Run** | `screenshots/part_C/C2_experiment_terminal_output.png` & `report/part_C1_C2.md` | Depth 1..5 experiment logs: Depth 1 fails gate (acc=0.633 &lt; 0.80, no CSV); Depths 2..5 pass gate (acc &ge; 0.967, `iris_predictions.csv` created). | ✅ **VERIFIED** |
| **C3: Evaluation Branch** | `screenshots/part_C/C1_C3_mage_pipeline_evidence.png` & `report/part_C3_C4.md` | Parallel evaluation branch `evaluate_model` computing confusion matrix and precision/recall/F1 table (all 1.000 for depth &ge; 3). | ✅ **VERIFIED** |
| **C4: Custom Block** | `screenshots/part_C/C4_github_ci_passed.png` & `iris_project/transformers/check_no_missing.py` | Custom data quality transformer asserting zero missing values and exactly 3 distinct species classes. | ✅ **VERIFIED** |
| **D1: Routing on Paper** | `report/D1_routing_predictions.md` & `screenshots/part_D/D1_routing_predictions_verified.png` | Theoretical topic routing predictions committed *before* run (commit `46b047a`); exact match for all 6 keys against 6 driver queues. | ✅ **VERIFIED** |
| **D2: Dispatcher/Driver**| `screenshots/part_D/D2_D3_stream_terminal_output.png` & `evidence/execution_logs/stream_execution.log` | Multi-driver execution on CloudAMQP topic exchange `rapido-dispatch`; drivers dynamically bind queues and consume matching events. | ✅ **VERIFIED** |
| **D3: ACK / NACK** | `screenshots/part_D/D2_D3_stream_terminal_output.png` & `report/part_D2_D3.md` | Malformed messages (missing `zone`) rejected via `basic_nack(requeue=False)` and discarded; valid messages acknowledged via `basic_ack`. | ✅ **VERIFIED** |
| **Bonus: Stream to Lakehouse** | `screenshots/bonus/01_rabbitmq_consumer_batch_collection.png`, `02_landing_file_and_snowflake_load.png`, `03_gold_first_batch_vs_replay_idempotency.png` | 60s stream batching into landing via `write_landing()`, Snowflake load via `load_file()`, and 2-run idempotency proof (Delta Rows: 0, Delta Revenue: ₹0.00). | ✅ **VERIFIED** |
| **Part E: Final Report** | `report/23bcs294_Vivek_Konadi_Pipelines.pdf`, `report/RollNo_Name_Pipelines.pdf`, `screenshots/part_E/02_report_page_count_proof.png` | Final academic report strictly 8 main pages, comprehensive integration diagram, half-page reflection, and 9-page appendix. | ✅ **VERIFIED** |

---

## 2. Execution Log Evidence Files

The directory `evidence/execution_logs/` contains raw, unedited logs recorded during live test runs:
- `bonus_experiment.log`: Full timestamped transcript of the Baseline check, 60s consumer collection (10 valid orders accepted, 2 malformed orders rejected), Snowflake load, Run 1 metrics, forced batch replay, Run 2 metrics, and the mathematical proof of idempotency (delta = 0).
- `stream_execution.log`: Multi-threaded runner log verifying concurrent drivers (Ravi, Arjun, Control) matching routing keys on CloudAMQP and rejecting malformed payloads with NACK.

<!-- Packaging and evidence verified -->