# Lessons Learned

Code review feedback and patterns to remember.

---

## PR #268 — AW-298 Senior Review Crosscheck (Feb 2026)

5 issues found in my PR #261 that were fixed in follow-up PR #268:

### 1. Use shared functions for the same logic
**Problem**: Reading competitor monthly sales from different fields in different places.
**Fix**: Created one `getCompMonthlySales()` function with consistent fallback paths.
**Lesson**: If the same data is read in multiple places, extract it into one shared function.

### 2. Always add safety limits
**Problem**: No cap on tied competitors — could show unlimited rows if all same sales.
**Fix**: Added `MAX_QUALIFIED_COMP_ROWS = 10` cap.
**Lesson**: Always consider "what if there are 1000 of these?" and add a reasonable limit.

### 3. Don't use magic strings as separators
**Problem**: Using `|||` to join competitor name + variation for dedup key — could collide if name contains `|||`.
**Fix**: Replaced with a two-level Map (`Map<name, Map<variation, data>>`).
**Lesson**: Use proper data structures instead of string concatenation with separators.

### 4. Don't duplicate code across files
**Problem**: Same `formatPriceValue()` function existed in 2 different files.
**Fix**: Moved to one shared location in `lib/shared/shopee-history-calculations.ts`.
**Lesson**: If you copy-paste a function, it should probably be shared.

### 5. Don't bypass TypeScript with `as any`
**Problem**: Using `(p as any).cofundVoucherLabel` when the field was already in the type.
**Fix**: Removed `as any` — just use `p.cofundVoucherLabel` directly.
**Lesson**: Check the type definition first before casting. `as any` hides bugs.
