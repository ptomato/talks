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
TC39 November 2025

---

## Progress update

- There are two implementations that pass 99% of the test262 tests.
- We've hunted down implementation divergences using a snapshot testing technique.
- Today, we have one normative change to propose, found using snapshot testing.

---

## Test conformance as of November 2025

<div class="twocol">
<div>

| Engine   | %PASS | Change |
| -------- | ----- | ------ |
| SM       | 99%   | ↓0.07% |
| V8       | 99%   | ↑0.4%  |
| Kiesel   | 97%   | ↑1%    |
| Boa      | 97%   | ↑0.3%  |
| Ladybird | 96%   | ↓0.2%  |
| GraalJS  | 96%   | ↑6%    |
| JSC      | 48%   | ↑7%    |

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': 9615,
    'V8': 9609,
    'Kiesel': 9412,
    'Boa': 9379,
    'Ladybird': 9323,
    'GraalJS': 9313,
    'JSC': 4657,
  };
  const totalTests = 9684;
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
        backgroundColor: '#a40000d0',
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
npx test262-harness --hostType=boa --hostPath=$HOME/.esvu/bin/boa-nightly -f Temporal -- "test/**/*.js"  # requires https://github.com/tc39/eshost/pull/147 and https://github.com/devsnek/esvu/pull/66
npx test262-harness --hostType=hermes --hostPath=$HOME/.esvu/bin/kiesel-nightly -f Temporal -- "test/**/*.js"
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Temporal --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
# note: have to kill process on test/staging/sm/Temporal/PlainMonthDay/from-chinese-leap-month-common.js (both strict and sloppy mode)
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --hostArgs=--useTemporal=1 -- "test/**/*.js"
npx test262-harness --hostType=node --hostPath=$HOME/.local/bin/deno -f Temporal --hostArgs='run --unstable-temporal' -- "test/**/*.js"
-->

---

## Path to Stage 4

- V8 implementation unflagged in trunk, scheduled for Dec 3 beta and mid-January release
- Remaining tests in staging are moved to main test262, updated and expanded as needed
- Identified gaps in test coverage are filled
- Temporal moves to stage 4, together with Time Zone Canonicalization and Intl Era/Month Code

---

## Test coverage and implementation divergences

- Introduced a snapshot testing technique
- Testing millions of combinations of "interesting" inputs
- Flagging for further inspection when:
  - an assertion fails
  - an assumed invariant does not hold (e.g. _a_ + _b_ = _a_ - -_b_)
  - results are different between implementations

<!--
"interesting" i.e. chosen to be somewhat likely to find bugs?
-->

---

## Check it out!

[tc39/proposal-temporal ⋄ polyfill/test/thorough/](https://github.com/tc39/proposal-temporal/tree/main/polyfill/test/thorough)

```
./all.sh "$path_to_your_interpreter_with_any_needed_cli_args"
```

(requires an interpreter with JSON imports support)

---

## Results: worth it!

- **Spec bug**: [#3168](https://github.com/tc39/proposal-temporal/issues/3168) (more on this shortly)
- **SpiderMonkey bug**: [#1998020](https://bugzilla.mozilla.org/show_bug.cgi?id=1998020)
- **JavaScriptCore bugs**: [#301659](https://bugs.webkit.org/show_bug.cgi?id=301659), [#301710](https://bugs.webkit.org/show_bug.cgi?id=301710), [#301717](https://bugs.webkit.org/show_bug.cgi?id=301717), [#301724](https://bugs.webkit.org/show_bug.cgi?id=301724)
- **temporal_rs bugs**: [#610](https://github.com/boa-dev/temporal/pull/610), [#613](https://github.com/boa-dev/temporal/issues/613), [#617](https://github.com/boa-dev/temporal/issues/617)
- **V8 bugs unrelated to temporal_rs**: [#446728405](https://issues.chromium.org/issues/446728405) (already filed but stale; added a current reproducer)
- **GraalJS bugs**: Found ≥4 distinct ones. Will open issues soon.
- **Test262**: Added specific test coverage for all of these bugs.

---

## More to come

- We will continue to add more of these snapshot tests
- Technique may be of interest to other proposal authors

---

## Assertion failure in Duration rounding

```js
> d = Temporal.Duration.from({ years: 1, hours: 1 });
> d.round({ smallestUnit: 'years', relativeTo: '2020-02-29', roundingMode: 'floor' });
  // Current: duration of 0 length (and assertion failure in Firefox debug builds)
  // Proposed: duration of 1 year
```

- Duration rounding and totalling uses a "bounding" algorithm
- We construct two bounding Durations _start_ and _end_ that are multiples of a rounding increment
- Such that _start_ ≤ _d_ < _end_
- Round based on whether _d_ is closer to _start_ or _end_
- Total based on fraction _d_ - _start_ / _end_ - _start_

---

## Assertion failure in Duration rounding (2)

```js
> d = Temporal.Duration.from({ years: 1, hours: 1 });
> d.round({ smallestUnit: 'years', relativeTo: '2020-02-29', roundingMode: 'floor' });
  // Current: duration of 0 length (and assertion failure in Firefox debug builds)
  // Proposed: duration of 1 year
```

- Via snapshot testing, found an edge case
- Algorithm generates bounds where _start_ ≤ _d_ < _end_ does not hold
- Fix in normative [PR #3172](https://github.com/tc39/proposal-temporal/pull/3172)
- h/t Tim Chevalier

---

## Assertion failure in Duration rounding (3)

- Normative?
- Usually we say that assertion failures are an editorial error
- In this case, ignoring assertions can give an incorrect result, which has been exposed to the Web. Thus, normative

<!-- 
There were cases with some inputs where the assertion was hit but the method still produced the correct answer when assertions were switched off in release builds. But there were also cases where it didn't.
-->

---

<!-- _class: lead -->

## Questions?

---

<!-- _class: lead -->

## Requesting consensus

---

# Proposed summary for notes

There are two almost completely conformant implementations, one still flagged. We outlined a path to stage 4 for the proposal and listed the blockers.

...
