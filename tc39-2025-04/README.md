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
TC39 April 2025

---

## Progress update

Temporal is shipping in Firefox Nightly builds. Some open questions have been raised about coordinating locale-defined behaviour between implementations, which TG2 is addressing. We will continue to analyze code coverage metrics, submit additional 
test262 tests, and answer any questions raised.

---

## Test conformance as of April 2025

<div class="twocol">
<div>

| Engine   | %PASS | Change |
| -------- | ----- | ------ |
| SM       | ~100% | ↓0.02% |
| Ladybird | 96%   | ↓0.4%  |
| GraalJS  | 91%   | ↑1.6%  |
| Boa      | 85%   | ↑13%   |
| V8       | 73%   | ↓0.5%  |
| JSC      | 40%   | ↓0.1%  |

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': 9137,
    'Ladybird': 8823,
    'GraalJS': 8298,
    'Boa': 7758,
    'V8': 6690,
    'JSC': 3647,
  };
  const totalTests = 9157;
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
npx test262-harness --hostType=sm --hostPath=$HOME/workspace/mozilla-unified/obj-debug-x86_64-pc-linux-gnu/dist/bin/js -f Temporal "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --hostArgs=--harmony -- "test/**/*.js"
npx test262-harness --hostType=libjs --hostPath=$HOME/.esvu/bin/ladybird-js -f Temporal --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --hostArgs=--useTemporal=1 -- "test/**/*.js"
npx test262-harness --hostType=boa --hostPath=$HOME/.esvu/bin/boa -f Temporal -- "test/**/*.js"  # requires https://github.com/tc39/eshost/pull/147
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Temporal --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
npx test262-harness --hostType=node --hostPath=$HOME/.local/bin/deno -f Temporal --hostArgs='run --unstable-temporal' -- "test/**/*.js"
-->

---

## Internal calculations

In the past there were performance concerns about the use of BigInts in the spec. I showed in a previous update that 75 integer bits were enough.

Based on a paper<sup>1</sup>, I wrote a proof of concept representing epoch nanoseconds and time durations as a pair of 64-bit floats each, so that you don't have to deal with nonstandard-sized integers. Let me know if interested for your implementation.

<span style="font-size:0.6em;"><sup>1</sup> Hida, Li, Bailey (2008). _Library for double-double and quad-double arithmetic._ Technical report, Lawrence Berkeley National Laboratory. https://www.davidhbailey.com/dhbpapers/qd.pdf</span>

---

<!-- _class: lead -->

## Questions?

---

# Proposed summary for notes

---

---

<!--
_class: invert lead
-->

# Guidelines for Locale-Sensitive Testing in Test262

**Philip Chimento**
Igalia
TC39 April 2025

---

## ILD

Implementation- and locale-defined.

```js
new Date().toLocaleString('en', { dayPeriod: 'long' })
// → "in the afternoon"
```

> iii. Let _fv_ be a String value representing the day period of _tm_ in the form given by _f_; the String value depends upon the implementation and the effective locale of _dateTimeFormat_.

---

## Stability in ILD behaviour

It's good for users of the web when...

- ILD behaviour is **stable**, so that websites don't break
- ILD behaviour is **updated** to follow changing cultural practices & better understanding thereof, so that websites are localized in a way that makes users comfortable

These two things are **both good**, and **directly in tension**

---

## Test coverage

* Should it be a goal to cover every combination of locale and options?
* IMO, no!
* Diminishing returns, after a certain point you are just testing CLDR

---

## Some existing strategies for ILD testing
#### (that are not ideal)

- "Golden" output
- "Mini-implementation"

---

## Golden output

- Testing jargon: comparing output of method under test to known-good output
- Undesirable in test262 because:
  - golden output varies between implementations
  - golden output varies over time
  - variations are permitted by specification
  - test suite can only reasonably be run by an implementation using a specific version of CLDR, and if you aren't using CLDR, forget it

---

## Mini-implementation

- Including what is essentially a polyfill in the test, comparing its output to that of method under test
- Undesirable in test262 because:
  - difficult to understand what's being tested
  - unclear whether the polyfill or implementation is at fault when tests fail

---

## But what to do instead?

- Stable substrings?
- Comparative testing?
- [Metamorphic testing?](https://www.hillelwayne.com/post/metamorphic-testing/)

---

## Stable substrings?

- Not golden output, but a part of the output that is reasonably expected to be stable
- More robust than golden output
- Still shares some disadvantages

```js
const formatter = new Intl.DateTimeFormat("en", { dateStyle: "full" });
const result = formatter.format(new Date(2024, 9, 24, 12));
assert(result.indexOf("October") > -1);
```

---

## Comparative testing?

- Each setting for an input option must produce a distinct output
- Could be good for getting coverage
- Assumption doesn't hold in all cases

```js
const date = new Date(2024, 9, 24, 12);
const weekdayResults = ["narrow", "short", "long"].map((weekday) => date.toLocaleString("en", { weekday }));
assertNoDuplicates(results);

// BUT:
const dayResults = ["numeric", "2-digit"].map((day) => date.toLocaleString("en", { day }));
assertNoDuplicates(dayResults); // ⚠️ fails for days ≥ 10
```

---

## Metamorphic testing?

- Find invariant properties of outputs that must hold across multiple inputs
- No need to specify what those outputs exactly are
- Can be difficult to find these properties if not specified

```js
const locale = /* any locale, with any calendar or any numbering system */;
const date = new Date(2024, 9, 24, 12);
const dayString = date.toLocaleString(locale, { day: "numeric" });
const monthName = date.toLocaleString(locale, { month: "long" });
const fullDateString = date.toLocaleString(locale, { dateStyle: "full" });
assert(fullDateString.indexOf(dayString) > -1);
assert(fullDateString.indexOf(monthName) > -1);
```

---

## Discussion

- Further thoughts welcome!
- What kind of guidelines would be helpful for implementations?
- More info: https://github.com/tc39/test262/issues/3786

---

# Proposed summary for notes
