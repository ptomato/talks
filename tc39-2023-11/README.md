---
theme: gaia
paginate: true
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.14.0/css/all.min.css');
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
  pre.mermaid { background: transparent; }
  pre.mermaid svg .background { fill: transparent; }
---

<!-- _class: invert lead -->

<!-- mermaid.js -->
<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
mermaid.initialize({startOnLoad:true});
</script>

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 November 2023

---

## Progress update

- Most approved normative changes merged with test262 coverage
- Working through some remaining review comments from Anba
- One normative change to propose today
  - (from usage experience in community)
- Follow the checklist in issue [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates
- **Will give a loud signal when this checklist is complete**

---

## Test conformance as of October 2023

<div class="twocol">
<div>

- SpiderMonkey: 92%
- V8: 74%
- JavaScriptCore: 31%
- LibJS: 24%
- Boa: 13%

</div>
<div>
<pre class="mermaid">
%%{
  init: {
    "xyChart": {
      "xAxis": {
        "labelPadding": 10
      },
      "yAxis": {
        "labelPadding": 10
      }
    },
    "themeVariables": {
      "fontFamily": "Rubik",
      "xyChart": {
        "plotColorPalette": "#a40000"
      }
    }
  }
}%%
xychart-beta horizontal
  x-axis [SM, V8, JSC, LibJS, Boa]
  y-axis "% of test262 passing" 0 --> 100
  bar [92, 74, 31, 24, 13]
</pre>
</div>
</div>

---

## IETF standardization progress ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))

- **Complete.** This is no longer a blocker for shipping unflagged.

---

## Leap days in PYM/PMD.toPlainDate ([#2718](https://github.com/tc39/proposal-temporal/pull/2718))


```js
// What day of the week is your birthday in 2030?
const bd = Temporal.PlainMonthDay.from("12-15");
bd.toPlainDate({ year: 2030 }).dayOfWeek;
```

- Current: throws an exception if your birthday is February 29th
- Design principle: "no data-driven exceptions"
- Instead, `toPlainDate()` should return 2030-02-28
- Can still get the throwing behaviour more verbosely:

```js
const { monthCode, day } = Temporal.PlainMonthDay.from("02-29");
Temporal.PlainDate.from({ year: 2030, monthCode, day }, { overflow: "reject" }).dayOfWeek;
```

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

On normative PR [#2718](https://github.com/tc39/proposal-temporal/pull/2718)

---

# Proposed summary for notes

> The blocker on IETF standardization of the string format has been resolved.
> The champions will give a signal when outstanding changes have been merged, and at that point implementations will be encouraged to continue their work and ship unflagged when ready.
> A normative change to overflow behaviour in PlainYearMonth/PlainMonthDay.p.toPlainDate (PR [#2718](https://github.com/tc39/proposal-temporal/pull/2718)) reached consensus.
