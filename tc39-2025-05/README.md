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
TC39 May 2025

---

## Progress update

Temporal is shipping in Firefox 139 (May 27th). V8 has started their integration of the Rust library `temporal_rs` and now has a partial implementation of Temporal.Instant. Work progresses on the JavaScriptCore implementation.

We have one normative change to propose, based on user feedback.

---

## Test conformance as of May 2025

<div class="twocol">
<div>

| Engine   | %PASS | Change |
| -------- | ----- | ------ |
| SM       | 99%   | ↓0.3%  |
| Ladybird | 96%   | ↓0.06% |
| GraalJS  | 90%   | ↓0.2%  |
| Boa      | 90%   | ↑5%    |
| JSC      | 41%   | ↑0.8%  |
| V8       | 18%   | ↓55%   |

</div>
<div>
  <canvas id="conformance-chart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const ctx = document.getElementById('conformance-chart');

  const results = {
    'SM': 9123,
    'Ladybird': 8831,
    'GraalJS': 8292,
    'Boa': 8257,
    'JSC': 3727,
    'V8': 1694,
  };
  const totalTests = 9171;
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

## Change to UTC offset matching (PR [#3107](https://github.com/tc39/proposal-temporal/pull/3107))

- UTC offset string part:
  "2025-05-05T12:47:43.054654178&zwj;**-04:00**&zwj;[America/New_York]"
- Support for seconds and below ("-04:00:00.0") is _not_ in RFC 3339
- We accept this (as it occurs in the wild) but do not emit it
- Discussed in [October 2021 TC39](https://ptomato.name/talks/tc39-2021-10/#5)

---

## Change to UTC offset matching (2)

- Some historical UTC offsets are not whole minutes, even in modern (post-1970) times
- October 2021: For UTC-00:44:30 we accept both `-00:44:30` and `-00:45`
- We do not accept `-00:44:40` or `-00:45:01`
- New feedback from Temporal users: we should _not_ accept `-00:45:00` — if the optional parts are _present_, they _must_ match

---

## Change to UTC offset matching (3)

- The Pacific island of Niue changed the UTC offset by 20 seconds in 1952 from UTC-11:19:40 to UTC-11:20
- Both offsets are valid at, e.g., 1952-10-15 23:59:59
- Previously, impossible to deserialize!
- `-11:20:00` would be treated as `-11:20` and match the first candidate, `-11:19:40`
- With this change, `-11:20:00` and `-11:19:40` work as expected
- `-11:20` still matches the first candidate

---

## Change to UTC offset matching (4)

Examples of change:

```js
Temporal.ZonedDateTime.from('1970-06-01T00:00:00-00:45[Africa/Monrovia]')
  // accepted; no change
Temporal.ZonedDateTime.from('1970-06-01T00:00:00-00:44:30[Africa/Monrovia]')
  // accepted; no change
Temporal.ZonedDateTime.from('1970-06-01T00:00:00-00:44:40[Africa/Monrovia]')
  // throws RangeError; no change
Temporal.ZonedDateTime.from('1970-06-01T00:00:00-00:45:00[Africa/Monrovia]')
  // Before: accepted
  // After: throws RangeError
Temporal.ZonedDateTime.from('1952-10-15T23:59:59-11:20:00[Pacific/Niue]').epochNanoseconds
  // Before: -543069621000000000n (matches candidate offset -11:19:40)
  // After:  -543069601000000000n (matches candidate offset -11:20:00)
```

---

<!-- _class: lead -->

## Questions?

---

<!-- _class: lead -->

## Requesting consensus
### Change the UTC offset matching as described

---

# Proposed summary for notes

Work on implementation in the larger and smaller JS engines continues.

A change to tighten the requirements on an optional string notation for unusual UTC offsets reached consensus. This change was based on user feedback.
