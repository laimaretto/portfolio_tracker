# Portfolio Tracker — User Guide

This guide explains how to use the tool, how to interpret the numbers, and how different types of investments behave inside the model.

---

## What you need to get started

Two things:

1. **A deposit history Excel file** — a spreadsheet with one row per deposit, containing at least: date, amount, currency, and portfolio name. This is the raw record of every time you put or witdraw money into or from your investments.

2. **Your current portfolio value (VAL)** — the number your broker shows today as the total market value of each portfolio. This is what you actually have right now.

That's it. No account, no login, no connection to any external service. Everything runs in your browser and is wiped when you close the tab.

### Two sources of truth

The tool computes everything from exactly two inputs that only you control:

- **Source of truth 1 — the Excel file.** You decide which cash flows are in scope. If a position is closed and you don't want it affecting the calculations, remove its rows from the active sheet and archive them in another tab. The tool does not validate your data or second-guess your decisions. Garbage in, garbage out — the math will always be correct, but the interpretation depends entirely on what you put in.
- **Source of truth 2 — the broker-reported VAL.** You enter the current market value of each portfolio manually. The tool trusts whatever you type. It has no connection to any broker or external service.

---

## Step 1 — Loading your deposit history

Upload your Excel file (`.xlsx` or `.xls`). The tool shows a column mapper where you match your file's column names to the required fields:

| Field | What it is |
|---|---|
| Date | When the deposit was made |
| Amount | How much was deposited |
| Currency | USD or ARS |
| Portfolio | Which portfolio it belongs to |

ARS deposits are automatically converted to USD using the exchange rate column if provided. If no exchange rate is available for an ARS row, that row is skipped and you are warned.

Once loaded, the deposit table shows every row with its parsed date, amount in USD, and portfolio. Check it for any obviously wrong dates or amounts before proceeding.

---

## Step 2 — Understanding your returns

### The inflation assumption

Before reading any results, set the inflation rate (%) in Section A. This single number does two things simultaneously:

- **Interprets the past** — strips inflation out of your historical nominal return to show how much your purchasing power actually grew.
- **Drives the future** — the real return derived here is the rate used in all projections in Step 3.

Since all deposits are converted to USD, the relevant benchmark is US CPI:

| Rate | When to use |
|---|---|
| 2.5% | Market-implied 10-year US CPI (TIPS breakeven) |
| 3% | Long-run US CPI average (post-WWII); standard base case |
| 4–4.5% | Conservative stress-test; reflects high-inflation decades |
| > 4.5% | Extreme stress-test |

This rate is an assumption, not a measured fact. Changing it shifts your interpretation of the past and your vision of the future together, consistently.

### Entering your VAL

Type the current market value of each portfolio directly in the VAL column. Returns calculate automatically. The combined row appears at the bottom once all VALs are filled.

### Reading the returns table

| Column | What it means |
|---|---|
| **Nom r** | Money-weighted rate of return (MWRR) — the annualized nominal rate at which your deposits have grown |
| **Real r** | Real return via Fisher equation: `(1 + r_n) / (1 + inflation) − 1` |
| **Quality** | Badge based on real r: Excellent (> 5%), Good (> 2%), Fair (> 0%), Poor (≤ 0%) |
| **Deps** | Number of deposit records |
| **Deposited** | Total amount deposited (sum of all deposit rows) |
| **Gain** | VAL − Deposited (absolute gain in USD) |
| **Gain/dep** | (VAL − Deposited) / Deposited — total return on deposited capital, not annualized |
| **Share** | VAL_i / VAL_combined — this portfolio's weight in your total |
| **Max life** | For ongoing investments (VAL > 0): age from first deposit to today. For completed investments (VAL = 0): duration from first to last deposit — the actual holding period, fixed at close. |
| **Wtd avg** | Dollar-weighted average age of deposits — how long your capital has effectively been deployed |
| **Status** | Ongoing (VAL > 0) or Completed (VAL = 0) |

**Gain/dep vs Nom r:** Gain/dep is the raw total multiplier — it does not account for time. Two portfolios with the same Gain/dep can have very different Nom r if one is older than the other. Nom r is the performance metric; Gain/dep is a cost-basis sanity check.

**Max life vs Wtd avg:** a portfolio that is 5 years old but has a weighted average time of 1 year means most of the capital arrived recently. The Nom r is effectively a 1-year story, not a 5-year one. Always read these two together.

### The historical growth chart

Once at least one VAL is entered, a chart appears below the table. For each portfolio with a solved return, it plots the implied portfolio value from the date of the first deposit to today — compounding each deposit at Nom r from its deposit date. The last point equals your entered VAL by construction.

The dashed reference line shows cumulative deposits over the same period. The gap between a portfolio line and the deposits line is the gain — past growth already achieved.

Use the pill selector above the chart to focus on one portfolio, the combined view, or all at once.

---

## How different investment types behave

The tool is designed for long-term variable-rate investments — stocks, funds, ETFs — where a broker reports a current market value and you have a deposit history. That is the primary use case and the spirit of the tool.

Cash holdings and fixed-rate investments (bonds, CDs, term deposits) are supported because the underlying math — MWRR via Newton-Raphson — handles any combination of positive and negative cash flows naturally. They are by-product possibilities of the design, not first-class features. They can be useful, but they also add noise to the combined picture: a completed bond contributes VAL = 0 to the projection starting point while its historical cash flows still influence the combined rate. If you find this confusing, the cleanest solution is to archive completed positions in a separate Excel tab and keep only active investments in the active sheet.

### Variable-rate portfolios (stocks, funds, ETFs)

This is the standard case the tool is designed for. Your broker reports a current market value that reflects all past price movements. You enter that VAL and the tool solves for the rate that makes your deposit history consistent with it.

**Today's VAL is simultaneously nominal and real** — it is today's money, so there is no inflation gap to correct. The r_r derived from it is the rate at which your purchasing power has grown, and it is the correct rate to use for future projections.

**Caveats for entering data:**

- **Deposits only, no reinvested dividends:** if your broker automatically reinvests dividends and they appear in the market value, you do not need to record them — they are already reflected in the VAL. Only record cash you transferred from outside the portfolio.
- **Withdrawals:** partial withdrawals from the portfolio are entered as negative deposit rows. The MWRR solver handles negative cash flows natively. Deposited (Σdᵢ) will reflect the net cash committed; Gain = VAL − Deposited remains correct. If total withdrawals exceed total deposits (Deposited < 0), Gain/dep and Wtd avg show `--` as the ratio is undefined.
- **Completeness matters:** the MWRR is only as accurate as your deposit history. Missing deposits distort the rate — the solver cannot infer cash flows that are not in the data.

#### Selling a position entirely

When you sell off a position completely, use the same pattern as a completed bond: add a negative row for the full sale proceeds on the sale date, and set VAL = 0.

| deposit_date | deposit_value | notes |
|---|---|---|
| 2022-01-01 | +10,000 | buy Cisco |
| 2024-01-01 | −15,000 | sell Cisco |
| VAL = **0** | | |

Deposited = 10,000 − 15,000 = **−$5,000** · Gain = 0 − (−5,000) = **+$5,000** · MWRR ≈ 22.5% annualized over 2 years.

The same "where did the money go?" logic applies: the $15K sale proceeds left this portfolio as a negative row. If you reinvested them into another portfolio, they already appear there as a positive deposit — nothing extra needed. If you spent them, they are out of scope. If they are sitting as idle cash, add a cash portfolio to keep them in the model.

### Cash holdings

If you hold cash that has not grown (e.g. a savings account paying near 0%), the tool will compute:

- **Nom r = 0%** (or near zero) — the number in the account didn't change.
- **Real r < 0%** — inflation has been eroding its purchasing power every year.
- **Quality badge: Poor**

This is correct. A 0% nominal return is not breaking even — it is losing ground to inflation. If inflation is 4%, you are losing approximately 3.85% of purchasing power per year (Fisher, not simple subtraction).

The projection chart for a cash portfolio shows this clearly: the nominal line is flat, the real line declines, and the gap between them is the cumulative cost of holding cash.

### Fixed-rate investments (bonds, CDs, term deposits)

The tool supports fixed-rate investments by recording interest payments as negative deposit rows. The MWRR solver treats them as cash outflows and computes the correct internal rate of return. Two scenarios apply depending on whether the investment is still running.

#### Ongoing bond (principal not yet returned)

Record all interest payments received to date as negative rows. Enter the current principal balance as VAL.

| deposit_date | deposit_value | notes |
|---|---|---|
| 2022-01-01 | +30000 | principal |
| 2022-04-01 | −600 | interest |
| 2022-07-01 | −600 | interest |
| … | −600 | interest |
| VAL = **30000** | | |

When all payments to date are recorded and VAL equals the intact principal, MWRR ≈ the bond's coupon rate. If payments are missing, the rate is understated — the solver sees idle capital and computes a lower blended return.

#### Completed bond (principal returned)

Add a final negative row for the principal return on the maturity date, at the same date as the last coupon payment. Enter VAL = 0.

| deposit_date | deposit_value | notes |
|---|---|---|
| 2022-01-01 | +30000 | principal |
| 2022-04-01 | −600 | interest |
| … | −600 | interest |
| 2025-01-01 | −600 | last interest |
| 2025-01-01 | −30000 | principal return |
| VAL = **0** | | |

With VAL = 0 the solver finds the IRR of the complete cash flow series, which equals the coupon rate exactly (subject to day-count rounding). This result is stable — it does not change as time passes after the bond matures, unlike the ongoing case.

**Caveats for entering data:**

- **Payment amount must match the actual coupon formula.** An 8% annual bond with quarterly payments pays `principal × 8% / 4` per quarter, not `principal × 8% / 3`. Using the wrong divisor produces a rate that does not match the stated coupon.
- **Payment completeness:** for ongoing bonds, every payment received to date must be in the Excel. A single missing period shifts the MWRR downward, because the solver sees capital sitting idle for that period.
- **Principal return on the same date as the last coupon:** if the last interest payment and principal return land on different dates, the solver sees a gap period where the capital earned nothing, which reduces the computed rate below the coupon rate.
- **Multiple tranches:** additional capital deployed into the same bond at a later date is entered as a positive deposit on the date it was committed. The MWRR correctly accounts for the different holding periods of each tranche.

**How completed bonds appear in the table:**

For VAL = 0, the Deposited column shows the net cash committed (Σdᵢ), which is negative — meaning you received more than you put in. Gain = VAL − Deposited = 0 − (negative) = the total interest earned. Gain/dep and Wtd avg show `--` because the ratios are undefined when the denominator is negative or zero. In the combined row, the completed bond's deposits are included in the global pool (consistent with the combined MWRR philosophy).

**Where did the money go?**

Every negative row — interest payments and the principal return — records money leaving the bond and returning to you. All of it is equally untracked once it exits. The model does not know what you did with the $4.8K in interest or the $30K in principal. Three cases apply to each outflow:

- **Reinvested into another portfolio** — it arrives as a positive deposit row in that portfolio. The model already sees it there; nothing extra is needed.
- **Spent (car, holiday, anything consumed)** — do not add it anywhere. Spent money is out of scope. The combined Deposited will include the bond's negative net contribution, which is correct: the bond was a net source of capital to your life, and you consumed it.
- **Sitting as untracked cash** — this is the only gap. The combined VAL does not include it, and Step 3 will project from a starting point that is lower than your actual total wealth by that amount. To close the gap, add a cash portfolio with positive deposit rows on the dates you received each outflow and a VAL equal to the current idle balance.

The combined Deposited in Step 3 will show the bond's contribution as a negative number. This is correct: it means the bond returned more than it consumed from your investment capital, and the full amount — principal and interest alike — either went somewhere else in the model or left the investment picture entirely.

---

## Step 3 — Projection and withdrawal

### What the projection shows

The projection table shows each portfolio as a row with its rates, starting VAL, monthly deposit input, total capital deployed by end of horizon, and projected end values.

| Column | What it means |
|---|---|
| **rn / rr** | The rates from Step 2 driving this portfolio's projection |
| **VAL today** | Starting point — today's market value from Step 2 |
| **Monthly** | Planned monthly deposit during the horizon (editable) |
| **Total dep** | Historical deposits + planned new deposits over the horizon |
| **Nom VAL** | Projected future value at nominal rate |
| **Real VAL** | Projected future value in today's purchasing power |
| **Alt VAL** | Projected future value at the alternate real return (if set) |

### Nominal vs real in the projection

Both lines start from the same VAL today (nominal = real at t=0). From there they diverge:

- **Nominal** grows at r_n — this is the number you would see in your account.
- **Real** grows at r_r — this is what that number can actually buy in today's dollars.

For retirement planning, the real line is what matters. The nominal line is useful for sanity-checking against broker statements, but it overstates your future purchasing power by the accumulated inflation over the horizon.

### The deposits line

The dashed reference line on the projection chart starts from your total historical deposits (not today's VAL) and adds planned monthly deposits over the horizon. The gap between the real or nominal line and the deposits line is the projected gain — the return on all capital ever deployed.

### The alternate real return

Enter a benchmark rate in Section A to compare two scenarios side by side. Useful for:

- Comparing your actual r_r against a target (e.g. "what if I had earned 5% real instead?")
- Stress-testing a lower rate (e.g. "what if returns drop to 2% real going forward?")
- Planning with a conservative assumption independent of your historical performance

### The sustainability simulation

After the horizon, enter a monthly withdrawal amount. The tool simulates month by month whether the portfolio sustains that withdrawal indefinitely or depletes.

**Perpetuity** — the maximum monthly withdrawal that leaves the portfolio intact forever. Click the perpetuity cell to auto-fill the withdrawal input.

If your withdrawal exceeds the perpetuity, the portfolio eventually depletes. The table shows:

| Color | Meaning |
|---|---|
| Green | Portfolio lasts 30+ years after retirement |
| Blue | 20–30 years |
| Yellow | 10–20 years |
| Red | Less than 10 years — review your withdrawal amount |

All withdrawal calculations use the **real** end VAL and **real** monthly rate — everything is in today's purchasing power, so the monthly withdrawal amount you enter is interpreted as a constant real amount (it keeps pace with inflation).

---

## Common questions

**Why is my real r not simply r_n minus inflation?**

Because the Fisher equation is multiplicative, not additive: `r_r = (1 + r_n) / (1 + inflation) − 1`. At low rates the difference is small (e.g. 8% − 4% = 4% vs Fisher 3.85%). At higher rates the gap matters more. The simple subtraction is an approximation; the tool always uses the exact Fisher formula.

**Why does the combined r differ from the average of my individual portfolio returns?**

Because combined MWRR is solved fresh by pooling all deposits from all portfolios against the combined VAL — it is not derived from the individual rates. There is no mathematically correct way to combine individual MWRRs after the fact. The tool always re-solves from the raw data.

**What does a Poor badge mean for a cash portfolio?**

It means inflation is eroding the purchasing power of the cash at |r_r|% per year. The nominal balance is unchanged, but each year it buys less. The projection shows this as a declining real line even though the nominal line is flat.

**Why does the inflation assumption affect my historical r_r?**

Because r_r is not a historical fact — it is a conditional statement: *given this inflation assumption, this is how much your purchasing power grew*. The historical nominal return (r_n) is the fact, derived purely from your deposit history and VAL. r_r is your interpretation of that fact through the lens of an inflation assumption. Change the assumption and the interpretation changes, while r_n stays fixed.

**What does "Today" on the historical chart mean?**

The last point on the chart is today's date, and by construction it equals exactly your entered VAL. This is not a coincidence — the MWRR is the rate that makes the compounded deposit history match your VAL. The chart is just showing you the path that rate implies over the full history.
