# Portfolio Tracker — Code Map

A guide to what the code does and how it is organized. Written for a reader who understands the tool's financial logic but does not need to know JavaScript.

---

## What the file is

The entire application is a single HTML file. It contains three things packed together:

- **HTML** — the page structure (panels, inputs, tables, buttons)
- **CSS** — the visual styling (dark theme, layout, colors)
- **JavaScript** — everything that moves: data, calculations, charts, navigation

There is no server, no database, no build process. Opening the file in a browser is enough to run the application.

---

## Data model — what the app remembers

At any point in time, the app holds six key pieces of state (variables that live in memory while the tab is open):

| Variable | Type | What it holds |
|---|---|---|
| `deposits` | List | Every deposit/withdrawal row: date, amount, portfolio name, currency, note |
| `portfolioVALs` | Dictionary | The current market value entered per portfolio: `{ 'IBKR': 82000, 'IOL': 24000 }` |
| `portfolioRs` | Dictionary | The solved MWRR per portfolio: `{ 'IBKR': { rn: 0.2444 }, 'IOL': { rn: 0.1140 } }` |
| `combinedR` | Number | The single MWRR solved from all deposits pooled together |
| `portfolioMonthly` | Dictionary | The planned monthly deposit per portfolio for the Step 3 projection |
| `lang` | String | Current language: `'en'` or `'es'` |

These six variables are the entire "database" of the application. Every display, every chart, every table is derived from them.

**Persistence:** all six are saved to `sessionStorage` (browser memory) after every change. This means the data survives a page refresh but is wiped when the tab is closed. The keys are `pt5_deps`, `pt5_vals`, `pt5_rs`, `pt5_monthly`, `pt5_combined`, `pt5_proj`.

---

## The 4-step wizard

The UI is divided into four panels. Only one panel is visible at a time. The function `goStep(n)` switches to step `n`, enforcing prerequisites:

- **Step 1** is always accessible.
- **Step 2** requires at least one deposit loaded.
- **Step 3** requires at least one MWRR calculated (single portfolio) or all VALs filled (multi-portfolio).
- **Step 4** requires `combinedR` to be solved AND a monthly withdrawal to be set.

---

## Step 1 — Data import

### What happens when you upload an Excel file

1. The browser reads the file using **SheetJS** (a bundled library).
2. A column-mapper UI appears, asking the user to match their column names to the required fields (date, amount, currency, portfolio, etc.).
3. On confirmation, `importFromExcel()` reads each row and calls `parseDate()` and currency conversion for each one.
4. Valid rows are pushed into the `deposits` array.
5. `renderTable()` is called to display the deposit list.
6. `autoCalcAll()` is called to trigger recalculation.

### Date parsing (`parseDate`, `inferDateFmt`)

Date parsing handles four formats: native Excel Date objects, ISO strings (`YYYY-MM-DD`), Argentine format (`DD/MM/YYYY`), and US format (`MM/DD/YYYY`).

Before importing any row, `inferDateFmt()` scans all raw date values in the file. If any date has a second component greater than 12 (e.g. `10/31/2025` — month 31 is impossible), the entire file is treated as MM/DD. This resolves ambiguous dates like `3/6/2024` consistently across the whole file rather than row by row.

### Currency conversion

ARS rows are divided by the `exchange_rate` column to produce a USD amount. Full floating-point precision is kept — no rounding at the row level, which would cause cumulative error across many rows.

---

## Step 2 — Returns

### Rendering the table (`renderRCards`)

Called once when navigating to Step 2. Builds the full returns table in HTML — one row per portfolio plus a combined row — and writes the VAL input fields. After this, only the result cells are updated (not the inputs), so that typing in a VAL field is not interrupted.

### VAL input behavior

Each VAL field has an `oninput` handler. The moment a character is typed:

1. The raw text is parsed as a float. If the field is empty, the value is stored as `undefined` (not zero) — this distinguishes "not entered" from "entered as zero".
2. `autoCalcAll()` is called immediately (no button press needed).

### The central recalculation engine (`autoCalcAll`)

This is the most important function in the application. It runs after every VAL change, every inflation change, and every deposit add/remove. It does three things in sequence:

**1. Solve MWRR for each portfolio:**
For each portfolio where a VAL has been entered (`valIsSet(p)` is true, including VAL = 0), it filters the deposit list to that portfolio's rows and calls `solveR(deps, val)`. If the result is valid, it is stored in `portfolioRs[p]`. If not, the entry is deleted (to clear any stale value from a previous keystroke).

**2. Solve combined MWRR:**
If all portfolios have a VAL entered (`allVALsFilled()`), it calls `solveR(deposits, getCombinedVAL())` — passing the *entire* deposit array (all portfolios together) and the sum of all VALs. The result is stored in `combinedR`. If not all VALs are filled, `combinedR` is set to null.

**3. Update the display:**
Calls `saveAll()` to persist state, `updateAllCardResults()` to refresh the result cells in the table, and `updateStepNav()` to update the step navigation bar.

### The MWRR solver (`solveR`)

Implements Newton-Raphson iteration to find the rate `r` that satisfies:

```
VAL = Σ dᵢ · (1 + r)^tᵢ
```

Where `tᵢ` is the number of years ago each deposit was made (`yearsAgo(date)`), calculated as integer days divided by 365.25 — matching Excel's `(TODAY() - date) / 365.25`.

The solver starts at an initial guess of 8% and iterates up to 2000 times, stopping when the change per step is smaller than 0.0000000001. After each step, the result is clamped to the range (−0.99, 10) to prevent the iteration from diverging into mathematically invalid territory.

A result is accepted only if it is not NaN and greater than −0.99. There is no upper cap — any positive return, no matter how large, is valid.

### Updating result cells (`updateAllCardResults`)

Reads from `portfolioRs` and `portfolioVALs` to compute and display, for each portfolio:

- **Nom r** — `portfolioRs[p].rn` directly
- **Real r** — Fisher equation: `(1 + rn) / (1 + inflation) - 1`
- **Quality badge** — based on real r: > 5% Excellent, > 2% Good, > 0% Fair, ≤ 0% Poor
- **Deposited** — `Σ dᵢ` (signed sum, can be negative for completed bonds)
- **Gain** — `VAL − Deposited`
- **Gain/dep** — `Gain / Deposited`, shown as `--` when Deposited ≤ 0
- **Share** — `VAL_portfolio / VAL_combined`
- **Max life** — if VAL > 0: age of oldest deposit; if VAL = 0: last deposit date − first deposit date
- **Wtd avg** — `Σ(amount × yearsAgo) / Σ(amount)`, shown as `--` when Deposited ≤ 0
- **Status** — "Ongoing" (VAL > 0) or "Completed" (VAL = 0)

After updating all portfolio rows, it calls `renderCombinedRResult()` if `combinedR` is available, and `drawHistoryChart()` to refresh the chart.

### Historical growth chart (`drawHistoryChart`)

Draws a line chart covering the period from the first deposit date to today. For each portfolio with a solved MWRR, it computes the implied portfolio value at each monthly interval by compounding each deposit at `rn` from its deposit date. The combined line uses `combinedR`. A dashed grey line shows cumulative deposits over the same period.

Tag annotations (deposits whose note starts with `tag:`) are drawn as vertical dashed lines at their date, in the portfolio's color, with word-wrapped label text. When two tags fall within 60px on the x-axis, their labels alternate top/bottom to avoid overlap. A dark semi-transparent background is drawn behind each label for readability.

---

## Step 3 — Projection and Withdrawal

### Projection table (`renderProjTable`)

Called once when navigating to Step 3. Builds the projection table with one row per portfolio plus a combined row. Each row has an editable monthly deposit input (disabled and locked at $0 for completed portfolios with VAL = 0). Calls `updateProjection()` at the end.

### Projection calculation (`updateProjection`)

For each portfolio with a solved rate, it computes:

- **Nom VAL** — `projectFV(val, monthly, rn, years)`
- **Real VAL** — `projectFV(val, monthly, rr, years)`
- **Alt VAL** — `projectFV(val, monthly, altRR, years)` (if an alternate rate is set)

The combined row uses `combinedR`, `getCombinedVAL()`, and the sum of all `portfolioMonthly` values.

### Future value formula (`projectFV`)

Standard compound future value with regular deposits:

```
FV = PV × (1 + rm)^n  +  PMT × [(1 + rm)^n − 1] / rm
```

Where `rm = (1 + r)^(1/12) − 1` is the exact geometric monthly rate (never the approximation `r/12`), and `n = years × 12` is the number of months. When `rm = 0` (zero return), the formula simplifies to `PV + PMT × n`.

### Sustainability simulation (`updateWithdrawal`)

After the projection horizon, the user enters a monthly withdrawal amount. The simulation runs month by month (up to 1200 months = 100 years) applying:

```
balance = balance × (1 + rm) − withdrawal
```

Earn first, withdraw second — the standard end-of-month convention. If `withdrawal ≤ perpetuity` (where `perpetuity = endVAL × rm`), the portfolio never depletes and "Indefinite" is shown. Otherwise, the simulation counts months until balance ≤ 0 and reports the depletion age.

Both the main scenario (using the portfolio's own real return) and the alternate scenario (using the user-specified alternate rate) are computed and shown in the same table.

---

## Step 4 — Summary and Print

### Entry point (`goStep4`)

Called when navigating to Step 4. Calls three table renderers in sequence, then draws the timeline chart:

1. `renderStep4ReturnTable()` — read-only snapshot of the Step 2 returns table, with inflation one-liner above
2. `renderStep4ProjTable()` — read-only snapshot of the Step 3 projection table, with Age today · Horizon one-liner above
3. `renderStep4WithdrawTable()` — read-only withdrawal summary, with Age at horizon one-liner above
4. `drawTimelineChart()` — the full three-phase chart

### Timeline chart (`drawTimelineChart`)

Draws a single continuous chart on an age x-axis covering three phases:

- **History** (first deposit → today): combined portfolio value compounded at `combinedR`, plus cumulative deposits line
- **Projection** (today → retirement): real VAL and alt real VAL, deposits line continuing with planned monthly additions
- **Withdrawal** (retirement → depletion): real VAL and alt real VAL declining under monthly withdrawal

Phase boundaries ("Today", "Retire") and age reference lines (60, 70, 80, 90, 100) are drawn as vertical dashed lines by a Chart.js `afterDraw` plugin called `phasePlugin`.

### Print rendering (`printReport`)

Canvas elements do not print reliably across browsers (especially Chrome). The solution:

1. Set `printMode = true` and redraw the chart with a light-background palette and `animation: {duration: 0}` (instant render)
2. Wait two animation frames for Chart.js to finish drawing
3. Snapshot the canvas to a PNG image via `canvas.toDataURL()`
4. Set that PNG as the `src` of a hidden `<img>` element in the page
5. Call `window.print()` from the image's `onload` event — Chrome loads data URLs asynchronously, so calling `window.print()` before `onload` produces a blank image
6. After the print dialog is dismissed, an `afterprint` event listener restores the dark-theme chart and clears the image

---

## Supporting systems

### Navigation and unlock logic (`updateStepNav`, `updateProceedBars`)

Called after every state change. Reads the current state of `deposits`, `portfolioRs`, `combinedR`, and the withdrawal input to decide which step tabs are unlocked, which show a checkmark, and what status message to show in the proceed bars.

### Bilingual support (`t`, `applyTrans`, `setLang`)

All user-facing strings are stored in a `TXT` dictionary with `en` and `es` keys. `t(key)` looks up the current language. `applyTrans()` walks all HTML elements whose `id` matches a translation key and updates their text. `setLang()` switches the language and triggers a full re-render.

### Number formatting

| Function | Used for | Example |
|---|---|---|
| `fmtUSD(v)` | Portfolio values, totals | `$84.4K`, `$1.24M`, `$842.50` |
| `fmtMo(v)` | Monthly amounts only | `$1,394/mo` |
| `fmtYrs(y)` | Time durations | `4yr 4mo`, `2yr`, `6mo` |
| `yearsAgo(date)` | Time since a deposit | `1.332` (decimal years) |

`fmtUSD` must never be used for monthly amounts — it rounds to K, which makes `$1,300/mo` and `$1,000/mo` both display as `$1K`.

### Color palette

Portfolio colors are assigned from a fixed palette in order of first appearance:

```
Teal · Purple · Pink · Lime · Cyan · Indigo
#2dd4bf  #c084fc  #f472b6  #a3e635  #67e8f9  #6366f1
```

The Combined portfolio always uses green (`#4ade80`). Colors are also used for quality badges (green/blue/yellow/red) and are intentionally non-overlapping with the portfolio palette.

---

## Key invariants (things that must always be true)

- **`valIsSet(p)`** is the only valid guard for "has a VAL been entered." Never use `portfolioVALs[p] > 0` — that excludes completed investments with VAL = 0.
- **Combined MWRR** is always solved from the raw deposit pool (`deposits`, all portfolios). It is never derived by blending individual portfolio rates.
- **`renderRCards()`** is only called on navigation to Step 2. Never call it from within auto-calc — it rebuilds the VAL inputs and destroys the user's cursor position. `updateAllCardResults()` is the function for in-place result updates.
- **Chart variables** (`sustainChart`, `timelineChart`, `histChart`, `projChart`) must be declared before the INIT block. The INIT block may call chart-drawing functions on session restore, and a `var x = null` declaration after that would overwrite the reference.
- **`portfolioMonthly[p]`** for completed portfolios (VAL = 0) is forced to 0 on render and the input is disabled. Completed investments accept no new capital.
