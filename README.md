# Pay Calculator

A UK take-home pay calculator in a single, self-contained web page. Open `index.html` in any modern browser - there is nothing to install, build or serve, and no data leaves the page.

It works out income tax, National Insurance and pension deductions for a salary plus an optional retention allowance, then shows what happens to pay and annual leave if you drop 1, 2 or 3 working days a week.

## Inputs

| Field | Default | Notes |
|---|---|---|
| Annual income | - | Basic salary before any deductions. |
| Annual retention allowance | - | Optional. Treated as taxable, NI-able pay. |
| Contracted weekly hours | 37 | Used for the hours rows in the drop-days tables. |
| Working days per week | 5 | 1-7, half days allowed. Defines what "one day" is as a share of the week. |
| Annual leave (days per year) | 25 | Full entitlement for the current pattern. Include bank holidays if your contract includes them. |
| Pension contribution | 7.35% | Employee contribution as a percentage of pensionable pay. |
| Pension scheme type | Net pay | Net pay arrangement, salary sacrifice or relief at source. |
| Retention allowance is pensionable | Off | When ticked, the pension percentage also applies to the retention allowance. |

## Outputs

1. **Your pay** - gross pay, pension, income tax, National Insurance and take-home pay per year, month (÷ 12) and week (÷ 52).
2. **Dropping days** - current pattern versus −1, −2 and −3 days, showing days, hours, annual leave (days and hours), gross pay, each deduction, take-home pay, the monthly change, the share of pay worked and the share of take-home pay kept.
3. If a retention allowance is entered, the drop-days table appears **twice**: once with the allowance reduced pro rata and once with it kept at its full value, because employers treat it differently.

## Calculation rules

All figures are for the **2026/27 tax year** (6 April 2026 - 5 April 2027) and use **England, Wales and Northern Ireland** income tax rates.

### Gross pay

```
gross = annual income + retention allowance
```

### Income tax

1. **Personal allowance** is £12,570. For income above £100,000 it is reduced by £1 for every £2 over (rounded down to whole pounds), reaching £0 at £125,140.
2. **Taxable income** = taxable pay − personal allowance.
3. Taxable income is charged in bands:

| Band | Taxable income | Rate | Equivalent gross income (full allowance) |
|---|---|---|---|
| Basic | £0 - £37,700 | 20% | £12,571 - £50,270 |
| Higher | £37,701 - £125,140 | 40% | £50,271 - £125,140 |
| Additional | over £125,140 | 45% | over £125,140 |

"Taxable pay" is gross pay less any net-pay or salary-sacrifice pension contribution (see below). The personal allowance taper is applied to that same figure.

### National Insurance (employee Class 1)

| Earnings (annual) | Rate |
|---|---|
| Up to £12,570 (Primary Threshold) | 0% |
| £12,570 - £50,270 (Upper Earnings Limit) | 8% |
| Over £50,270 | 2% |

NI is calculated on the **annual** figure. Real payroll calculates NI per pay period, so results match exactly only when pay is the same every period.

### Pension

```
pensionable pay = annual income (+ retention allowance if ticked)
contribution    = pensionable pay × percentage
```

How the contribution affects tax and NI depends on the scheme type:

| Scheme type | Reduces pay for income tax | Reduces pay for NI | Taken from take-home pay |
|---|---|---|---|
| Net pay arrangement | Yes | No | 100% of contribution |
| Salary sacrifice | Yes | Yes | 100% of contribution |
| Relief at source | No | No | 80% of contribution (the provider adds 20% basic-rate relief) |

Relief at source simplifications:

- Higher and additional-rate relief, claimed through Self Assessment, is not included.
- Relief-at-source contributions should also reduce the income used for the £100,000 personal allowance taper; this is not modelled.

### Take-home pay

```
take-home = gross − pension deduction − income tax − NI
```

### Dropping days (pro rata)

For each scenario:

```
ratio          = (working days − days dropped) ÷ working days
income         = annual income × ratio
retention      = retention allowance × ratio   (pro-rata table)
               = retention allowance           (fixed table)
hours per week = weekly hours × ratio
leave (days)   = annual leave × ratio
leave (hours)  = leave (days) × (weekly hours ÷ working days)
```

- Tax, NI and pension are recalculated from scratch for each scenario; they are not scaled.
- The pension stays at the same percentage of the reduced pensionable pay.
- A day is assumed to stay the same length, so leave hours scale with leave days.
- Scenarios that would leave zero working days are not shown (e.g. a 3-day week shows only −1 and −2).
- Leave is not rounded. Many employers round part-time leave up to the nearest half day.

## What is not included

- Scottish income tax rates and bands (different from the rest of the UK)
- Tax codes other than the standard 1257L, other income, benefits in kind, Marriage Allowance
- Student and postgraduate loan repayments
- Higher-rate relief on relief-at-source pensions
- Employer pension and employer NI contributions
- Per-pay-period NI and tax tables (everything is annualised)
- Pension tier changes (see below)

## What needs updating, and when

All tax and NI values live in the `TAX_YEAR` object at the top of the `<script>` in `index.html`.

| What | When to check | Source |
|---|---|---|
| Tax year label, personal allowance, taper threshold, income tax bands and rates | Every year before **6 April**, after the Autumn Budget (usually October/November) and the Spring Statement | [GOV.UK: Income Tax rates and allowances](https://www.gov.uk/government/publications/rates-and-allowances-income-tax) |
| NI Primary Threshold, Upper Earnings Limit and rates | Every year before **6 April**, and after any fiscal event. NI rates have changed mid-year before (January 2024) | [GOV.UK: Rates and thresholds for employers](https://www.gov.uk/guidance/rates-and-thresholds-for-employers-2026-to-2027) |
| Relief-at-source rate (uses the basic rate, so it follows the first band automatically) | Only if the basic rate changes | - |
| Salary sacrifice NI treatment | Before **6 April 2029**. From 2029/30, only the first £2,000 a year of pension contributions made by salary sacrifice is expected to be exempt from NI; the calculator currently exempts all of it | [GOV.UK: Budget 2025](https://www.gov.uk/government/topical-events/budget-2025) |
| Default pension contribution (7.35%) | Every April, or when your pay changes. Many schemes (especially public-sector schemes) set the rate by pay tier, and dropping days may move you into a different tier - the calculator keeps the same rate in every scenario | Your pension scheme |
| Default annual leave (25 days) and hours (37) | When your contract changes | Your employer |

Income tax thresholds (personal allowance and higher-rate threshold) and the NI thresholds are currently frozen. Even so, check them every year, since freezes can be extended, ended or changed at any fiscal event.

## Disclaimer

This calculator is provided "as is", without warranty of any kind, express or implied. Results are estimates for general guidance only and may contain errors or omissions, or be out of date. They are not financial, tax or legal advice. The authors accept no responsibility or liability for any loss or decision arising from its use. Check figures with your employer, payroll department, pension scheme or HMRC before relying on them.
