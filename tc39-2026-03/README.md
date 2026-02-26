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
footer: <b>DRAFT</b>
---

<!--
_class: invert lead
-->

# ⌚ **Temporal for Stage 4** :tada:

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 March 2026

![w:200](igalia.svg)

---

## Stage 4 Entrance Criteria

- Two compatible implementations which pass the Test262 acceptance tests
- Significant in-the-field experience with shipping implementations, such as that provided by two independent VMs
- A pull request has been sent to tc39/ecma262 or tc39/ecma402, as appropriate, with the integrated spec text
- The relevant editor group has signed off on the pull request 

---

## Two Implementations Passing Tests

- Test262 covers every feature, and all the edge cases we've encountered
- We have 2 implementations at ~100% test conformance

---

## Test conformance as of March 2026

<div class="chart">
  <canvas id="conformance-chart"></canvas>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.5.1"></script>
<script src="https://cdn.jsdelivr.net/npm/patternomaly@1.3.2"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': { Temporal: 9952, Monthcode: 3054 },
    'V8': { Temporal: 9932, Monthcode: 3048 },
    'Kiesel': { Temporal: 9684, Monthcode: 2690 },
    'Ladybird': { Temporal: 9658, Monthcode: 462 },
    'Boa': { Temporal: 9618, Monthcode: 2464 },
    'GraalJS': { Temporal: 9408, Monthcode: 796 },
    'JSC': { Temporal: 5814, Monthcode: 78 },
  };
  const totalTests = { Temporal: 9970, Monthcode: 3056 };
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
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --fe Intl.Era-monthcode -- "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Intl.Era-monthcode -- "test/**/*.js"
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

## In-the-field Experience

2 browser-based implementations have been shipping Temporal unflagged for several months.
- **SpiderMonkey**: May 2025
- **V8**: January 2026

Non-browser engines shipping unflagged implementations: **Boa**, **Kiesel**, **Ladybird**.

**GraalJS** has an implementation, scheduled to be unflagged in 25.1 (no public release date)

---

## In-the-field Experience

Several polyfills exist (figures are gzipped size and weekly downloads):
- https://github.com/fullcalendar/temporal-polyfill (20kB, 891k)
- https://github.com/js-temporal/temporal-polyfill (47kB, 618k)
- https://github.com/fabon-f/temporal-polyfill-lite (19kB, 3k)

---

## In-the-field Experience

Shipping Temporal to the web, and having community-authored polyfills, has led to tangible improvements to the proposal during the 5 years of stage 3.
- Implementors & polyfill authors noticing bugs
- Implementors & polyfill authors noticing opportunities for improving the spec algorithms
- Users noticing bugs and surprising edge cases

**Thank you for all these contributions!**

---

## In-the-field Experience

Shipping Temporal to the web has also enabled automated detection of edge cases and divergences between implementations, by testing each implementation with millions of inputs against snapshots.
- This has led to filing 16+ bug reports in implementations
  - (and also fixing some of them)
- Identified many cases for more targeted test262 coverage

**This snapshot testing technique was helpful and we recommmend it for smaller proposals too!**

---

## Pull request to the spec

- ECMA-262: https://github.com/tc39/ecma262/pull/3759
- ECMA-402: https://github.com/tc39/ecma402/pull/1044
  - (also includes the Intl Era/Month Code proposal stage 4 PR, topic of a separate presentation in this meeting)

---

## Editor signoff

- ECMA-262 Editors:
  - We have discussed the PR in the editor call
  - There is an open question about how to publish the proposal

- ECMA-402 Editors:
  - TBA

---

## Form in which to Publish Temporal

- Stick it in ECMA-262 like any other proposal
- Give it a separate page in ECMA-262 (would be ECMA-262-2)
- Make a separate standard (would be "ECMA-430-ish")

---

## TODO: Slides for pros and cons

---

<!-- _class: lead -->

## Questions?

---

<!-- _class: lead -->

## Requesting consensus for Stage 4

---

## Finally, Some Statistics

* Lines of spec text in ECMA-262: **12k**
* Lines of spec text in ECMA-402: **1.6k**
* Current and former proposal champions: **9?**
* Total word count of champions meeting minutes: **212k**
  * That is, number of average-sized novels: **2** to **3**

---

## Finally, Some Statistics

How many years since [Maggie's kickoff blog post](https://maggiepint.com/2017/04/09/fixing-javascript-date-getting-started/)?
```js
Temporal.PlainDate.from('2017-04-09')
  .until(Temporal.Now.plainDateISO(), { smallestUnit: 'months', largestUnit: 'years' })
  .toLocaleString()
```

* ## '8 yrs, 11 mos'

---

# Proposed summary for notes
