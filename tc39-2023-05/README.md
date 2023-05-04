---
theme: gaia
paginate: true
footer: DRAFT
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
- Handling bug reports from implementations, polyfill writers, users
- Work continues on integer arithmetic changes discussed in March

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

---

## Fix rounding bug (PR [#2571](https://github.com/tc39/proposal-temporal/pull/2571))

- Bug reported by Adam Shaw (a polyfill author)

---

## >1 calendar annotation (PR [#2572](https://github.com/tc39/proposal-temporal/pull/2572))

- Required due to a clarification from the IETF on how to interpret multiple calendar annotations
- Also incorporates a suggestion from Anba (Firefox) for better optimizability

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
