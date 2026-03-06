# 2026-03-06 — Similarity Scoring & Exclusion Support for VVIP and SG Pages

## Session Info
- **Date**: 2026-03-06
- **Branch**: `comp-analysis-vvip-and-sg-page` (based on `test`)
- **Target**: PR to `test`
- **PR**: Created but GitHub MCP failed — need to create manually or retry
- **Branch pushed**: Yes (pushed to origin)

## What Was Done

### 1. Reviewed DATE-KELLY branch (date filter fix)
- Reviewed committed subquery approach ("first slow version") and uncommitted two-query approach
- The two-query approach is better: names query uses date filter for pagination, rows query strips date filter to show all variations
- Confirmed `db.ts` socketPath→host changes should NOT be committed (breaks GAE production)

### 2. Investigated comp_monthly_sales = 0 issue
- Found 751 rows with `comp_monthly_sales = 0` and 33,757 with NULL out of 56,509 total rows
- Example: "campfire" shop selling camping tents shows 0 monthly sales in DB but comp_sales = 526
- Prepared message for freelancer to investigate scraping logic

### 3. Added Similarity Scoring & Exclusion Support to VVIP/SG (MAIN WORK)
Brought the full similarity feature from Shopee MY to VVIP and SG pages, targeting `webapp_test` schema only.

**Key design decision**: Do everything in one go (scores + exclusions read + exclusions write) to avoid half-done state. When bot team finishes their work, it just works.

## Files Changed

### Modified (3 files)
1. **`lib/services/shopee-vvip-products-repository.ts`**
   - Swapped 10 `NULL AS` → `mp.*` for similarity columns (lines 198-207)
   - Added `EXCL` constant for `webapp_test.Shopee_Comp_Similarity_Exclusions`
   - Added `VVIP_EXCLUSIONS_JOIN` with COLLATE fix for collation mismatch (`utf8mb4_unicode_ci` vs `utf8mb4_0900_ai_ci`)
   - Updated grouped rows query: `NULL AS` → `pe.excluded_differences AS product_exclusions_json` + added EXCLUSIONS JOIN
   - Updated ungrouped rows query: same changes
   - Added 3 exclusion functions: `addVvipSimilarityExclusion`, `removeVvipSimilarityExclusion`, `getVvipSimilarityExclusions`
   - Flag UPDATE uses `our_link + sku` only (no `comp_link` — doesn't exist in `Shopee_My_Products`)

2. **`app/analytics/table/shopee-my-vvip/page.tsx`**
   - Changed exclusion API endpoint: `/api/products/similarity-exclusion` → `/api/shopee-my-vvip/similarity-exclusion`

3. **`app/analytics/table/shopee-sg/page.tsx`**
   - Same endpoint change as VVIP

### Created (1 file)
4. **`app/api/shopee-my-vvip/similarity-exclusion/route.ts`**
   - New API route (POST/GET/DELETE) mirroring `/api/products/similarity-exclusion`
   - Calls VVIP repo functions → targets `webapp_test` schema only
   - Full input validation and sanitization (same security as Shopee MY version)

### NOT Touched
- `shopee-products-repository.ts` (Shopee MY repo) — zero changes
- `/api/products/similarity-exclusion/route.ts` (Shopee MY exclusion API) — zero changes
- `app/analytics/table/shopee-my/page.tsx` — zero changes
- `AllBots.Shopee_Comp` — never referenced
- All shared UI components — zero changes

## Key Technical Details

### Collation Fix
- `Shopee_Comp_Similarity_Exclusions` uses `utf8mb4_unicode_ci`
- `Shopee_My_Products` and `Shopee_Comp_Data` use `utf8mb4_0900_ai_ci`
- Without COLLATE, MySQL throws: "Illegal mix of collations"
- Fix: Added `COLLATE utf8mb4_unicode_ci` to all JOIN conditions in `VVIP_EXCLUSIONS_JOIN`

### Table Structure Difference (VVIP vs Shopee MY)
- Shopee MY: single table `Shopee_Comp` has both our data + comp data → `comp_link` exists
- VVIP: two tables — `Shopee_My_Products` (our data, no `comp_link`) + `Shopee_Comp_Data` (comp data)
- Flag UPDATE in VVIP uses `our_link + sku` only (not `comp_link`)

### Data Flow
```
Bot writes to → webapp_test.Shopee_My_Products (similarity columns)
Web app reads → mp.product_similarity_score etc. (VVIP_SELECT_COLUMNS)
Exclusions stored in → webapp_test.Shopee_Comp_Similarity_Exclusions
Exclusions read via → VVIP_EXCLUSIONS_JOIN (LEFT JOIN with COLLATE)
Exclusions written via → /api/shopee-my-vvip/similarity-exclusion → addVvipSimilarityExclusion()
```

## Current State
- All similarity columns are NULL (bot hasn't run yet)
- Pages display exactly as before (dashes for NULL values)
- Once bot writes scores, they appear automatically — zero code changes needed

## What Bot Team Needs to Know
1. Write scores to: `webapp_test.Shopee_My_Products` — 6 columns
2. Reset flags after processing: `has_product_exclusions = 0`, `has_variation_exclusions = 0`
3. Read exclusions from: `webapp_test.Shopee_Comp_Similarity_Exclusions`

## Pending
- PR creation failed via GitHub MCP — need to create manually on GitHub
- Bot work is someone else's responsibility
- `comp_monthly_sales = 0` issue — message prepared for freelancer
