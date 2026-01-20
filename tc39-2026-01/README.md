---
marp: true
theme: gaia
math: mathjax
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
  .chart { max-width: 60%; margin: 0 auto; }
---

<!--
_class: invert lead
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 January 2026

![w:200](igalia.svg)

---

## Progress update

- Test262 coverage has been greatly expanded, exposing many implementation bugs.
- We have 2 implementations at ~100% test conformance.
- Today, we have 2 normative changes to propose to eliminate surprising behaviour in edge cases.

---

## Test conformance as of January 2026

<div class="chart">
  <canvas id="conformance-chart"></canvas>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.5.1"></script>
<script src="https://cdn.jsdelivr.net/npm/patternomaly@1.3.2"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'V8': { Temporal: 9882, Monthcode: 2930 },
    'SM': { Temporal: 9888, Monthcode: 2652 },
    'Kiesel': { Temporal: 9648, Monthcode: 2514 },
    'Boa': { Temporal: 9628, Monthcode: 2488 },
    'GraalJS': { Temporal: 9418, Monthcode: 800 },
    'Ladybird': { Temporal: 9590, Monthcode: 464 },
    'JSC': { Temporal: 5828, Monthcode: 80 },
  };
  const totalTests = { Temporal: 9912, Monthcode: 2986 };
  // test/staging/sm tests have noStrict flag. it's too much hassle to
  // keep track of whether an implementation fails the noStrict tests,
  // so we just count strict mode and default as two separate tests,
  // which is what test262-harness does

  const barData = {};
  for (const tag of ['Temporal', 'Monthcode']) {
    // do not use ()=>{}
    barData[tag] = Object.values(results).map(function (result) {
      return result[tag] * 100 / totalTests[tag];
    });
  }

  Chart.defaults.font.family = 'Rubik';
  Chart.defaults.font.size = 16;
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: Object.keys(results),
      datasets: [{
        label: 'Temporal',
        data: barData.Temporal,
        backgroundColor: '#a40000d0',
      }, {
        label: 'Intl Era/Month Code',
        data: barData.Monthcode,
        backgroundColor: pattern.draw('diagonal', '#204a87'),
      }],
    },
    options: {
      aspectRatio: 1.4,
      label: '% of test262 passing',
      indexAxis: 'y',
      scales: {
        x: {
          title: {
            display: true,
            text: '% test conformance',
          },
        },
      },
    },
  });
</script>

<!--
npx test262-harness --hostType=sm --hostPath=$HOME/.esvu/bin/sm -f Temporal --fe Intl.Era-monthcode "test/**/*.js"
npx test262-harness --hostType=sm --hostPath=$HOME/.esvu/bin/sm -f Intl.Era-monthcode "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --fe Intl.Era-monthcode --hostArgs=--harmony-temporal -- "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Intl.Era-monthcode --hostArgs=--harmony-temporal -- "test/**/*.js"
npx test262-harness --hostType=libjs --hostPath=$HOME/.esvu/bin/ladybird-js -f Temporal --fe Intl.Era-monthcode --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=libjs --hostPath=$HOME/.esvu/bin/ladybird-js -f Intl.Era-monthcode --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Temporal --fe Intl.Era-monthcode --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Intl.Era-monthcode --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --fe Intl.Era-monthcode --hostArgs=--useTemporal=1 -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Intl.Era-monthcode --hostArgs=--useTemporal=1 -- "test/**/*.js"
npx test262-harness --hostType=boa --hostPath=$HOME/.esvu/bin/boa-nightly -f Temporal --fe Intl.Era-monthcode -- "test/**/*.js"  # requires https://github.com/tc39/eshost/pull/147 and https://github.com/devsnek/esvu/pull/66
npx test262-harness --hostType=boa --hostPath=$HOME/.esvu/bin/boa-nightly -f Intl.Era-monthcode -- "test/**/*.js"
npx test262-harness --hostType=kiesel --hostPath=$HOME/.esvu/bin/kiesel-nightly -f Temporal --fe Intl.Era-monthcode -- "test/**/*.js"
npx test262-harness --hostType=kiesel --hostPath=$HOME/.esvu/bin/kiesel-nightly -f Intl.Era-monthcode -- "test/**/*.js"
npx test262-harness --hostType=node --hostPath=$(which node) -f Temporal --hostArgs=... -- "test/**/*.js"
npx test262-harness --hostType=hermes --hostPath=$(which deno) -f Temporal --hostArgs='run --unstable-temporal --v8-flags=--expose-gc' -- "test/**/*.js"
-->

---

## Path to Stage 4

- V8 implementation is now available on the Web!
- GraalJS implementation scheduled for unflagged release
- Intl Era/Month Code request for stage 3 in this meeting
- Investigate last conformance bugs in implementations
- Move remaining tests in staging to main test262, update and expand as needed
- Fill identified gaps in test coverage
- Temporal moves to stage 4, together with Intl Era/Month Code

---

## Surprise in PlainYearMonth subtract

```js
> Temporal.PlainYearMonth.from("2025-07").subtract({ months: 1 }, { overflow: 'reject' })
  // Current:  RangeError
  // Proposed: Temporal.PlainYearMonth of 2025-06
```

- Bug has been present in the spec for a _long_ time (>5 years)
- Found using snapshot testing technique
- Caused by a bug in the addition/subtraction code that existed to accommodate durations with days

---

## Surprise in PlainYearMonth subtract (2)

- Temporal champions recommend removing the subtract-days-from-PlainYearMonth feature ([PR #3253](https://github.com/tc39/proposal-temporal/pull/3253))
```js
> Temporal.PlainYearMonth.from("2025-07").add({ days: 31 })
  // Current:  Temporal.PlainYearMonth of 2025-08
  // Proposed: RangeError
```
- This also fixes the bug on the previous slide
- :hourglass: This is a late addition to the agenda
- Fallback bugfix in [PR #3208](https://github.com/tc39/proposal-temporal/pull/3208) (h/t Tim Chevalier)

---

## Surprise in Plain types' toLocaleString

```js
> Temporal.PlainDateTime.from('2026-03-08T02:00').toLocaleString()
  // Current:  "2026-03-08, 3:00:00 a.m." (in my locale)
  // Proposed: "2026-03-08, 2:00:00 a.m." (in my locale)
> Temporal.PlainDate.from('2011-12-30').toLocaleString('en-ca', { timeZone: 'Pacific/Apia' })
  // Current:  "2011-12-31"
  // Proposed: "2011-12-30"
```

- h/t fabon-f and Adam Shaw, JS community members developing tools downstream of Temporal
- Plain types are wall-clock times
- Should not be subject to the formatter's time zone
- Fix in normative [PR #3246](https://github.com/tc39/proposal-temporal/pull/3246)

---

## Summary of Intl Era/Month Code changes

Will be discussed in the other presentation, but noting here:
- [PR #99](https://github.com/tc39/proposal-intl-era-monthcode/pull/99) - Intl Era/Month Code must support only the listed calendars
- [PR #101](https://github.com/tc39/proposal-intl-era-monthcode/pull/101) - clarification of date difference behaviour
  - Note corresponding editorial change in Temporal, to keep the two proposals in sync: [PR #3245](https://github.com/tc39/proposal-temporal/pull/3245)
- [PR #108](https://github.com/tc39/proposal-intl-era-monthcode/pull/108) - sets an expectation for implementation-defined behaviour regarding nonsensical lunar dates in PlainMonthDay

---

<!-- _class: lead -->

## Questions?

---

<!-- _class: lead -->

## Requesting consensus

---

# Proposed summary for notes

We outlined a path to stage 4 for the proposal and listed the blockers.

Two normative changes reached consensus, to eliminate surprising behaviour in:
- Temporal.PlainYearMonth subtraction
- toLocaleString methods of Temporal.Plain___ types.

We summarized the related changes happening in the Intl Era/Month Code proposal as it goes to stage 3.
