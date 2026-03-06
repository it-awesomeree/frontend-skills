# Session Log — 2026-03-06 — Fix Duplicate Variations + Show 0-Sales Comps

## Branch & PR
- **Branch**: `fix/dup-var-to-test` (based off `test`)
- **Target**: `test`
- **PR**: Not yet created via GitHub (MCP token issue) — branch is pushed, create PR manually
- **PR URL**: https://github.com/it-awesomeree/awesomeree-web-app/pull/new/fix/dup-var-to-test
- **Commits**:
  - `5351603` — make fix on showing variation once if using the same sku
  - `c143d31` — fix: show competitors with monthly sales = 0 in UI

## What Was Done

### 1. Deduplicate Product Variations by SKU
**Problem**: Some products showed extra variations in the Product Identity UI because different variation names shared the same SKU. For example, the Adelmar scooter (item_id 22026149091) had 28 variations in DB but should show 20 (8 SKUs had 2 variation names each, e.g., "Black (Adelmar)" and "Adelmar Edn - Black" both with SKU `ADLMR-BLK`).

**Fix**: In `groupProducts()`, use SKU as the variation grouping key when available. Falls back to normalized variation name when SKU is empty (~3% of rows).

**React key fix**: Added `key?: string` to `ProductVariationGroup` type and exposed the internal map key. Updated `grouped-rows.tsx` to use `vv.key || vv.variation` for React keys and `hasCompMap` lookups, fixing the "Encountered two children with the same key" console error.

**Cross-shop safety**: Verified SKU dedup is scoped per `product_id` (the outer grouping). Same SKU across different products/shops (e.g., `ADLMR-BLK` across 15 products in 4 shops) is never mixed.

### 2. Show Competitors with Monthly Sales = 0
**Problem**: The bot/script no longer removes 0-sales competitors from the DB. Agnes confirmed the UI should show them.

**Fix**: Removed the `> 0` filter in 3 places:
1. `getQualifiedCompRows()` — was `getCompMonthlySales(r) > 0`, now just checks valid comp name
2. Variation-level expand in `grouped-rows.tsx` — was `visibleRows.filter(r => getCompMonthlySales(r) > 0)`, now uses all `visibleRows`
3. `hasCompetitors` check — was checking `ms > 0`, now just checks comp name exists
4. Backfill logic — removed `getCompMonthlySales(r) <= 0` skip

0-sales comps rank at the bottom naturally via the existing sort-by-sales-descending logic. They only appear in top 3 if fewer than 3 comps have actual sales.

## Files Changed

### `lib/shared/shopee-history-calculations.ts`
| Change | Details |
|--------|---------|
| `ProductVariationGroup` type | Added optional `key?: string` field |
| `groupProducts()` | Uses `sku` as variation key when available, exposes key on output |
| `getQualifiedCompRows()` | Removed `> 0` filter, only checks valid comp name |
| Backfill logic | Removed `getCompMonthlySales(r) <= 0` skip |

### `components/Shopee-MY-History/grouped-rows.tsx`
| Change | Details |
|--------|---------|
| `vKey` construction | Uses `vv.key \|\| vv.variation` for unique React keys |
| `hasCompMap` set/get | Uses `vv.key \|\| vv.variation` instead of just `vv.variation` |
| `hasCompetitors` check | Removed `ms > 0` requirement, just checks comp name exists |
| Level 3 leaf rows | Removed `nonZero` filter, sorts all `visibleRows` directly |

## DB Investigation Results
- Adelmar (item_id 22026149091): 28 variations, 8 duplicate SKU pairs -> 20 after dedup
- Rattan Chair (item_id 29285505494): 26 variations, no duplicate SKUs -> unaffected
- 1,719 / 56,509 rows (~3%) have empty SKU -> falls back to name-based grouping
- Same SKU shared across 15 products in 4 shops -> safe because dedup is per product_id

## Comp Monthly Sales 0 — Logic Check (main branch)
Confirmed on `main` branch that the `> 0` filter existed in:
1. `getQualifiedCompRows()` (shopee-history-calculations.ts:605-608)
2. Variation-level expand (grouped-rows.tsx:2509)
Both now removed on the fix branch.

## Pending / Not Yet Done
- PR needs to be created manually (GitHub MCP token doesn't have access to private repo)
- Agnes's latest message: "Keep all the variations with the same category" — needs clarification on what she means (category consistency across variations? or don't dedup?)
- Main branch still has the stashed changes from `comp-analysis-vvip-and-sg-page` branch
