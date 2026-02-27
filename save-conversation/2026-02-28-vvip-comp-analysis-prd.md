# PRD: Shopee MY VVIP — Competitor Analysis Display

**Author:** Kelly
**Date:** 28 Feb 2026
**Branch:** `fix/vvip-competitor-display` → `test`
**Status:** Ready for review

---

## Overview

Connected the Shopee MY VVIP Comp Analysis page to the new normalized database tables (`Shopee_My_Products` + `Shopee_Comp_Data`) and implemented the competitor display logic with variation-level matching.

Previously, VVIP was pointing to the old `Shopee_Comp` table. Now it reads from two dedicated tables in `webapp_test` schema, with a post-processing layer that controls how competitors appear at each UI level.

---

## What Changed

| Area | Change |
|------|--------|
| Database | Switched from `Shopee_Comp` (old, denormalized) to `Shopee_My_Products` + `Shopee_Comp_Data` (new, normalized) |
| API Routes | All 3 VVIP API routes now query the new tables via LEFT JOIN |
| Competitor Logic | New post-processing pipeline limits to top 3 competitors with word overlap variation matching |
| Shopee MY | Zero changes — completely untouched |

---

## UI Behavior

| UI Level | What the user sees |
|---|---|
| **Product row (collapsed)** | Top 3 competitor products ranked by monthly sales |
| **Expand comp product icon** | ALL variations from that competitor product (full list) |
| **Our variation rows** | All our variations are always visible |
| **Expand comp on our variation** | Only shows competitors if our variation name matches a competitor's variation. If no match → blank, not expandable |

**Example:**
- Our variation "Blue - B" → matches comp variation "Blue" → expandable, shows matched competitors
- Our variation "Gift Box Set" → no matching comp variation exists → visible but not expandable (blank)

---

## Variation Matching Logic

Since our variation names and competitor variation names don't always match exactly (e.g., our "300x300cm,Beige" vs comp "3MX3M Beige"), the system uses a progressive matching approach:

| Priority | Strategy | Example |
|----------|----------|---------|
| 1st | Exact match | "blue" = "blue" |
| 2nd | Contains | "blue" found in "blue - b" |
| 3rd | Word overlap | "beige" shared between "300x300cm,Beige" and "3MX3M Beige" |

This was necessary because analysis of the 5 active VVIP products showed that 3 out of 5 had zero exact matches between our and competitor variation names.

---

## Files Changed

| File | Type | Description |
|------|------|-------------|
| `app/api/shopee-my-vvip/products/route.ts` | Modified | Added competitor limiting + word overlap matching logic |
| `lib/services/shopee-vvip-products-repository.ts` | Modified | Reverted SQL JOIN to product-level for correct data flow |

**Shared components (NOT modified):** `grouped-rows.tsx`, `shopee-history-calculations.ts`, `shopee-products-enrichment.ts` — all reused as-is.

---

## Testing Checklist

| Test | Status |
|------|--------|
| Top 3 comp products show on product row | Pending (needs deployed env) |
| Expand comp icon → all variations visible | Pending |
| Our variations all visible | Pending |
| Unmatched our variations not expandable | Pending |
| Matched our variations show correct comps | Pending |
| Shopee MY tab unchanged | Pending |
| `npm run build` passes | Pending |

> Note: Cannot test locally — IP not whitelisted for DB. Needs to be tested on deployed `test` environment after merge.

---

## What's Not Included (Out of Scope)

- **Similarity features** — new tables don't have similarity columns (fields will be NULL, UI handles gracefully)
- **Remarks filtering** — needs a VVIP-specific remarks table (future work)
- **Enrichment backfill** — VVIP tables already have their own `discounted_price`/`voucher_name`
