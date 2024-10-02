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
TC39 October 2024

---

## Progress update

After 7 years, Temporal is close to done. We have cleaned out the issue tracker and finished all the editorial changes that anyone has requested. We will continue to analyze code coverage metrics and submit additional test262 tests.

---

## Ship it! 🐿️

Please go ahead with implementations and **ship them, unflagged** when they are ready.
- If something is preventing you from shipping Temporal, let us know and work with us to resolve ASAP
- Don't wait! If we need to make changes, we want to make them now

You are welcome at the **Temporal champions meeting, biweekly Thursdays at 08:00 Pacific** (or we will happily meet another time if this time doesn't work for you)

---

## Test conformance as of October 2024

<div class="twocol">
<div>

- SpiderMonkey: 95%
- GraalJS: 89%
- LibJS: 75%
- V8: 74%
- JavaScriptCore: 41%
- Boa: 37%

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': 4042,
    'GraalJS': 3785,
    'LibJS': 3192,
    'V8': 3147,
    'JSC': 1720,
    'Boa': 1582,
  };
  const totalTests = 4239;

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
npx test262-harness --hostType=sm --hostPath=$HOME/workspace/gecko/obj-debug-x86_64-pc-linux-gnu/dist/bin/js -f Temporal "test/**/*.js"
npx test262-harness --hostType=v8 --hostPath=$HOME/.esvu/bin/v8 -f Temporal --hostArgs=--harmony-temporal -- "test/**/*.js"  # requires https://github.com/bterlson/eshost/pull/139
LD_LIBRARY_PATH=$HOME/.local/lib64 npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/ladybird-js -f Temporal --hostArgs=--use-test262-global -- "test/**/*.js"
npx test262-harness --hostType=jsc --hostPath=$HOME/.esvu/bin/jsc -f Temporal --hostArgs=--useTemporal=1 -- "test/**/*.js"
cargo run --release --bin boa_tester -- run --test262-path $HOME/workspace/test262 -s ...
  (test/built-ins/Temporal, test/intl402/Temporal, test/staging/Temporal, test/staging/Intl402/Temporal, test/intl402/DateTimeFormat/**/*temporal*)
npx test262-harness --hostType=graaljs --hostPath=$HOME/.esvu/bin/graaljs -f Temporal --hostArgs='--experimental-options --js.temporal' -- "test/**/*.js"
npx test262-harness --hostType=node --hostPath=$HOME/.local/bin/deno -f Temporal --hostArgs='run --unstable-temporal' -- "test/**/*.js"
-->

---

<!-- _class: lead -->

# Questions?

Do you have existing plans for a timeline to ship Temporal in your implementation? If so, would you like to share now?

---

# Proposed summary for notes

> The proposal is as close to frozen as anything can be in Stage 3. Implementations should complete work on the proposal and ship it, and let the champions know ASAP if anything is blocking or complicating that. You are welcome to join the champions meetings.
