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
TC39 February 6–8, 2024

---

## Progress update

- Backlog of approved normative changes is all caught up
  - Will merge after this meeting
- 4 normative changes proposed today for bugs found during implementation, polyfilling, or community use
  - Narrowly scoped: edge cases with minimal implementer impact
- NOT an indication of more changes coming
  - No more known outstanding bugs
  - Last normative changes were in Nov 2023 and July 2023
  - Next one... never?

---

## Progress update (2)

- Next: finalize any editorial changes that implementors might find disruptive
- **Will give a loud signal when this is done**
- Expect this **soon**
- Follow the checklist in [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates

---

### weekOfYear/yearOfWeek optional (PR [#2756](https://github.com/tc39/proposal-temporal/pull/2756))

- Reported by early adopters
- Some calendars have **week numbers**, some don't
- Prior art: some calendars have eras, some don't
  - Dates' `era` and `eraYear` readonly properties may be `undefined`
- **Proposal**
  - Allow `weekOfYear` and `yearOfWeek` readonly properties to be `undefined`
  - Calendars may return `undefined` from `weekOfYear()` and `yearOfWeek()` methods

---

### Notes for specifying calendars

- **Which** calendars have week numbers and how are they numbered?
- ECMAScript does **not** have to specify this
- Data will come from CLDR and ICU
- However, may be in scope of the Intl Era and Month Codes proposal
- See [tc39/proposal-intl-era-monthcode#15](https://github.com/tc39/proposal-intl-era-monthcode/issues/15)

---

### Duration rounding bug (PR [#2758](https://github.com/tc39/proposal-temporal/pull/2758))

- Reported by Adam Shaw (polyfill author)
```js
const duration = Temporal.Duration.from({years: 1, hours: 24});
const past = Temporal.ZonedDateTime.from("2019-11-01T00:00-07:00[America/Vancouver]");
const future = past.add(duration);  // 2020-11-01T23:00-08:00[America/Vancouver]
  // (note, not 2020-11-02, because 2020-11-01 is a 25-hour day in that time zone)

past.until(future, {largestUnit: 'years'})  // 1 year, 24 hours (CORRECT)
duration.round({largestUnit: 'years', relativeTo: past})  // 1 year, 1 day (WRONG)
```
- In `round`, the wrong reference date was used

---

### Duration rounding bug (2)

- Generalizing, these should _always_ be equivalent:
```js
past.until(past.add(duration), { largestUnit, smallestUnit, roundingMode, etc })
duration.round({ relativeTo: past, largestUnit, smallestUnit, roundingMode, etc })
```
- Today, results disagree if `past` is a Temporal.ZonedDateTime and a DST change happens on `past` + date part of duration
- Fix bug + refactor spec to unify codepath for both operations
- Prevents future discrepancies

---

### ZonedDateTime difference bug (PR [#2760](https://github.com/tc39/proposal-temporal/pull/2760))

- Reported by Adam Shaw (polyfill author)
```js
const duration = Temporal.Duration.from({ months: 1, days: 15, hours: 12 });
const past = Temporal.ZonedDateTime.from('2024-02-10T02:00[America/New_York]');
const future = past.add(duration);
past.until(future, { largestUnit: 'months' })  // => 1 month, 15 days, 11 (!) hours
```
- Above: DST disambiguation happened at the month/day boundary, but should have been ignored
- DST disambiguation only expected at the day/hour boundary
- Narrowly-scoped fix to ZDT addition algorithm

---

### End-of-month edge cases (PR [#2759](https://github.com/tc39/proposal-temporal/pull/2759))

- From André Bargull (Firefox impl.) and Adam Shaw (polyfill author)
- Unexpected `since`/`until` result near end of month

```js
const end = new Temporal.PlainDate(1970, 2, 28);
new Temporal.PlainDate(1970, 1, 28).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 29).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 30).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 31).until(end, {largestUnit:"months"})  // "1 month"
```
- Expected (e.g. java.time): 1 month, 30 days, 29 days, 28 days resp. 
- Affects `largestUnit: 'months'|'years'` (default options are OK)
- Narrow scope: one-line change affects only 0.2% of date pairs

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

On normative PRs [#2756](https://github.com/tc39/proposal-temporal/pull/2756), [#2758](https://github.com/tc39/proposal-temporal/pull/2758), [#2759](https://github.com/tc39/proposal-temporal/pull/2759), [#2760](https://github.com/tc39/proposal-temporal/pull/2760)

---

# Proposed summary for notes

> A normative change making week numbering optional for calendars (PR [#2756](https://github.com/tc39/proposal-temporal/pull/2756)) reached consensus.
> A normative change to fix a bug in duration rounding (PR [#2758](https://github.com/tc39/proposal-temporal/pull/2758)) reached consensus.
> A normative change to return more useful results from date differences in end-of-month edge cases ([#2759](https://github.com/tc39/proposal-temporal/pull/2759)) reached consensus.
> A normative change to fix a bug in ZonedDateTime differences (PR [#2760](https://github.com/tc39/proposal-temporal/pull/2760)) reached consensus.

---
