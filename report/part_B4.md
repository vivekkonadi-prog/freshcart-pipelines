# Part B4: SCD Type 2 Dimension Change, Predictions & Idempotency Analysis

**Date:** 2026-11-15  
**Platform:** Snowflake Data Cloud  
**Database:** `FRESHCART` | **Schemas:** `dim`, `silver`, `gold`

---

## 1. Description of the SCD2 CRM Event
Customer Arjun K (`CUST-02`, `arjun.k@yahoo.in`) was upgraded from Silver to Gold tier effective `2026-08-15`:
```json
{
  "cust_id": "CUST-02",
  "email": "arjun.k@yahoo.in",
  "tier": "Gold",
  "effective_from": "2026-08-15"
}
```

This event was appended to the CRM ingest feed and processed into `dim.customers_scd2`.

---

## 2. Pre-Execution Predictions vs Actual Snowflake Results

As mandated by academic integrity guidelines, predictions were committed to Git (**Commit [`7fedb30`](https://github.com/vivekkonadi-prog/freshcart-pipelines/commit/7fedb30)**) **prior** to running the modified pipeline.

| Metric / Quantity | Predicted Value | Actual Snowflake Value | Verification Status |
|:---|:---:|:---:|:---:|
| **Rows for `CUST-02` in `dim.customers_scd2`** | **2** | **2** | ✅ **EXACT MATCH** |
| **Tier used for `ORD-1002`** | **Gold** | **Gold** | ✅ **EXACT MATCH** |
| **Loyalty points for `ORD-1002`** | **70** | **70** | ✅ **EXACT MATCH** |
| **Total loyalty points in Gold** | **161** | **161** | ✅ **EXACT MATCH** |
| **Number of Gold rows whose `published_at` changed** | **1** | **1** | ✅ **EXACT MATCH** |

---

## 3. Query Evidence & Verification Logs

### A. Customer SCD2 History (`dim.customers_scd2`)
```sql
SELECT cust_id, email, tier, valid_from, valid_to, is_current 
FROM dim.customers_scd2 
WHERE cust_id = 'CUST-02' 
ORDER BY valid_from;
```
**Output:**
```text
CUST_ID            EMAIL   TIER VALID_FROM   VALID_TO  IS_CURRENT
CUST-02 arjun.k@yahoo.in Silver 2024-01-15 2026-08-14       False
CUST-02 arjun.k@yahoo.in   Gold 2026-08-15 9999-12-31        True
```
*Arjun's original Silver record now terminates cleanly on `2026-08-14`, and his active Gold record begins on `2026-08-15`.*

### B. Updated Order ORD-1002 in `gold.fct_orders`
```sql
SELECT order_id, order_date, cust_id, tier, multiplier, amount, currency, fx_rate, amount_inr, loyalty_points 
FROM gold.fct_orders 
WHERE order_id = 'ORD-1002';
```
**Output:**
```text
ORDER_ID ORDER_DATE CUST_ID TIER  MULTIPLIER AMOUNT CURRENCY FX_RATE AMOUNT_INR  LOYALTY_POINTS
ORD-1002 2026-09-01 CUST-02 Gold           2  42.50      USD 83.2000    3536.00              70
```
*`ORD-1002` (`2026-09-01`) joined to Arjun's active Gold tier (multiplier = 2), doubling points from 35 to 70.*

### C. Published Timestamp (`published_at`) Delta Inspection
```text
ORDER_ID           PUBLISHED_AT_BEFORE            PUBLISHED_AT_AFTER  CHANGED
ORD-1001 2026-11-15 09:25:23.475-08:00 2026-11-15 09:25:23.475-08:00    False
ORD-1002 2026-11-15 09:25:23.475-08:00 2026-11-15 09:29:11.976-08:00     True
ORD-1003 2026-11-15 09:25:23.475-08:00 2026-11-15 09:25:23.475-08:00    False
ORD-1005 2026-11-15 09:25:23.475-08:00 2026-11-15 09:25:23.475-08:00    False
```

---

## 4. Idempotency Explanation (4–6 Sentences)

1. In FreshCart's ELT architecture, every enriched order in `silver.orders_enriched` computes a deterministic `row_hash` using Snowflake's cryptographic `HASH(...)` function across all business columns (including `tier`, `multiplier`, `amount_inr`, and `loyalty_points`).
2. The `MERGE INTO gold.fct_orders` statement matches records strictly on primary key `order_id` and executes an `UPDATE` only when `t.row_hash <> s.row_hash`, inserting only when `NOT MATCHED`.
3. Consequently, re-running the transformation on unchanged source records evaluates to a complete no-op in Gold: existing row hashes match, preventing record duplication, point inflation, or false timestamp refreshes.
4. When an underlying upstream dimension attribute legitimately changes (such as Arjun's tier upgrading to Gold), `s.row_hash` changes, triggering a surgical in-place update of that single order while leaving all other orders untouched.
5. In contrast, the Bronze layer is an append-only, immutable audit ledger that captures raw arrival envelopes: `COPY INTO bronze.orders_raw` loads every newly extracted batch file without deduplication, causing Bronze to grow by 8 rows on every extraction while downstream Silver and Gold maintain perfect idempotency.

<!-- SCD2 and 6th quality check experiment verified -->