# Portfolio Tracker

A single-file, zero-dependency web app for tracking personal investment portfolios, computing money-weighted returns, and projecting retirement sustainability.

No server. No login. No data ever leaves your browser.

---

## Why this tool exists

Most investment dashboards show you time-weighted returns — a metric designed for fund managers to compare performance against benchmarks. That number is almost useless for an individual investor whose portfolio grows through irregular, real-world deposits.

This tool computes the **Money-Weighted Rate of Return (MWRR)**, which answers the question that actually matters:

> *Given exactly when and how much I invested, what annualized return does my portfolio imply?*

It then applies Fisher's real-return equation to strip out inflation, and runs a month-by-month simulation to tell you whether your projected retirement portfolio is sustainable at your target withdrawal rate.

---

## Workflow

The app is a 3-step wizard. Each step unlocks the next.

### Step 1 — Data

Upload a CSV or Excel file containing your deposit history. The mapper UI lets you match your file's column names to the required fields (date, amount, currency, portfolio name). ARS deposits are converted to USD using the exchange rate column if provided.

Deposits are stored in `sessionStorage` — they exist only for the duration of the browser tab and are wiped automatically when the tab is closed. Data never leaves your machine.

### Step 2 — Returns

Each portfolio is shown as a card. Enter the current market value (VAL) directly in the card. Returns calculate automatically — no button to press. Change the VAL or the inflation assumption and the rates update instantly.

**Inflation assumption:** enter the annual inflation rate (%) to use for the Fisher real-return calculation.

| Assumption | When to use |
|---|---|
| 3% | Standard long-run USD assumption; consistent with US Federal Reserve target and historical 20-year averages |
| 4–4.5% | Moderately conservative; appropriate if you expect persistently above-target inflation |
| > 4.5% | Stress-test / high-safety scenario; increases the conservativeness of the real-return estimate |

A higher inflation assumption lowers the computed real return. When in doubt, run the tool twice (once at 3%, once at 4.5%) to bracket the outcome.

Each card shows: nominal r, real r (Fisher), quality badge, deposit count, current VAL, total deposited, gain, max lifetime (age of oldest deposit), and weighted average time (dollar-weighted average age of deposits).

The combined card auto-populates once all portfolio VALs are entered and shows aggregated metrics across all portfolios.

### Step 3 — Projection & Withdrawal

Configure planned monthly deposits per portfolio and a time horizon. The app projects:

- **Nominal VAL** — future value at the nominal MWRR
- **Real VAL** — future value in today's purchasing power (at Fisher real return)
- **Alternate Real VAL** — same projection at a user-specified alternate real return, for scenario comparison

A 15-year growth chart overlays all three curves.

Below the chart, a sustainability simulation asks: *after the horizon ends, how much do you withdraw monthly?* It computes whether the portfolio survives indefinitely (perpetuity condition) or depletes, and if so, at what age. Both the main scenario and the alternate scenario are plotted together.

---

## The math

### Money-Weighted Rate of Return (MWRR)

The MWRR is the rate **r** that satisfies:

```
VAL = Σ dᵢ · (1 + r)^tᵢ
```

Where:
- `VAL` is the current portfolio value
- `dᵢ` is deposit *i* (in USD)
- `tᵢ` is the number of years that deposit has been invested

**tᵢ calculation:**

```
tᵢ = round((today − deposit_date) / 86400) / 365.25
```

Integer days divided by 365.25. This matches the Excel formula `(TODAY() - deposit_date) / 365.25` exactly. Using fractional milliseconds or noon timestamps produces different results.

**Solver:** Newton-Raphson iteration on the equation above, starting from an initial guess of 10%. Converges in under 20 iterations for typical portfolios.

This is equivalent to the Internal Rate of Return (IRR) of the deposit cash flow with the current portfolio value as the terminal payoff. It is the industry-standard MWRR as defined by the CFA Institute Global Investment Performance Standards (GIPS).

---

### Portfolio lifetime metrics

Two additional metrics are shown on each card to give context for interpreting r.

**Max lifetime** — the age of the oldest deposit:

```
max_lifetime = max(yearsAgo(deposit_date_i))
```

A 9% MWRR over 6 months and a 9% MWRR over 6 years are not equivalent in terms of statistical weight or confidence. Max lifetime answers: *how long has this portfolio actually been running?*

**Weighted average time** — the dollar-weighted average age of all deposits:

```
wtd_avg_time = Σ(amount_i × yearsAgo(date_i)) / Σ(amount_i)
```

The gap between max lifetime and wtd avg time reveals whether capital is front-loaded (early large deposits) or back-loaded (recent large deposits). A portfolio that is 5 years old but has a weighted average time of 2 years means the bulk of the capital arrived recently — so the MWRR is effectively a 2-year story, not a 5-year one.

`wtd_avg_time` also has a direct practical interpretation: it is the single effective time horizon that, combined with total deposits and r, approximately reconstructs the current portfolio value:

```
VAL ≈ total_deposited × (1 + r)^wtd_avg_time
```

This is an approximation (the exact MWRR equation is `VAL = Σ dᵢ·(1+r)^tᵢ`; Jensen's inequality means the two are not identical), but for typical deposit patterns and return levels the error is small. The formula gives `wtd_avg_time` a concrete meaning: it is the equivalent lump-sum holding period that produces the same portfolio value as the actual staggered deposit history.

Note: using the computed `r` (which was solved from the broker-reported VAL) makes the formula circular — convergence is guaranteed by construction and tells you nothing new. The formula becomes genuinely informative when used with an **independent** return assumption, such as a benchmark or target rate, to estimate what the portfolio *should* be worth given the deposit history and holding period.

---

### Real return (Fisher equation)

```
r_real = (1 + r_nominal) / (1 + inflation) - 1
```

The simple approximation `r_real ≈ r_nominal − inflation` is not used. At inflation rates above 5% (relevant for ARS-denominated portfolios), the approximation error compounds significantly over multi-year projections.

---

### Future value projection

Standard compound growth with regular monthly contributions:

```
FV = PV · (1 + r/12)^(12·years) + PMT · [(1 + r/12)^(12·years) − 1] / (r/12)
```

Where `PMT` is the monthly deposit and `r` is either the nominal or real rate depending on which projection is being computed.

When multiple portfolios exist, each portfolio's projection uses its own `r` and its own share of the planned monthly contribution. The combined projection uses the blended MWRR computed from all deposits together.

---

### Combined MWRR across multiple portfolios

When you have two or more portfolios, the combined `r` is **not** derived from the individual portfolio returns. It is solved independently by pooling every deposit from every portfolio into a single cash flow stream and solving the MWRR equation once more against the combined VAL:

```
combined_VAL = Σ all_deposits (all portfolios) × (1 + r_combined)^t_i
```

The individual `r_1`, `r_2`, … results play no role in this calculation. They are discarded and the solver runs fresh on the full deposit history.

This matters because there is no algebraically correct way to combine individual MWRRs into a portfolio-level MWRR after the fact — not by simple averaging, not by deposit-weighting, not by VAL-weighting. The only correct approach is to re-solve the equation from the raw cash flows. Any blending formula applied to pre-computed individual returns is an approximation whose error depends on how differently the portfolios were funded over time.

The practical implication: if you maintain portfolio records in separate spreadsheets, you would need to consolidate all deposit rows into a single sheet and run a third goal-seek to get the true combined MWRR. This tool does that automatically since all deposits are stored in a single flat structure regardless of which portfolio they belong to.

---

### Sustainability simulation

After the accumulation horizon, the portfolio enters a withdrawal phase. All withdrawal-phase calculations use **real** values (today's purchasing power) throughout.

**Perpetuity condition:**

```
perpetuity_monthly = endVal_real × (r_real / 12)
```

If the planned monthly withdrawal ≤ perpetuity, the portfolio never depletes. This is the classic "4% rule" threshold generalized to any real return.

**Depletion simulation (if withdrawal > perpetuity):**

Month-by-month iteration:

```
r_monthly = (1 + r_real)^(1/12) − 1
balance[t+1] = balance[t] · (1 + r_monthly) − withdrawal_monthly
```

The monthly rate uses the exact geometric conversion rather than the common approximation `r_real / 12`. The approximation understates the monthly rate (e.g. 0.4167% vs 0.4074% at 5% real), which slightly overstates long-run growth. The geometric rate is exact: compounding it 12 times recovers the original annual rate precisely.

The simulation runs until `balance ≤ 0` or 100 years, whichever comes first. The depletion month determines the age at which the portfolio reaches zero.

**Why monthly simulation differs from a back-of-envelope annual estimate:**

A quick annual check might subtract 12 months of withdrawals upfront and then apply one year of growth:

```
rough_R1 = (endVal − withdrawal × 12) × (1 + r_real)
```

The simulation gives a slightly higher result for two reasons:

1. **Monthly compounding earns more than annual compounding** — the geometric monthly rate `(1 + r)^(1/12) − 1`, compounded 12 times, recovers exactly `r`. But applying it monthly to a balance that is also being drawn down monthly produces a different trajectory than one annual growth step on a depleted balance.
2. **Withdrawal timing** — in the monthly model the balance earning returns is larger early in the year, before each withdrawal reduces it. Subtracting all 12 withdrawals upfront understates the average invested balance.

Both effects are small at typical return and withdrawal levels (a few hundred dollars per year on a $200K portfolio) but compound over multi-decade horizons. The monthly model matches how a real brokerage account behaves.

**Withdrawal rate cards:**

Three reference cards show the implied monthly withdrawal at 4%, 3.3%, and 2.5% of the real end-VAL. These correspond roughly to the academic "safe withdrawal rate" literature:
- 4% — Trinity Study baseline (30-year horizon, mixed equity/bond portfolio)
- 3.3% — More conservative estimate for longer horizons
- 2.5% — Near-perpetuity; appropriate for early retirement

All three are computed from the **real** end-VAL, keeping them consistent with the simulation.

---

## Technical notes

- Single HTML file — no build system, no npm, no dependencies to install
- CDN libraries: [Chart.js 4.4.1](https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js), [SheetJS 0.18.5](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js)
- State persisted in `sessionStorage` under keys `pt5_deps`, `pt5_vals`, `pt5_rs`, `pt5_monthly` — cleared automatically on tab close
- Fully bilingual: English and Spanish (Argentine locale)
- Supports CSV (comma / semicolon / tab) and `.xlsx` / `.xls` files
- Currency: USD native; ARS converted via per-row exchange rate

---

## CSV format

| Field | Required | Accepted column names |
|---|---|---|
| Deposit date | Yes | `date`, `fecha`, `fecha_deposito`, `transaction_date` |
| Deposit amount | Yes | `deposit_value`, `amount`, `monto`, `valor`, `importe` |
| Currency | Yes | `currency`, `moneda`, `currency_code` |
| Portfolio name | Yes | `portfolio`, `cartera`, `account`, `cuenta`, `fund` |
| USD exchange rate | No | `exchange_rate`, `tipo_cambio`, `fx`, `fx_rate`, `rate`, `tc` |
| Note | No | `nota`, `description`, `descripcion`, `comment` |

**Currency values accepted:**
- USD: `USD`, `US$`, `U$S`, `Dollar`, `Dolares`, `Dólar`
- ARS: `ARS`, `AR$`, `Pesos`, `Peso`

**Date formats accepted:**

| Format | Example | Notes |
|---|---|---|
| Excel date cell | *(native)* | Read directly as a Date object by SheetJS — most reliable, no ambiguity |
| `YYYY-MM-DD` | `2021-12-13` | ISO 8601; always unambiguous — recommended for CSV exports |
| ISO with time | `2021-12-13T00:00:00.000Z` | Time component stripped automatically |
| `DD/MM/YYYY` | `13/12/2021` | Argentine default; `/`, `-`, and `.` separators accepted |
| `MM/DD/YYYY` | `12/13/2021` | Detected automatically at the file level (see below) |

**MM/DD vs DD/MM detection:** the tool scans all dates in the file before importing any row. If any date has a second component greater than 12 (e.g. `10/31/2025` — month 31 is impossible), the entire file is treated as MM/DD/YYYY. This means ambiguous dates in the same file, such as `3/6/2024`, are correctly resolved to the same format (March 6, not June 3). If no unambiguous date is found, the file defaults to DD/MM/YYYY.

To guarantee correct parsing regardless of file format, use `YYYY-MM-DD` in your CSV.

---

## Disclaimer

This tool is provided for informational and educational purposes only. It is not financial advice. Past returns do not guarantee future results. The projections produced by this tool are mathematical illustrations based on inputs you provide — they are not predictions. Do not make investment decisions based solely on this tool. The author bears no responsibility for financial outcomes resulting from use of this software.

---

## Return quality labels

Each portfolio in Step 2 displays a badge rating based on its **real return** (Fisher-adjusted), not the nominal return. Nominal return is misleading as a quality signal because it includes inflation — a portfolio earning 8% nominal in a 10% inflation environment is actually losing purchasing power.

| Real r | Label | Color |
|---|---|---|
| > 5% | Excellent | Green |
| > 2% | Good | Blue |
| > 0% | Fair | Yellow |
| ≤ 0% | Poor | Red |

A portfolio with a high nominal r but negative real r will correctly show as **Poor**.

---

## Usage tips

**Setting monthly deposits to $0** strips out the effect of future contributions entirely. The projection becomes:

```
FV = VAL × (1 + r)^years
```

This isolates the compounding of your existing capital — what today's money becomes on its own, with no new saving. Running the tool twice, once at $0 and once at your planned monthly amount, gives you a natural decomposition:

- **$0 line** — growth of capital you already have
- **Gap between the two lines** — value added by future deposits

This is useful for understanding how much of your projected retirement portfolio is already "locked in" by past decisions vs. how much depends on continued saving.

---

## License

BSD 3-Clause — see [LICENSE](LICENSE)
