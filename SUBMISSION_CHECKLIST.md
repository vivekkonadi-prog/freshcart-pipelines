# Submission Checklist: Data Engineering Pipeline Orchestration
**Student:** Vivek Konadi &nbsp;|&nbsp; **Roll Number:** 23bcs294 &nbsp;|&nbsp; **Repository:** `vivekkonadi-prog/freshcart-pipelines`

Every item listed below has been verified against actual project files, execution logs, and live cloud systems.

---

## Required Submission Deliverables

- [x] **GitHub repository URL included in report**  
  *Verified:* The public URL `https://github.com/vivekkonadi-prog/freshcart-pipelines` is explicitly listed on Page 1, Page 2, and in the metadata box of `report/23bcs294_Vivek_Konadi_Pipelines.pdf` and `report/RollNo_Name_Pipelines.pdf`.

- [x] **GitHub repository public or shared with instructor**  
  *Verified:* The repository is public on GitHub and accessible at `https://github.com/vivekkonadi-prog/freshcart-pipelines`.

- [x] **FreshCart notebook with visible outputs in `elt/`**  
  *Verified:* `elt/freshcart_elt.ipynb` exists (61 cells), containing full execution outputs for Baseline run (B1), record tracing (B2), unsupported currency validation (B3), Arjun SCD2 update (B4), 6th quality check experiment with failure injection (B5), and the Lakehouse streaming idempotency experiment.

- [x] **Mage project in `iris_project/`**  
  *Verified:* Complete Mage project structure exists in `iris_project/` including `metadata.yaml`, `data_loaders/load_iris.py`, `transformers/prepare_data.py`, `transformers/train_model.py`, `transformers/evaluate_model.py`, `transformers/check_no_missing.py`, and `data_exporters/save_results.py`.

- [x] **Mage notebook with visible outputs**  
  *Verified:* `iris_project/Mage_Iris_Pipeline.ipynb` exists (34 cells) with full execution outputs across cells 4 through 32 demonstrating DAG execution, test assertions, and metrics generation.

- [x] **`stream/dispatcher.py`**  
  *Verified:* Executable script exists in `stream/dispatcher.py` publishing food and ride requests every 2 seconds to topic exchange `rapido-dispatch` without hardcoded credentials.

- [x] **`stream/driver.py`**  
  *Verified:* Executable driver consumer exists in `stream/driver.py` declaring private queues, binding patterns, acknowledging valid orders with `basic_ack`, and rejecting malformed payloads with `basic_nack(requeue=False)` without hardcoded credentials.

- [x] **PDF report named `RollNo_Name_Pipelines.pdf`**  
  *Verified:* `report/RollNo_Name_Pipelines.pdf` and `report/23bcs294_Vivek_Konadi_Pipelines.pdf` exist and are generated via Microsoft Edge headless.

- [x] **Main report within eight pages, excluding screenshot appendix**  
  *Verified:* The main academic report spans **strictly 8 pages** (Pages 1 to 8), with page breaks separating topics cleanly, verified via PDF catalog inspect (`/Count [8, 8, 17]`).

- [x] **Screenshot appendix**  
  *Verified:* Pages 9 through 17 contain the full screenshot appendix covering Parts A, B, C, D, and Bonus execution evidence.

- [x] **At least five meaningful Git commits**  
  *Verified:* 21+ meaningful, conventional-commit messages exist on `main` (e.g., `feat(elt): ...`, `docs(stream): ...`, `test(ci): ...`, `feat(bonus): ...`).

- [x] **Git history included in report**  
  *Verified:* Full `git log --oneline` output is embedded directly on Page 2 of the final report.

- [x] **All required tasks have evidence**  
  *Verified:* Detailed audit recorded in `evidence/evidence_index.md` mapping tasks A1 through E to 24 verified PNG screenshots.

- [x] **Bonus implementation and evidence, if completed**  
  *Verified:* `stream/freshcart_consumer.py`, `stream/run_bonus_experiment.py`, `report/bonus_stream_to_snowflake.md`, and 3 screenshots in `screenshots/bonus/` prove 60s stream batching and two-run idempotency (Delta Rows: 0, Delta Revenue: ₹0.00).

- [x] **Viva preparation document retained for personal preparation**  
  *Verified:* `VIVA_PREPARATION.md` (and `report/VIVA_PREPARATION.md`) provides interview-ready answers, technical explanations, SQL traces, and follow-up defenses across all 5 assignment topics.

<!-- Cleaned marks verification table -->