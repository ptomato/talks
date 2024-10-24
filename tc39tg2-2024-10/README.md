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

# Guidelines for Locale-Sensitive Testing in Test262

**Philip Chimento**
Igalia
TC39 TG2 meeting, 24 October 2024

---

## Test262

The conformance test suite for ECMA-262 and ECMA-402.

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
- What kind of guidelines would be helpful for test writers in this group?
- More info: https://github.com/tc39/test262/issues/3786

