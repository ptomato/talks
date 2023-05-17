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
---

<!-- _class: invert lead -->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 May 2023

---

## Progress update

- Implementation continues; large Firefox patchset under review
- Only one change from March is outstanding: integer arithmetic optimization
- No other big changes expected
    - Issues reported since March have been minor 

---

## IETF standardization progress ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))

- Steering group review prompted one change that affects us
    - (More details in the second section of the presentation)
- Review continues

---

## Integer arithmetic progress

- Solution framework remains largely the same as discussed in March
- A few changes necessary:
  - Upper bound placed on the total of _days_...nanoseconds (instead of hours...nanoseconds)
  - (Possibly) bound years, months, and weeks individually

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: invert lead -->

# Normative changes

---

## Deduplication of field names (PR [#2570](https://github.com/tc39/proposal-temporal/pull/2570))

- Request from Anba (Firefox) for a small optimizability improvement
- ~~Removes~~ _throws on_ duplicate property names in PrepareTemporalFields
- _Also throws on property names `constructor` and `__proto__`_
- Unlikely edge case, unless intentional with a custom calendar

```js
const calendar = new class extends Temporal.Calendar {
  fields(fieldNames) { return super.fields(fieldNames).concat(['monthCode', 'monthCode']); }
}('iso8601');

const ym = new Temporal.PlainYearMonth(2023, 3, calendar);
ym.toPlainDate({day: 1});
// Current spec: monthCode property was accessed thrice.
// After this change: throws a RangeError.
```
---

## Fix rounding bug (PR [#2571](https://github.com/tc39/proposal-temporal/pull/2571))

- Bug reported by Adam Shaw (a polyfill author)
- In cases where a number is rounded up to next unit, current behaviour is inconsistent with `Duration.prototype.round()`
- Fixes the result after rounding in case of `until()/since()/toString()`


```js
const earlier = new Temporal.PlainDate(2022, 1, 1);
const later = new Temporal.PlainDate(2023, 12, 25);
const duration = earlier.until(later, { largestUnit: "years", smallestUnit: "months", roundingMode: "expand" });
console.log(duration.toString());
// Current spec text: 1Y12M 
// After this change: 2Y
```
---

## >1 calendar annotation (PR [#2572](https://github.com/tc39/proposal-temporal/pull/2572))

- Required due to a clarification from the IETF on how to interpret multiple calendar annotations
- Also incorporates a suggestion from Anba (Firefox) for better optimizability in YYYY-MM and MM-DD strings

```js
Temporal.PlainDate.from("2000-05-02T15:23[u-ca=iso8601][!u-ca=gregory]");
// Current spec text: Created a PlainDate object with ISO 8601 calendar
// After this change: Throws a RangeError

Temporal.PlainYearMonth.from("2000-05[u-ca=iso8601][u-ca=gregory]");
// Current spec text: Throws a RangeError
// After this change: Creates a PlainYearMonth object with ISO 8601 calendar
```
---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
