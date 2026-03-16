# 2026-03-10 — VVIP Similarity Logic Investigation

## Session Type
Research / Investigation — NO code changes made

## Branch
`feature/vvip-sg-similarity` (based on `test`)

## What Was Done
Investigated whether VVIP similarity logic matches Shopee MY, and what's needed to make them work the same.

### Key Findings

#### 1. Branch Origin
- `feature/vvip-sg-similarity` is created from **`test`** (merge-base = exact tip of test)

#### 2. Database Schema Comparison

| Table | Location | Similarity Columns? | Role |
|---|---|---|---|
| `Shopee_Comp` | AllBots (Shopee MY) | YES — on comp row | Each our+comp pair has its own score |
| `webapp_test.Shopee_My_Products` | webapp_test (VVIP) | YES — on our product row | Score stored per our product/variation |
| `webapp_test.Shopee_Comp_Data` | webapp_test (VVIP) | NO | Only competitor data, no similarity |

- `Shopee_My_Products` has 1,450 rows, **1,235 have similarity data** already populated
- `Shopee_Comp_Data` has 3,378 rows, **zero similarity columns**

#### 3. Bot Comparison (VM3 vs VM TT)

**VM3** — Shopee MY bot (`C:\Users\Admin\Desktop\Shopee Comp My links Api\ca_similarity_check.py`)
- Targets: `Shopee_Comp` (single table)
- Processes: **Every** competitor pair
- No ROW_NUMBER — each row gets its own score
- `PROPAGATE_SIMILARITY_DATA = True`

**VM TT** — VVIP bot (`C:\Users\Admin\Desktop\ca_sg\ca_similarity_check.py`)
- Targets: `Shopee_My_Products` + `Shopee_Comp_Data` (two tables)
- Processes: **1 competitor per product** (ROW_NUMBER, rn = 1)
- Writes score to `Shopee_My_Products` (our product row)
- `PROPAGATE_SIMILARITY_DATA = False`
- Comment in code: "scores are stored on Shopee_My_Products (not per-pair), so processing multiple competitors per product wastes API calls (only the last write survives)"

Both bots use the **same AI logic** (Gemini API, same prompt, same scoring rules).

#### 4. Current Web App Code
- `shopee-vvip-products-repository.ts` lines 178-187: All 10 similarity columns are `NULL AS` — not wired up
- `shopee-products-repository.ts` (Shopee MY): Reads real columns from `sc.*` (Shopee_Comp)
- Shared enrichment layer (`shopee-products-enrichment.ts`) maps both — works with either real data or NULLs

### Two Options Identified

#### Option A — Easy (swap NULL to mp.*)
- **What to change**: Only `shopee-vvip-products-repository.ts` (10 lines)
- **DB change**: None
- **Bot change**: None
- **Result**: Similarity shows but ALL competitors under one product show the SAME score
- **Why**: Bot only compares 1 competitor, writes to our product row

#### Option B — Match Shopee MY exactly
- **DB change**: Add 10 similarity columns to `webapp_test.Shopee_Comp_Data`
- **Bot change**: Remove ROW_NUMBER, write scores to `Shopee_Comp_Data` per competitor pair
- **Code change**: Read from `cd.*` instead of `NULL AS` in repository
- **Result**: Each competitor gets its own unique score — same as Shopee MY

### Columns Needed for Option B (DB Migration)
```sql
ALTER TABLE webapp_test.Shopee_Comp_Data
  ADD COLUMN product_similarity_score INT NULL,
  ADD COLUMN product_similarity_reason TEXT NULL,
  ADD COLUMN product_similarity_datetime DATETIME NULL,
  ADD COLUMN variation_similarity_score INT NULL,
  ADD COLUMN variation_similarity_reason TEXT NULL,
  ADD COLUMN variation_similarity_datetime DATETIME NULL,
  ADD COLUMN product_similarity_excluded TINYINT DEFAULT 0,
  ADD COLUMN variation_similarity_excluded TINYINT DEFAULT 0,
  ADD COLUMN has_product_exclusions TINYINT DEFAULT 0,
  ADD COLUMN has_variation_exclusions TINYINT DEFAULT 0;
```

## Files Changed
None — research only

## Status
**Pending decision** — Kelly needs to discuss with buddy (bot developer) on Option A vs Option B

## Key Decision Points
- Option A = Kelly's work only (frontend), quick, but limited accuracy
- Option B = Kelly (frontend) + buddy (DB + bot), fuller solution, matches Shopee MY
