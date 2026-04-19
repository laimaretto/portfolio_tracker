# Changelog

All notable changes to Portfolio Tracker are documented here.

---

## [v1.1.3] — 2026-04-19

### Changed
- **Author comment added** — HTML file now opens with a comment block identifying the author, license, and source repository.

---

## [v1.1.2] — 2026-04-17

### Changed
- **Variable names aligned with README nomenclature** — code variables now match the r_n / r_r / r_m naming used in the README math sections. `var r` (nominal MWRR) → `var rn`; `var realR` → `var rr`; `var r` inside `projectFV` (monthly rate) → `var rm`; Newton-Raphson step variable renamed from `rn` to `rnext` to free the name; `portfolioRs[p].r` property → `.rn`.

---

## [v1.1.1] — 2026-04-17

### Fixed
- **Return rates in Step 3 shown with 2 decimal places** — nominal r and real rr sub-labels in the projection section now show `.toFixed(2)` (e.g. "6.05%") instead of `.toFixed(1)` ("6.1%"), consistent with Step 2 cards. The rounding mismatch caused manual perpetuity calculations to differ from the tool by up to ~$15/month.

### Changed
- **README rewritten for clarity** — plain language throughout, no academic references. Removed Jensen's inequality, Trinity Study, "algebraically correct", "goal-seek". Simplified combined MWRR and sustainability sections.
- **README corrections** — initial guess fixed from 10% to 8% (matches code); FV formula updated to show exact geometric monthly rate $(1+r)^{1/12}-1$ instead of the approximation $r/12$ (matches code); "15-year chart" replaced with "growth chart" since the horizon is user-configurable.

---

## [v1.1.0] — 2026-04-16

### Changed
- **Bundled libraries** — Chart.js 4.4.1 and SheetJS 0.18.5 are now embedded directly in the HTML file. No CDN requests are made at any point. The tool works fully offline, from a USB stick, or any air-gapped environment.
- **"No data ever leaves your browser"** — claim restored to full accuracy now that no external requests are made.

---

## [v1.0.0] — 2026-04-16

### Changed
- **Excel-only import** — CSV support removed from Step 1. File input now accepts `.xlsx` and `.xls` only. The delimiter selector (Comma / Semicolon / Tab) has been removed.
- **Upload summary cleared on "Clear all"** — the import row count shown in Section A is now wiped together with the deposit history when "Clear all" is clicked.

---

## [v0.9.2] — 2026-04-14

### Fixed
- **Excel Date objects not parsed** — when SheetJS reads an `.xlsx` file with `cellDates:true`, date cells arrive as JavaScript `Date` objects, not strings. `parseDate` was calling `String(raw)` on them, producing a locale-dependent string (`"Mon Apr 13 2026 00:00:00 GMT-0300 ..."`) that none of the regex patterns matched. Dates now short-circuit: if `raw instanceof Date`, the local year/month/day are extracted directly. This eliminates "Invalid Date" entries in the Step 1 table and allows the solver to run in Step 2.
- **MM/DD/YYYY date format rejected** — `parseDate` assumed all two-component dates were DD/MM/YYYY. If an Excel file exported dates in US format (e.g. `04/14/2026`), the second component (14) was placed in the month position, producing `"2026-14-04"` — an invalid date string that passed the null check but broke `yearsAgo()` and `fmtDate()`. The parser now detects when the second component exceeds 12 (can't be a month) and automatically retries as MM/DD/YYYY.
- **Ambiguous dates not resolved consistently** — per-row format detection failed for dates where both components are ≤ 12 (e.g. `3/6/2024`). Even when the same file contained an unambiguous MM/DD date (`10/31/2025`), each row was parsed independently, so ambiguous rows defaulted to DD/MM and were parsed incorrectly. Fixed by adding `inferDateFmt()`, which scans all raw date values in the file before importing any row. If any date has a second component > 12, the entire file is treated as MM/DD/YYYY. The inferred format is passed to every `parseDate()` call in that import, ensuring consistency.

---

## [v0.9.1] — 2026-04-13

### Changed
- **3-step wizard** — removed Step 2 (Valuation). App is now Step 1 (Data) → Step 2 (Returns) → Step 3 (Projection).
- **VAL inputs moved into portfolio cards** — each portfolio card in Step 2 now has its own Current VAL input. The combined card has no VAL input; its totals are derived automatically.
- **Auto-calculation** — removed "Calculate r" and "Calculate combined r" buttons. Returns recalculate automatically whenever any VAL input or the inflation field changes (`autoCalcAll()`).
- **Separate render/update cycle** — `renderRCards()` builds the card structure (including VAL inputs) once on navigation. `updateAllCardResults()` updates only the result section within each card on subsequent changes, preserving input focus.
- **Inflation change** — now triggers `autoCalcAll()` instead of `renderRCards()`, so real r and badge update live without rebuilding the cards.
- **Deposit changes** (add/remove/import) — now trigger `autoCalcAll()` to keep returns in sync with data changes.

### Fixed
- **`sustainChart` variable declaration order** — `var sustainChart = null` was declared after the INIT block. On sessions with saved state, the INIT block calls `goStep(3)` → `updateProjection()` → `updateWithdrawal()` → `drawSustainChart()`, which assigned the Chart.js instance to `sustainChart`. The subsequent `var sustainChart = null` statement then overwrote that reference, leaving the variable null. Any later attempt to redraw the chart failed with a "Canvas already in use" error from Chart.js, silently preventing the withdrawal plot from updating. Fixed by moving the declaration to the global variable block at the top of the script, alongside `projChart`.

---

## [v0.9] — 2026-04-13

### Added
- **Max lifetime** — age of the oldest deposit, shown in the bottom metrics row of each portfolio card and the combined card. Gives context for interpreting r (a 9% return over 6 months is very different from 9% over 5 years).
- **Wtd avg time** — dollar-weighted average age of deposits (`Σ(amount × age) / Σ(amount)`), shown alongside Max lifetime. Reveals how much of the capital has actually been deployed over time and how strongly recent vs. older deposits dominate the MWRR calculation.
- **`fmtYrs(y)` helper** — formats decimal years as `Xyr Ymo` (e.g. `4yr 4mo`, `2yr`, `6mo`).

### Changed
- **Step 3 metrics grid** — expanded from 4 to 6 columns on both individual portfolio cards and the combined card to accommodate the two new metrics.
- **Version badge** — updated to v0.9.

---

## [v0.7] — 2026-04-13

### Changed
- **Step 3 combined card** — rebuilt to match individual portfolio card layout. Header shows "Combined" tag + button. Bottom metrics row: Deposits · Current VAL · Total deposited · Gain. Fully bilingual.
- **Step 3 portfolio card layout** — deposit count and total dep removed from card header. Header now shows only the portfolio name tag. Bottom metrics row expanded to 4 columns: Deposits · Current VAL · Total deposited · Gain.
- **Smart number formatting (`fmtUSD`)** — K values now show one decimal place when non-zero (e.g. `$84.4K`, `$109.8K`), and drop the decimal when it is zero (e.g. `$250K`). Prevents rounding mismatches like $84K + $25K = $110K.
- **Tool name** — "Portfolio Tracker" is now the fixed name in both languages; Spanish no longer translates it to "Seguimiento de Cartera".
- **Version badge** — header and page title updated to v0.7.

### Fixed
- **Untranslated strings** — "Continue to Valuation / Returns / Projection" buttons, deposit count ("N deposits"), and `años` (was `anos`) now fully translated in both EN and ES.
- **Language switch** — `setLang()` now calls `updateProceedBars()` so status labels and button text update immediately when toggling EN↔ES.

---

## [v0.7] — 2026-04-12

### Changed
- **Geometric monthly rate** — replaced `r/12` approximation with exact `(1+r)^(1/12)−1` in `projectFV`, withdrawal depletion simulation, perpetuity condition, and sustain chart. Applies to both main and alternate scenarios.
- **sessionStorage** — switched from `localStorage` to `sessionStorage`. Data is now wiped automatically when the tab is closed. Source-of-truth is the CSV/Excel file.

### Fixed
- **Monthly amount display** — added `fmtMo()` formatter so values like `$1,394/mo` no longer round to `$1K`. Applied to perpetuity display and the three withdrawal rate cards (4%, 3.3%, 2.5%).
- **Chart tooltips** — added `interaction: { mode: 'index', intersect: false }` to both projection and sustainability charts. Hovering anywhere on the chart now snaps to the nearest x-position and shows all curves simultaneously.

---

## [v0.6] — 2026-04-11

### Added
- **Alternate real return (arr)** — new input in Step 4, Section A for scenario comparison. Produces independent projection line and independent sustainability curves alongside the computed real return (rr).
- **Alt scenario metrics** — age-out-of-money and years-of-withdrawal for the alternate scenario displayed in Section D.
- **Per-portfolio monthly deposits** — Step 4 now has individual monthly deposit inputs per portfolio instead of a single shared input. Combined projection uses the sum; individual projections use each portfolio's own value.
- **Disclaimer banner** — added to Step 1 in both English and Spanish.
- **Staleness warning** — `r-stale-warn` indicator in Step 3 when deposits or VALs change after r has been calculated.
- **combinedR persistence** — combined r result now saved to `pt5_combined` and restored on reload.

### Changed
- **Badge labels use real r** — return quality badges (Excellent / Good / Fair / Poor) now based on Fisher real return, not nominal. Nominal return is misleading as a quality signal because it includes inflation.
- **Fisher real return shown in projection** — real r label in Section A sub-label shows the Fisher-computed rate.

### Fixed
- **HTML button IDs** — step buttons corrected from `sbtn-1/2` to `sbt-1/2`.
- **clearAll** — now removes all 6 sessionStorage keys including `pt5_monthly` and `pt5_combined`.
- **Monthly total display** — `updateMonthlyTotal()` uses `toLocaleString()` instead of `fmtUSD()` to avoid rounding.

---

## [v0.5] — 2026-04-10

### Added
- Combined MWRR — solved independently by pooling all deposits, not blended from individual rs.
- Real return via Fisher equation throughout (never simple subtraction).
- Sustainability simulation — month-by-month depletion with perpetuity condition.
- Withdrawal rate cards — 4%, 3.3%, 2.5% of real end-VAL.
- 15-year projection chart with nominal, real, and deposited curves.
- Bilingual support (English / Spanish) via `t(key)` / `TXT[lang]`.
- CSV and Excel import with column mapper UI.
- ARS → USD conversion via per-row exchange rate.
