# Changelog

All notable changes to Portfolio Tracker are documented here.

---

## [v1.5.7] — 2026-05-18

### Fixed
- **Step 3 & Step 4 — withdrawal chart x-axis when indefinite** — when monthly withdrawal is below the perpetuity threshold (portfolio never depletes), the withdrawal phase chart previously ran for a hardcoded 50 years, causing the balance curve to grow exponentially off-screen. The x-axis is now capped at `wdrawCapYears`: the number of years until the balance doubles (min 5yr, max 30yr). Fast growers get a shorter window; near-perpetuity portfolios get up to 30 years. Applied consistently to both the Step 3 sustainability chart and the Step 4 timeline chart.

---

## [v1.5.6] — 2026-05-18

### Fixed
- **Step 1 — total amount column alignment** — the total sum displayed in the table footer was misaligned, appearing under the Portfolio column instead of the Amount column.
- **Step 2 — column wrapping** — data cells now use `white-space:nowrap`, preventing values such as `5yr 10mo` from breaking across two lines in narrow columns.
- **Step 4 Sec B one-liner** — the metadata bar above the Projection table now includes the alternate real return (`Alt rr`) when one is set, alongside Age today and Horizon.
- **Step 4 print — one-liner font size** — the metadata one-liners above each table were disproportionately large relative to the 8px table content; reduced to 7px in the print stylesheet.

---

## [v1.5.5] — 2026-05-03

### Fixed
- **Step 3 — monthly deposit disabled for completed portfolios** — portfolios with VAL = 0 (completed) are no longer active; their monthly deposit input is now disabled and grayed out (locked at $0). Previously the input was editable, which was incorrect since a completed investment accepts no new capital.
- **Step 2 — tag label readability when overlapping curves** — tag annotation labels now have a dark semi-transparent background fill behind the text, ensuring they are readable regardless of where the portfolio value curves are on the chart (previously labels were invisible or hard to read when curves reached the top of the chart near the tag position).

---

## [v1.5.4] — 2026-05-02

### Added
- **Step 4 Sec A — inflation one-liner** — a small muted metadata bar above the Returns table now shows the inflation assumption used to compute real returns (e.g. `Inflation: 3.3%`), consistent with the one-liners above Sec B (Age today · Horizon) and Sec C (Age at horizon).

---

## [v1.5.3] — 2026-05-02

### Added
- **Step 2 — tag annotation lines on history chart** — deposit or withdrawal rows whose `note` field starts with `tag:` (case-insensitive) produce a vertical dashed line on the Step 2 historical growth chart at that row's date. The line is drawn in the portfolio's color. The text following `tag:` is word-wrapped at 12 characters and displayed as a small label above the line (or below it when two tags fall within 60 px on the x-axis, alternating top/bottom to prevent label overlap).

---

## [v1.5.2] — 2026-05-02

### Added
- **Step 4 — context one-liners above Sec B and Sec C tables** — a small muted metadata bar above each table shows the inputs that produced the results. Sec B (Projection) shows: Age today · Horizon. Sec C (Withdrawal) shows: Age at horizon. The rates and withdrawal amount are omitted since they already appear as columns in the respective tables.

### Changed
- **Step 3 — projection chart hidden when horizon = 0** — when the horizon input is set to 0, the growth chart in Step 3 Sec A is hidden (there is nothing to project). The sustainability simulation in Sec B still runs normally, starting directly from today's combined VAL.

### Fixed
- **Step 4 — timeline chart print rendering** — the chart previously printed with a black background and invisible phase/age-reference lines (Chromium) or not at all. Root causes: (1) canvas elements print unreliably across browsers; (2) `window.print()` returns immediately in Chrome before the print preview has painted, so any synchronous cleanup after the call clears the image before it renders; (3) Chrome loads data URLs on `<img>` asynchronously, so setting `img.src` and calling `window.print()` in the same tick shows a blank image. Fix: the Print button calls `printReport()`, which redraws the chart with a light palette (`printMode=true`, `animation:{duration:0}`), waits two `requestAnimationFrame` ticks for Chart.js to finish drawing, snapshots the canvas via `canvas.toDataURL()`, sets it as the `src` of a hidden `<img id="timeline-print-img">`, and only calls `window.print()` from `img.onload` — ensuring Chrome has decoded and painted the image before the print dialog opens. `@media print` hides the canvas and shows the image. An `afterprint` listener restores the dark-theme chart after the dialog is dismissed.
- **Step 4 — table crop in print** — cards with `overflow:hidden` clipped wide tables when printing. `@media print` now sets `overflow:visible` on Step 4 cards, adds `@page{margin:0.8cm}` for portrait A4 with tighter margins, and reduces table cell font to `8px` / padding to `2px 3px` so the 13-column Returns table fits without truncation.
- **Step 4 — timeline chart age reference lines colour** — the 60/70/80/90/100 age lines now use `clToday` (same colour as the "Today" phase separator) on both screen and print. The visual distinction from the phase separator is provided by the shorter dash pattern (`[3,5]`) and the numeric age labels.
- **Step 4 — timeline chart canvas overflowing card rounded corners** — the chart canvas, being a rectangular element, visually escaped the card's `border-radius` clip. Fixed by adding `overflow:hidden` to the chart card.

---

## [v1.5.1] — 2026-04-30

### Changed
- **Horizon can be zero** — the projection horizon input now accepts `0`. A zero horizon means the withdrawal simulation starts from today's combined VAL directly, covering two use cases: (1) the user has already reached their retirement date; (2) the user wants to see how their current portfolio depletes under a given withdrawal rate without any accumulation phase. The `getProjYears()` helper replaces the previous `||15` default pattern throughout, which silently treated `0` as "not set" and substituted 15 years.

### Fixed
- **Timeline chart — horizon=0 label collision** — when horizon is 0, the "Today" and "Retire" phase boundaries coincide at the same point. A single combined label "Today/Retire" is now drawn instead of two overlapping lines.

### Added
- **Timeline chart — age reference lines** — vertical dashed lines at ages 60, 70, 80, 90, and 100 are drawn on the Step 4 timeline chart whenever those ages fall within the chart's x-axis range. Styled more subtly than the phase separators so they serve as background reference without dominating the chart.

---

## [v1.5.0] — 2026-04-30

### Added
- **Step 4 — Summary** — a new fourth step that produces a printable report. Unlocks when all portfolio VALs are filled (combined r solved) and a monthly withdrawal is set in Step 3.
  - **Section A — Returns table**: read-only snapshot of the Step 2 returns table (all columns, all portfolios + combined row).
  - **Section B — Projection table**: read-only snapshot of the Step 3 projection table (rn · rr, VAL today, monthly, total dep, Nom VAL, Real VAL, Alt VAL).
  - **Section C — Withdrawal table**: read-only sustainability summary (real VAL at horizon, perpetuity, monthly withdrawal, age out of money, years of withdrawal; main + alt scenarios).
  - **Section D — Full timeline chart**: a single continuous chart spanning three connected phases, x-axis in age units:
    - *History* (first deposit → today): combined portfolio value compounded at r_n_combined; continuous deposit line.
    - *Projection* (today → retirement): real VAL and alt real VAL (if set) starting from combined VAL; deposit line continues with planned monthly additions. No nominal curve.
    - *Withdrawal* (retirement → depletion): real VAL and alt real VAL drawn under withdrawal simulation. Deposit line stops.
    - Vertical dashed lines mark "Today" and "Retire" phase boundaries directly on the chart.
  - **Print / PDF button**: calls `window.print()`. `@media print` CSS hides all navigation and interactive panels, leaving only the Step 4 content for clean printing or PDF export.

---

## [v1.4.1] — 2026-04-29

### Fixed
- **Stale MWRR after intermediate keystroke** — `autoCalcAll` fires on every `oninput` event, so typing a multi-digit VAL (e.g. "1000") triggers the solver four times. The previous guard only wrote to `portfolioRs[p]` on success; on failure it left whatever value was stored from a prior keystroke. This caused the wrong rate to remain on screen when a later, correct result happened to fall outside the (now removed) upper cap. Fixed: added `else delete portfolioRs[p]` so any guard failure immediately clears the stale entry. Same fix applied to `combinedR`.
- **Removed arbitrary 500% upper cap on MWRR** — results above 500% annualized were silently rejected. This has no mathematical or financial basis: any positive return is a valid answer to the MWRR equation. A new investor with a short window and a strong gain legitimately produces high annualized rates. The cap violated the GIGO design principle. Removed. Valid rejections are now limited to `NaN` (solver failed to converge) and `rn ≤ −0.99` (invalid mathematical domain — `(1+r)` must be positive for the power function to be real-valued).

---

## [v1.4.0] — 2026-04-28

### Changed
- **Documentation reorganised** — GUIDE.md, CHANGELOG.md, and MATH.md moved to `docs/` folder. README links updated. No functional changes.
- **Design philosophy documented** — CLAUDE.md now states explicitly: the tool validates math, not financial reality. Garbage in → garbage out is acceptable. No protective guardrails beyond ambiguous parsing errors.
- **Two sources of truth documented** — GUIDE.md and README.md now state explicitly that the Excel file (source of truth #1, user-controlled) and the broker-reported VAL (source of truth #2) are the only inputs the tool trusts.
- **Edge use cases framed** — GUIDE.md investment types section now opens by stating that variable-rate investments are the primary use case; cash and fixed-rate investments are by-product possibilities of the design, not first-class features.

---

## [v1.3.4] — 2026-04-27

### Added
- **Status column in Step 2 table** — a new "Status" column shows "Ongoing" (green) when VAL > 0 and "Completed" (grey) when VAL = 0. Makes it immediately visible which portfolios are still active and which are closed. Combined row shows "--".

### Changed
- **Max life for completed investments** — when VAL = 0, Max life now shows the actual holding period (last deposit date − first deposit date) instead of the age of the oldest deposit relative to today. For a bond that matured 2 years ago, this shows the investment's duration rather than a number that keeps growing indefinitely. For ongoing investments (VAL > 0), behaviour is unchanged: Max life = today − first deposit date.

---

## [v1.3.3] — 2026-04-27

### Added
- **VAL = 0 support for completed investments** — entering 0 as VAL is now treated as a valid, deliberately-set value (distinct from an empty/unentered field). When VAL = 0, the MWRR solver finds the IRR of the complete cash flow series, which is mathematically correct for a completed bond or term deposit where all principal and interest have been returned. This result is stable over time and does not drift as the analysis date advances.
- **`valIsSet(p)` helper** — distinguishes "not yet entered" (`undefined`) from "entered as zero" (a valid VAL). All guards that previously checked `portfolioVALs[p] > 0` now use `valIsSet(p)`, so completed investments with VAL = 0 participate correctly in auto-calculation, the historical chart, and the combined solver.
- **Negative deposit rows (cash outflows)** — interest payments and principal returns on fixed-rate investments are entered as negative deposit rows. The MWRR solver (`solveR`) handles negative cash flows natively via Newton-Raphson. No code change was needed — the math already supported this; only documentation was missing.

### Changed
- **Uniform display formula for all investment types** — `Deposited = Σdᵢ` (raw signed sum), `Gain = VAL − Deposited`. No special-casing for VAL = 0 or completed bonds. When `totalDep ≤ 0` (net outflows exceed net inflows), `Gain/dep` and `Wtd avg` show `--` rather than a meaningless ratio. The same formula applies to individual portfolio rows, the combined row, and Step 3's Total dep column.
- **Historical chart includes VAL = 0 portfolios** — `activePorts` filter now uses `valIsSet(p)` so a completed investment with VAL = 0 and a solved return contributes its compounded-deposit line to the chart. The deposits reference line is clamped to `Math.max(0, cumulative)` so it never goes negative when net outflows exceed inflows.

---

## [v1.3.2] — 2026-04-27

### Fixed
- **`projectFV` division-by-zero for r_n = 0%** — when nominal rate is exactly 0%, monthly rate `rm = 0` caused `PMT × 0 / 0 = NaN`. Added guard: when `rm = 0`, FV simplifies to `PV + PMT × n`. Nom VAL now correctly shows the flat nominal value for zero-return assets (e.g. cash held without growth).

---

## [v1.3.1] — 2026-04-26

### Changed
- **VAL today column added to Step 3 projection table** — a new "VAL today" column sits between rn·rr and Monthly, showing the current VAL for each portfolio and the combined total. Makes the handoff from Step 2 explicit: the user sees the starting value, the rates, and the projected values all in one row.
- **rn·rr column displays on one line** — rates now shown as `rn% / rr%` instead of stacked, keeping rows compact.
- **"New dep" column renamed to "Total dep" and corrected** — now shows `DEP_i + monthly_i × 12 × horizon`, the total capital deployed by end of horizon (past deposits plus planned future deposits). Previously only showed the incremental future deposits.
- **Projection chart deposits line corrected** — the dashed reference line now starts from historical `DEP_i` (total deposited to date) instead of today's `VAL`. This makes the line consistent with the "Total dep" column in the table, and makes the chart's gain gap meaningful: the distance between the VAL/Nom/Real lines and the deposits line is the true gain, past and future combined.
- **Withdrawal depletion color scale extended to red** — age/years cells now use four colors: green (≥ 30 yrs), blue (≥ 20 yrs), yellow (≥ 10 yrs), red (< 10 yrs). Previously < 20 yrs always showed yellow, understating short depletion scenarios.
- **Portfolio selector added to historical growth chart (Step 2)** — pill buttons above the chart let the user choose which history to plot: each individual portfolio, Combined, or All. Deposits reference line adjusts to match the selected scope. Defaults to All; resets on Clear all.

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
