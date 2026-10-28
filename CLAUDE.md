# CLAUDE.md

UK take-home pay calculator. See `README.md` for the full calculation rules, assumptions and the yearly update schedule — keep it in sync with any change to the logic.

## Constraints

- **Single self-contained file.** Everything (HTML, CSS, JS) lives in `index.html`. No external scripts, stylesheets, fonts, build step or dependencies.
- Plain HTML5 + vanilla JS. No frameworks.
- UK English (`en-GB`) in UI text and number formatting.
- Not a git repository at the time of writing.

## Structure of `index.html`

- `<style>` — CSS custom properties on `:root`; `main` is capped at 720px so the 5-column comparison tables fit. `.table-scroll` handles narrow screens.
- `<form id="pay-form">` — inputs; defaults are set with `value=` attributes (37 hours, 5 days, 25 days leave, 7.35% pension).
- `<script>`:
  - `TAX_YEAR` — **all** tax and NI rates/thresholds. This is the only place yearly figures should live.
  - `personalAllowance()`, `incomeTax()`, `nationalInsurance()` — pure functions over annual amounts.
  - `calculatePay()` — annual breakdown for one set of inputs, including pension scheme handling. Returns `{ gross, pension, pensionDeduction, taxablePay, tax, ni, net }`.
  - `gbp`, `pct`, `num` — shared `Intl.NumberFormat` formatters.
  - `window.onload` — wires the form; on submit it validates, calls `calculatePay()`, builds the drop-days scenarios (`buildScenarios(retentionProRata)`) and renders via template strings. `compareTable()` renders a drop-days table.

## Conventions

- Calculations run on unrounded annual values; rounding happens only in display formatting.
- Add new rates/thresholds to `TAX_YEAR` rather than hard-coding numbers in functions.
- Scenario logic should reuse `calculatePay()` — recalculate tax/NI per scenario, never scale them.
- Results are rendered as HTML strings into `#results`; match the existing `row()` / `compareTable()` pattern.
- When adding a new assumption or simplification, add it to the result notes on the page and to `README.md`.

## Testing

There is no test suite. Verify changes by running the page in jsdom from the scratchpad (not the project directory), setting input values, dispatching `submit` on `#pay-form` and reading `#results`:

```js
const { JSDOM } = require('jsdom');
const dom = new JSDOM(fs.readFileSync('index.html', 'utf8'), { runScripts: 'dangerously' });
const w = dom.window, d = w.document;
w.onload();
d.getElementById('annual-income').value = '60000';
// ...set other fields...
d.getElementById('pay-form').dispatchEvent(new w.Event('submit', { cancelable: true }));
```

Reference values (2026/27, net pay arrangement):

| Gross | Pension | Income tax | NI |
|---|---|---|---|
| £30,000 | none | £3,486.00 | £1,394.40 |
| £60,000 | none | £11,432.00 | £3,210.60 |
| £150,000 | none | £53,703.00 | £5,010.60 |
| £60,000 salary + £5,000 retention allowance | 5% of salary, net pay | £12,232.00 | £3,310.60 |

Same last row with salary sacrifice: NI £3,250.60. With relief at source: pension deduction £2,400, tax £13,432.00.

## Yearly update

Before each 6 April (and after any Budget), update `TAX_YEAR` and its comment, then re-check the reference values above and the "What needs updating" table in `README.md`. Before 6 April 2029, model the £2,000 cap on NI-free salary sacrifice pension contributions.
