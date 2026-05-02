# The math

## Money-Weighted Rate of Return (MWRR)

The starting point is $r_n$ — the **nominal annual return** — which is the rate that satisfies:

$$\text{VAL}  = \sum_i d_i \times (1+r_n)^{t_i}$$

Where:
- $\text{VAL}$ is the current portfolio value
- $d_i$ is deposit *i* (in USD)
- $t_i$ is the number of **years** that deposit has been invested

Because $t_i$ is in years, $r_n$ is an **annual** rate. It answers: *given my deposit history, how fast has my portfolio actually grown in nominal terms?*

**tᵢ calculation:**

$$t_i = \frac{\text{round}(\text{today} - \text{date}_i)}{365.25}$$

Where the difference is in **integer days** — identical to the Excel formula `(TODAY() - deposit_date) / 365.25`. Using fractional days or a different divisor breaks parity with Excel.

**Solver:** Newton-Raphson iteration on the equation above, starting from an initial guess of 8% — a reasonable starting point for a typical investment portfolio. Converges in under 20 iterations.

Each Newton step is $r \leftarrow r - f(r)/f'(r)$. Two conditions can cause the step to blow up: (1) when $t_i$ is small (recent deposits), the derivative $f'(r) = \sum d_i \, t_i (1+r)^{t_i-1}$ approaches zero, making the step enormous; (2) when $r \to -1$, $(1+r)^{t_i} \to 0$, again collapsing the derivative. Below $r = -1$, the base $(1+r)$ is negative and fractional powers are not real-valued — the domain is invalid. To keep the iteration stable, the solver clamps each step to $r \in (-0.99,\ 10]$ before the next iteration. These clamps do not constrain the final result; they only keep the iteration path in a region where the function is well-behaved.

The only principled bound on the result itself is the lower limit $r > -0.99$ (a return below $-99\%$ is outside the valid domain). There is no upper bound — any positive return, no matter how large, is a mathematically valid answer to the MWRR equation.

---

## Portfolio lifetime metrics

Two additional metrics are shown on each card to give context for interpreting $r_n$.

**Max lifetime** — the age of the oldest deposit:

$$\text{max-lifetime} = \max_i\, t_i$$

A 9% return over 6 months and a 9% return over 6 years tell very different stories. Max lifetime answers: *how long has this portfolio actually been running?*

**Weighted average time** — the dollar-weighted average age of all deposits:

$$\text{wtd-avg-time} = \frac{\sum_i \text{amount}_i \times t_i}{\sum_i \text{amount}_i}$$

The gap between max lifetime and wtd avg time reveals whether capital is front-loaded (early large deposits) or back-loaded (recent large deposits). A portfolio that is 5 years old but has a weighted average time of 2 years means the bulk of the capital arrived recently — so the return figure is effectively a 2-year story, not a 5-year one.

`wtd_avg_time` also has a practical interpretation: it is the equivalent holding period that approximately connects your deposit history to your current portfolio value:

$$\text{VAL} \approx \text{total-deposited} \times (1+r_n)^{\text{wtd-avg-time}}$$

This is an approximation, but for typical deposit patterns the error is small. It gives `wtd_avg_time` a concrete meaning: the single time horizon that, combined with your total deposits and return, reconstructs roughly what your portfolio is worth today.

---

## Real return

From $r_n$, the **real annual return** $r_r$ is derived using the Fisher equation:

$$r_r = \frac{1 + r_n}{1 + \pi} - 1$$

where $\pi$ is the annual inflation rate. Like $r_n$, $r_r$ is an **annual** rate — it answers: *how much purchasing power has my portfolio actually gained after discounting inflation?* The simple approximation $r_r \approx r_n - \pi$ is not used — at higher inflation rates the error compounds meaningfully over multi-year projections.

---

## Combined MWRR across multiple portfolios

When you have two or more portfolios, the combined return, $r_{nc}$, follows the same three steps as each individual portfolio:

1. **Solve $r_{nc}$** — pool every deposit from every portfolio and find the rate that satisfies:

$$\sum_i d_i \times (1 + r_{nc})^{t_i} = \sum_j \text{VAL}_j = \text{VAL}_{c}$$

Where $\text{VAL}_c$ is the sum of all portfolio values entered in Step 2; $d_i$ sum runs over every deposit across all portfolios; $r_{nc}$ is the unknown; $\text{VAL}_c$ is the known input.

2. **Derive $r_{rc}$** — apply Fisher;

3. **Derive $r_{mc}$** — convert to monthly.

There is no correct way to combine individual returns into a portfolio-level return after the fact — not by averaging, not by weighting. The only correct approach is to re-solve from the raw deposit data. This tool does that automatically.

---

## Future value projection

For projection, the annual rates $r_n$ and $r_r$ are converted to their **monthly** equivalent $r_m$:

$$r_m = (1 + r_r)^{\frac{1}{12}} - 1$$

$r_m$ is the only rate used during projection and withdrawal — $r_n$ and $r_r$ are annual and never applied directly to monthly calculations.

The projection formula is:

$$FV = \text{VAL} \cdot (1+r_m)^{12n} + \text{PMT} \cdot \frac{(1+r_m)^{12n} - 1}{r_m}$$

where $\text{PMT}$ is the monthly deposit and $n$ is the horizon in years. The tool computes three projection lines, each deriving its own $r_m$ from the corresponding annual rate:

| Line | Annual rate | Monthly rate |
|---|---|---|
| Nominal VAL | $r_n$ — MWRR solved from deposits | $(1+r_n)^{\frac{1}{12}}-1$ |
| Real VAL | $r_r$ — Fisher-adjusted for inflation | $(1+r_r)^{\frac{1}{12}}-1$ |
| Alternate Real VAL | user-supplied $ar_r$ | $(1+ar_r)^{\frac{1}{12}}-1$ |

When multiple portfolios exist, each portfolio's projection uses its own $r_n$ and $r_r$. The combined projection plugs three combined quantities into the same FV formula:

1. $\text{VAL}_c = \sum_i \text{VAL}_i$
2. $\text{PMT}_c = \sum_i \text{PMT}_i$
3. $r_{rc}$ from the combined MWRR solver (computed in the section above)

$\text{PMT}_c$ is the sum of the per-portfolio monthly deposit fields in Step 3 — so those fields matter for the combined projection, not just the individual ones.

An alternative would be to sum the individual portfolio projections:

$$\text{FV}_c = \sum_i \text{FV}_i$$

where each portfolio projection uses its own real return and monthly deposit. When individual real returns differ significantly, the two approaches can diverge. However, when returns are similar across portfolios (the common case for a long-term investor), the difference is small. The pooled-rate approach is a deliberate design choice: it produces a smooth, stable combined projection that avoids amplifying noise from short-lived rate differences between portfolios.

### Usage tips

**Setting monthly deposits to $0** strips out the effect of future contributions entirely. The formula simplifies to:

$$FV = \text{VAL} \cdot (1+r_m)^{12n} = \text{VAL} \times (1+r_r)^n$$

The two forms are mathematically identical:

$$(1+r_m)^{12n} = ((1+r_r)^{\frac{1}{12}})^{12n} = (1+r_r)^n$$
 
This isolates the compounding of your existing capital — what today's money becomes on its own, with no new saving. Running the tool twice, once at $0 and once at your planned monthly amount, gives you a natural decomposition:

- **$0 line** — growth of capital you already have
- **Gap between the two lines** — value added by future deposits

This is useful for understanding how much of your projected retirement portfolio is already "locked in" by past decisions vs. how much depends on continued saving.

**Setting the horizon to $0$** skips the accumulation phase entirely. The formula evaluates at $n=0$:

$$FV = \text{VAL} \cdot (1+r_m)^{0} + \text{PMT} \cdot \frac{(1+r_m)^{0} - 1}{r_m} = \text{VAL}$$

The end value equals today's VAL — no growth, no new deposits. The sustainability simulation then runs directly from the current portfolio, answering the question: *if I retired today, how long would this portfolio last at my target withdrawal rate?* This is the correct setup for someone already at retirement age or for stress-testing the current portfolio with no assumed future accumulation.

---

## Sustainability simulation

After the accumulation horizon, the portfolio enters a withdrawal phase. All calculations here use **real** values (today's purchasing power) throughout. The same $r_m = (1+r_r)^{\frac{1}{12}}-1$ used in the real projection line is used here.

**Perpetuity condition:**

$$W_\infty = \text{VAL}_\text{real} \times r_m$$

This is the maximum monthly withdrawal that leaves the portfolio intact forever — you only spend the monthly return, never the principal. If your portfolio is worth \$500K in real terms and your real return is 4%, the perpetuity withdrawal is:

$$\$500K \times \left(1.04^{\frac{1}{12}} - 1\right) \approx \$1{,}637\text{/month}$$

As long as you withdraw no more than that, the \$500K stays intact indefinitely.

If the planned monthly withdrawal $W \leq W_\infty$, the portfolio never depletes.

**Depletion simulation (if withdrawal > perpetuity):**

The simulation runs month by month: each month the portfolio earns one month of real return, then the withdrawal is subtracted. If the balance eventually reaches zero, the tool shows the age at depletion — your current age plus the accumulation horizon plus the years of withdrawals the portfolio sustained. If the balance never reaches zero, it shows "Never".

**Withdrawal table:**

A compact table shows the main scenario and, when an alternate real return is set, a second row for the alternate scenario. Each row shows: Real VAL at end of horizon, perpetuity monthly (the maximum withdrawal that leaves the portfolio intact forever), age at depletion, and years of withdrawal sustained. Clicking the perpetuity cell auto-fills the monthly withdrawal input.