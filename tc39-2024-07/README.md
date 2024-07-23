---
marp: true
theme: gaia
paginate: true
style: |
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  @import url('https://fonts.googleapis.com/css2?family=Rubik:ital,wght@0,400;0,700;1,400;1,700&display=swap');
  code { font-family: Hack; }
  section { font-family: Rubik, sans-serif; letter-spacing: 0; }
  section.lead.invert { text-shadow: 0 0 10px black, 0 0 20px black; }
  section.smaller-type li { font-size: 85% }
  marp-pre code, marp-pre { background-color: #042029; }
  .hljs-string { color: #8ae234; }
  .hljs-number, .hljs-literal { color: #729fcf; }
  .hljs-params { color: #e9b96e; font-style: italic; }
  .hljs-built_in { color: #fce94f; font-weight: bold; }
  .hljs-title.class_ { color: #fce94f; text-decoration: none; font-weight: bold; }
  .hljs-title.function_ { color: #c8a8c4; }
  .hljs-keyword { color: #fcaf3e; font-weight: bold; }
  .hljs-attr { color: #e9b96e; }
  .hljs-variable { color: red; font-weight: bold; }
  /* .hljs-comment, .hljs-regexp, .hljs-symbol */
  .twocol {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

<!--
_class: invert lead
footer: DRAFT
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 July 2024

---

## Progress update

After 7 years, Temporal is close to done. Our only focus is making sure in-progress implementations are successful, by fixing corner-case bugs found by implementors, and making editorial changes to improve clarity and simplicity.

We trust that the smaller API we've had since June has resolved all concerns about complexity and code size. Please speak up ASAP if not.

---

## Ship it! 🐿️

At this point we see no reason to delay! Please go ahead with implementations and **ship them, unflagged** when they are ready.
- If something is preventing you from shipping Temporal, let us know and work with us to resolve ASAP
- Don't wait! If we need to make changes, we want to make them now

You are welcome at the **Temporal champions meeting, biweekly Thursdays at 08:00 Pacific** (or we will happily meet another time if this time doesn't work for you)

---

## Test conformance as of July 2024

<div class="twocol">
<div>

- SpiderMonkey: 96%
- V8: 75%
- LibJS: 75%
- JavaScriptCore: 40%
- Boa: 23% (pending 32%)

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  Chart.defaults.font.family = 'Rubik';
  Chart.defaults.font.size = 16;
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: ['SM', 'V8', 'LibJS', 'JSC', 'Boa'],
      datasets: [{
        label: '% of test262 passing',
        data: [4044, 3150, 3143, 1675, 984].map(function (x) { return x * 100 / 4215 }),
        backgroundColor: '#a40000',
      }],
    },
    options: {
      aspectRatio: 1.4,
      indexAxis: 'y',
    },
  });
</script>

<!--
npx test262-harness --hostType=sm --hostPath=$HOME/workspace/gecko/obj-debug-x86_64-pc-linux-gnu/dist/bin/js -f Temporal "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --hostArgs=--harmony-temporal -- "test/**/*.js"  # requires https://github.com/bterlson/eshost/pull/139
LD_LIBRARY_PATH=$HOME/.local/lib64 npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/ladybird-js -f Temporal --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --hostArgs=--useTemporal=1 -- "test/**/*.js"
cargo run --release --bin boa_tester -- run --test262-path $HOME/workspace/test262 -s ...
-->

---

## Follow up from June TC39

Make all `valueOf` methods of Temporal objects the same function object?

Do the same for `toJSON` methods?

Mozilla investigated the feasibility of this. Their position is that we should only do this if we still get feedback that Temporal is too large to ship after all other proposed changes have been implemented.

Therefore, we propose not to pursue this at this time.

---

<!-- _class: invert lead -->

# Bug fixes

---

### Start-of-day in Toronto in 1919 (PR [#2918](https://github.com/tc39/proposal-temporal/pull/2918))

The IANA time zone DB has a corner case: On 1919-03-30, Toronto switched to DST at 23:30, skipping the hour between 23:30 and 00:30. That means the day of 1919-03-31 started at 00:30. The algorithm for calculating start-of-day did not take this into account.

Affects ZonedDateTime.p.startOfDay, .hoursInDay, .withPlainTime, .round, PlainDate.p.toZonedDateTime, and parsing a date-only string with a time zone annotation.

Credit to Andrew Gallant, who is implementing a Rust library with operations based on Temporal.

---

### Start-of-day in Toronto in 1919: What changes

```js
edgeCase = Temporal.ZonedDateTime.from('1919-03-31T12:00[America/Toronto]');
edgeCase.hoursInDay
  // Current: 23
  // Proposed: 23.5
edgeCase.startOfDay()
edgeCase.withPlainTime()
Temporal.ZonedDateTime.from('1919-03-31[America/Toronto]')
Temporal.PlainDateTime.from('1919-03-31').toZonedDateTime('America/Toronto')
  // (all) Current: ZonedDateTime at 1919-03-31T01:00-04:00[America/Toronto]
  // Proposed: ZonedDateTime at 1919-03-31T00:30-04:00[America/Toronto]
```

---

### Ambiguous rounding operation (PR [#2916](https://github.com/tc39/proposal-temporal/pull/2916)) (1)

Rounding a duration to a number of calendar units >1, let's say to the next increment of 8 months, is fine. This is an anticipated and intended use case.

```js
const d1 = Temporal.Duration.from({ months: 9 });
d1.round({
  relativeTo: '2024-01-01',
  smallestUnit: 'months',
  roundingIncrement: 8,
  roundingMode: 'ceil',
});  // => 16 months
```

---

### Ambiguous rounding operation (PR [#2916](https://github.com/tc39/proposal-temporal/pull/2916)) (2)

Doing the same while simultaneously balancing to a larger calendar unit is unclear what the programmer should expect:

```js
d1.round({
  relativeTo: '2024-01-01',
  largestUnit: 'years',  // <= added
  smallestUnit: 'months',
  roundingIncrement: 8,
  roundingMode: 'ceil',
});  // => ???
```

Should this be 1 year? 1 year 4 months? Neither is obviously right.

---

### Ambiguous rounding operation (PR [#2916](https://github.com/tc39/proposal-temporal/pull/2916)) (3)

- Probably we just never considered this case
- Did not come up in real-world use, but as a result of an implementor looking for corner cases
- To support would ideally require research into real use cases
- Would complicate the rounding algorithm for dubious benefit
- Given where we are in the proposal, prefer to remove/simplify

---

### Ambiguous rounding operation (PR [#2916](https://github.com/tc39/proposal-temporal/pull/2916)) (4)

Proposal is to throw RangeError on the following combination of options in `Temporal.Duration.prototype.round`:
- `roundingIncrement` &gt; 1
- `smallestUnit` years, months, weeks, or days
- `largestUnit` ≠ `smallestUnit`

Credit to Adam Shaw, who is implementing a Temporal polyfill, and also again to Andrew Gallant.

---

<!-- _class: lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

PR [#2916](https://github.com/tc39/proposal-temporal/pull/2916) and [#2918](https://github.com/tc39/proposal-temporal/pull/2918)

---

# Proposed summary for notes

> Consensus was reached on two normative changes: one to fix a TZDB corner case in calculating the start-of-day of March 31, 1919 in Ontario, Canada, and another to disallow a particular ambiguous combination of options in `Temporal.Duration.prototype.round()`.
> 
> Implementations should complete work on the proposal and ship it, and let the champions know ASAP if anything is blocking or complicating that. Follow the checklist in [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates or feel free to join the champions meetings.

---
