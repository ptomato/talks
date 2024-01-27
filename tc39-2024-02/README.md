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

- Four normative changes to propose today
  - (from implementors or usage experience in community)
- Follow the checklist in issue [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates
- **Will give a loud signal when this checklist is complete**

---

### weekOfYear/yearOfWeek optional (PR [#2756](https://github.com/tc39/proposal-temporal/pull/2756))

- Brought to our attention by community members
- Some calendars have **week numbers**, some don't
- Prior art: some calendars have eras, some don't
  - dates' `era` and `eraYear` readonly properties may be `undefined`
- **Proposal**:
  - allow dates' `weekOfYear` and `yearOfWeek` readonly properties to be `undefined`
  - allow calendars to return `undefined` from `weekOfYear()` and `yearOfWeek()` methods

---

### Note on specifying calendar numbering systems

- **Which** calendars have week numbers and how are they numbered?
- ECMAScript does **not** have to specify this
- Data will come from CLDR
- However, may be in scope of the Intl Era and Month Codes proposal
- See issue [tc39/proposal-intl-era-monthcode#15](https://github.com/tc39/proposal-intl-era-monthcode/issues/15) for discussion

---

<!-- _footer: "*Except for the default roundingMode, which has a reason we won't get into here" -->

### Duration rounding bug (PR [#2758](https://github.com/tc39/proposal-temporal/pull/2758))

- Brought to our attention by a Temporal polyfill author
- These should be equivalent\*:
```js
past.until(past.add(duration), { largestUnit, etc })
duration.round({ relativeTo: past, largestUnit, etc })
```
- Results disagree when `past` is a Temporal.ZonedDateTime and `past` + date part of duration is a day with DST
- Fix bug and also refactor the spec so these go through the same code path

---

### ZonedDateTime difference bug (PR [#2760](https://github.com/tc39/proposal-temporal/pull/2760))

- Brought to our attention by a Temporal polyfill author
- Surprising:
```js
const duration = Temporal.Duration.from({ months: 1, days: 15, hours: 12 });
const past = Temporal.ZonedDateTime.from('2024-02-10T02:00[America/New_York]');
const future = past.add(duration);
past.until(future, { largestUnit: 'months' })  // => 1 month, 15 days, 11 hours
```
- A shift due to DST is expected, but in this case DST is on 2024-03-10, not at the start or the end of the calculation
- Changes calculation of an intermediate ZDT value

---

### End-of-month edge cases (PR [#2759](https://github.com/tc39/proposal-temporal/pull/2759))

- Raised by André Bargull as part of the Firefox implementation
- Surprising results when adding and subtracting dates near end of month with non-default `largestUnit`:

```js
const end = new Temporal.PlainDate(1970, 2, 28);
new Temporal.PlainDate(1970, 1, 28).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 29).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 30).until(end, {largestUnit:"months"})  // "1 month"
new Temporal.PlainDate(1970, 1, 31).until(end, {largestUnit:"months"})  // "1 month"
```
- Not necessarily incorrect; results agree with Moment and Luxon
- Disagree with java.time (1 month, 30 days, 29 days, 28 days resp.)

---

### End-of-month edge cases

- Long discussion and some help from polyfill author Adam Shaw
- Tweak date difference algorithm to differentiate these cases where possible
- Changes results of 396 date-pairs out of all 2.1M possible date pairs in a 4-year Gregorian calendar cycle
- "_N_ months and 0 days" → "_N_ - 1 months and 28, 29, or 30 days"
- Results for default `largestUnit` remain the same

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
