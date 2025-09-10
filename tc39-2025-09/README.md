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
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 September 2025

---

## Progress update

- There are two implementations that pass 99% of the test262 tests.
- Today, we have one normative change to propose, which fixes a bug found by a user.

---

## Test conformance as of September 2025

<div class="twocol">
<div>

| Engine   | %PASS | Change |
| -------- | ----- | ------ |
| SM       | 99%   | ↓0.1%  |
| V8       | 99%   | ↑80%   |
| Ladybird | 97%   | ↑0.2%  |
| Boa      | 96%   | ↑6%    |
| GraalJS  | 90%   | ↓0.2%  |
| JSC      | 41%   | ↑0.5%  |

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': 9301,
    'V8': 9255,
    'Ladybird': 9034,
    'Boa': 9032,
    'GraalJS': 8447,
    'JSC': 3850,
  };
  const totalTests = 9361;
  // test/staging/sm tests have noStrict flag. it's too much hassle to
  // keep track of whether an implementation fails the noStrict tests,
  // so we just count strict mode and default as two separate tests,
  // which is what test262-harness does

  Chart.defaults.font.family = 'Rubik';
  Chart.defaults.font.size = 16;
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: Object.keys(results),
      datasets: [{
        label: '% of test262 passing',
        // do not use =>
        data: Object.values(results).map(function (x) { return x * 100 / totalTests }),
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
npx test262-harness --hostType=sm --hostPath=$HOME/.esvu/bin/sm -f Temporal "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --hostArgs=--harmony-temporal -- "test/**/*.js"
npx test262-harness --hostType=libjs --hostPath=$HOME/.esvu/bin/ladybird-js -f Temporal --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --hostArgs=--useTemporal=1 -- "test/**/*.js"
npx test262-harness --hostType=boa --hostPath=$HOME/.esvu/bin/boa-nightly -f Temporal -- "test/**/*.js"  # requires https://github.com/tc39/eshost/pull/147 and https://github.com/devsnek/esvu/pull/66
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Temporal --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
npx test262-harness --hostType=node --hostPath=$HOME/.local/bin/deno -f Temporal --hostArgs='run --unstable-temporal' -- "test/**/*.js"
-->

---

## Path to Stage 4

- Intl Era/Month Code moves to stage 3
- V8 unflags their implementation
- Remaining tests in staging are moved to main test262, updated and expanded as needed
- Identified gaps in test coverage are filled
- Temporal moves to stage 4, together with Time Zone Canonicalization and Intl Era/Month Code

---

## ZonedDateTime difference fix (PR [#3147](https://github.com/tc39/proposal-temporal/pull/3147))

- Bug reported by Patrick Hensley, a Temporal user
- Edge case when exact time and wall-clock time differences have opposite signs

```js
const d1 = Temporal.ZonedDateTime.from('2025-11-02T01:01-05:00[America/Chicago]');
const d2 = Temporal.ZonedDateTime.from('2025-11-02T01:00-06:00[America/Chicago]');
d1.until(d2)
  // => Temporal.Duration of +59 minutes
d1.toPlainDateTime().until(d2.toPlainDateTime())
  // => Temporal.Duration of -1 minute
// all good so far...
```

---

## ZonedDateTime difference fix (2)

```js
// ...but requesting a calendar largestUnit is broken
d1.until(d2, { largestUnit: 'years' })
  // => current: fails assertion
  // => should be: Temporal.Duration of +59 minutes
```

Proposing:
- The above change
- Similar change in `Temporal.Duration.prototype.round()`
- Similar change in `Temporal.Duration.prototype.total()`

---

## ZonedDateTime difference fix (3)

- Sadly this _used_ to work but we broke it with a refactor (to avoid extra user code calls) in Feb. 2024
- Mitigations to prevent this sort of regression in the future:
  - Added test262 coverage for this specific case
  - Wrote a script to test arithmetic with many different permutations of "interesting" ZonedDateTimes, PlainDateTimes, etc. and compare the results against a snapshot

---

<!-- _class: lead -->

## Questions?

---

<!-- _class: lead -->

## Requesting consensus
### Change ZonedDateTime difference and Duration round/total to handle DST edge case

---

# Proposed summary for notes

There are two almost completely conformant implementations, one still flagged. We outlined a path to stage 4 for the proposal and listed the blockers.

A normative change to fix a bug with daylight saving time arithmetic reached consensus. This bug was discovered in the wild by a user.
