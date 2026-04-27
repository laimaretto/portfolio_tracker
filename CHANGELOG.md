# Changelog

All notable changes to Portfolio Tracker are documented here.

---

## [v1.3.1] — 2026-04-26

### Changed
- **VAL today column added to Step 3 projection table** — a new "VAL today" column sits between rn·rr and Monthly, showing the current VAL for each portfolio and the combined total. Makes the handoff from Step 2 explicit: the user sees the starting value, the rates, and the projected values all in one row.

---

## [v1.3.0] — 2026-04-26

### Added
- **Historical growth chart in Step 2** — a line chart appears below the returns table once at least one VAL is entered. For each portfolio with a solved r, it plots the implied portfolio value from the date of the first deposit to today (compounding each deposit at r_n from its deposit date). The combined line appears once all VALs are filled. A dashed reference line shows cumulative deposits over time. The chart uses the same color scheme as the rest of the app (portfolio colors, green for combined, grey dashed for deposits).

### Fixed
- **Deposits reference line filtered to active portfolios** — the dashed cumulative-deposits line in the historical chart previously summed deposits from all portfolios regardless of whether a VAL had been entered. It now only includes deposits from portfolios that have a VAL and a solved r, matching exactly the set of value lines drawn.

---

## [v1.2.0] — 2026-04-26

### Changed
- **Portfolio color palette redesigned** — previous colors conflicted with the VAL line and quality badge colors (green = Nom VAL / Excellent, blue = Real VAL / Good, yellow = Fair, red = Poor, orange = Alt VAL). New palette avoids all five reserved colors: Teal `#2dd4bf` · Purple `#c084fc` · Pink `#f472b6` · Lime `#a3e635` · Cyan `#67e8f9` · Indigo `#6366f1`. Combined portfolio retains `#4ade80` (green) intentionally.
- **Sustain chart accumulation curve fixed to blue** — the accumulation (pre-withdrawal) segment in the sustainability chart was green (`#4ade80`); changed to blue (`#60a5fa`) to match the Real VAL / rr color used throughout the projection table and chart.
- **Two new columns in Step 2 table** — "Gain/dep" shows `(VAL − DEP) / DEP` as a percentage (total return on deposited capital, color-coded green/red); "Share" shows `VAL_i / VAL_c` as a percentage (each portfolio's weight in the combined total). The combined row shows its own gain/dep and always 100% for share. Both columns show `--` until a VAL is entered.

---

## [v1.1.9] — 2026-04-25

### Changed
- **Rate column added to projection table** — a new "rn · rr" column sits between Portfolio and Monthly. Each row shows the nominal rate (top, in portfolio color) and the real rate (bottom, in quality color: green/blue/yellow/red matching the Step 2 badge). The combined row shows the pooled rn_c and rr_c. Cells show "--" until a VAL is entered.

---

## [v1.1.8] — 2026-04-25

### Changed
- **Current age moved to Section A** — sits alongside Horizon and Alternate real return as a 3-column form row; age conceptually belongs with the other projection parameters, not the withdrawal controls.
- **Section B reduced to one input** — only Monthly withdrawal remains; age removed.
- **Projection chart x-axis shows actual age** — labels are `age, age+1, …, age+n` instead of `Now, Y1, …, Yn`. Tooltip header updated to "Age N". Changing age in Section A immediately redraws the chart with the new axis.

---

## [v1.1.7] — 2026-04-25

### Changed
- **Compact table layout for Step 3 withdrawal** — the metric cards (Real VAL, Perpetuity, Age out of money, Years of withdrawal) and the 3 rate-hint cards (4%, 3.3%, 2.5%) are replaced with a two-row table. Row 1 shows the main scenario (real rr from the selected portfolio); row 2 appears only when an alternate real return is set. Clicking a perpetuity cell auto-fills the monthly withdrawal input.
- **Withdrawal section simplified to 2 inputs** — the descriptive note text and section D head removed; only Monthly withdrawal and Current age remain as controls above the table.

---

## [v1.1.6] — 2026-04-25

### Changed
- **Compact table layout for Step 3 projection** — individual metric cards (Nominal VAL, Real VAL, New deposits, Alt. Real VAL) and the per-portfolio monthly deposit inputs replaced with a single table. Each portfolio is one row with columns: Portfolio · Monthly deposit (input) · New deposits · Nom VAL · Real VAL · Alt VAL (hidden unless alt return is set). The combined row appears at the bottom once all VALs are filled.
- **Portfolio selector moved below projection table** — the "Chart & withdrawal:" pill selector now lives directly under the projection table instead of inside the settings card, making the data flow more natural: fill the table, then pick which portfolio drives the chart and withdrawal simulation.

---

## [v1.1.5] — 2026-04-25

### Changed
- **Compact table layout for Step 2** — portfolio cards replaced with a single table where each portfolio is one row. Columns: Portfolio · Current VAL (USD) · Nominal r · Real r (Fisher) · Quality · Deposits · Total deposited · Gain · Max lifetime · Wtd avg time. The combined row appears at the bottom of the same table once all VALs are filled, separated by a stronger border. Section C (separate combined card) removed.
- **"Current VAL" removed as a display metric** — it was redundant with the inline input field; the input column in the table serves both roles.

---

## [v1.1.4] — 2026-04-21

### Fixed
- **Stale combined R blocks Step 3 when any VAL is zeroed** — if the user had all VALs filled (combined R computed), then set any portfolio VAL to 0, `combinedR` was not cleared. Step 3 remained accessible and silently used the stale combined rate against a combined VAL that no longer included the zeroed portfolio — producing wrong projections. Fixed by clearing `combinedR` in `autoCalcAll` whenever `allVALsFilled()` is false.
- **Step 3 blocked until all VALs are filled (multi-portfolio)** — for a single portfolio, Step 3 unlocks as soon as that portfolio's return is calculated. For multiple portfolios, Step 3 now requires all VALs to be filled and combined R to be computed. The proceed bar and toast message tell the user explicitly: "Enter VAL for all portfolios".

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
