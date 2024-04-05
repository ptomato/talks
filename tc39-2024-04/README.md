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
---

<!-- _class: invert lead -->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 April 2024

---

## Progress update

- 1 normative change proposed today for bug caught by a polyfill implementor
- A few other bugs found in Duration arithmetic, to be addressed in June plenary
- We hear continuing concerns from implementations and will work with them to address these before June plenary

---

## Bug in rounding edge cases (PR [#2797](https://github.com/tc39/proposal-temporal/pull/2797))

When rounding a ZonedDateTime to the day boundary, it was possible in rare cases due to DST to add a spurious extra day.

```js
const zdt = Temporal.ZonedDateTime.from('2024-03-10T23:00:01[America/New_York]')
zdt.round({ smallestUnit: 'day', roundingMode: 'ceil' })
  // Current: 2024-03-12T00:00:00-04:00[America/New_York] (incorrect)
  // Proposed: 2024-03-11T00:00:00-04:00[America/New_York] (correct)
```
The incorrect result comes from not taking into account that 23:00:01 is 22h 1s into a 23-hour day. (March 10th was DST "spring forward".)

(Discovered by Adam Shaw, a polyfill implementor)

---

## Implementor concerns

- Concerns raised about the size of Temporal
- Discussed in hallway during February plenary
- V8: Concerns about compiled binary size on 32-bit Android ([#2786](https://github.com/tc39/proposal-temporal/issues/2786))
  - We made a proof of concept showing how to cut binary size up to 38% with no change in functionality
- JavaScriptCore: General concerns about standard library growth, but not specifically about Temporal
- (cont'd)

---

## Implementor concerns (2)

- SpiderMonkey: Concerns about Firefox installer size
- V8: Concerns about complexity of proposal
- Polyfill: Newly discovered bugs in duration arithmetic
- Considering dropping or reducing:
  - User-defined calendars and time zones, associated subclassing?
  - `relativeTo` parameter in Duration.p.add/subtract?
- Open to suggestions. It helps us if concerns can be made specific

---

## Implementor concerns (3)

We would like to make sure doubts are addressed before the June 2024 plenary, so there are no remaining obstacles to implementation.

If you have opinions or would like to discuss, please drop in on the Temporal champions meeting, biweekly Thursdays at `08:00[America/Los_Angeles]`!

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

On normative PR [#2797](https://github.com/tc39/proposal-temporal/pull/2797)

---

# Proposed summary for notes

> Consensus was reached on a normative change to fix a bug in rounding that occurred in rare cases having to do with DST.
> Over the next few weeks, we plan to dig into remaining concerns from TC39 delegates.
> Follow the checklist in [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates.

---
