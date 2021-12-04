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
  pre code { background-color: #042029; }
  .hljs-string { color: #8ae234; }
  .hljs-number, .hljs-literal { color: #729fcf; }
  .hljs-params { color: #e9b96e; font-style: italic; }
  .hljs-built_in { color: #fce94f; font-weight: bold; }
  .hljs-keyword { color: #fcaf3e; font-weight: bold; }
  .hljs-attr { color: #e9b96e; }
  .hljs-variable { color: red; font-weight: bold; }
  /* .hljs-comment, .hljs-regexp, .hljs-symbol */
---

<!-- _class: invert lead -->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 December 2021

---

# Temporal update

- Similar to what we asked for consensus on last time!
- Asking for consensus on several minor normative changes:
  - Changes suggested by implementors ("adjustments")
  - Changes to make spec text accurately reflect the intentions of the champions ("bugs")
- No discussions this time, just fixes

---

# Temporal update

- Will continue turning implementor feedback into presentations like this as bandwidth allows
- Number of normative PRs per plenary meeting seems to be decreasing 📉

---

<!-- _class: invert lead -->

# Adjustments

---

### ISO 8601 time-only representation ([#1952](https://github.com/tc39/proposal-temporal/pull/1952))

- Time-only strings may be prefixed with a `T` designator; Temporal did not yet conform to this
- The `T` is _required_ where ambiguity with date representations is possible
  - Originally did not support this because `Temporal.PlainTime.from()` was not considered ambiguous
  - Since then, we moved towards accepting strings in other places where ambiguity may occur

---

### ISO 8601 time-only representation (cont'd)

Examples of ambiguous strings:

- **2021-12**: year-month, or time with UTC offset **20:21:00-12:00**?
- **202112**: year-month, or time **20:21:12**?
  - Times now require **T2021-12** and **T202112**
- **12-14**: month-day, or time with UTC offset **12:00:00-14:00**?
- **1214**: month-day, or time **12:14:00**?
  - Times now require **T12-14** and **T1214**
- **2021-12-14**: date, or date-time with time defaulting to 00:00?
  - If you want "midnight" from a date-only string, parse as PDT

<!--
    Note that a lot of the ambiguities arise because ISO 8601 allows omitting the colon in a time representation (this is called "basic format" whereas the format with the colon is called "extended format")
-->

---

### More conformant annotation validation ([#1953](https://github.com/tc39/proposal-temporal/pull/1953))

- Clearer distinction of what is invalid ISO 8601 syntax and what isn't
  - e.g. `2021-13-32T99:99:99` — syntactically invalid
  - `2021-12-14T10:00[Mars/Olympus_Mons]` - syntactically valid, but time zone not known to ECMA-402

```js
Temporal.ZonedDateTime.from('2021-12-14T10:00[Mars/Olympus_Mons]')
  // throws (no change)
Temporal.Instant.from('2021-12-14T10:00Z[Mars/Olympus_Mons]')
  // Before: throws
  // After: same result as '2021-12-14T10:00Z'; time zone annotation ignored
```

---

<!-- _class: invert lead -->

# Bugs

---

### Brand checks in TimeZone methods ([#1693](https://github.com/tc39/proposal-temporal/pull/1693))

- Some TimeZone methods were missing brand checks
- Left over from how these methods worked at one time
```js
const timeZone = Object.create(Temporal.TimeZone.prototype, {
  getPossibleInstantsFor() { ... },
  getOffsetNanosecondsFor() { ... },
  toString() { return "my own UTC variant" },
});
timeZone.toJSON() === "my own UTC variant"
  // Intended: throw due to timeZone not having correct internal slot
  // According to current spec text: true
```
<span style="font-size:24px">Note: no change if you inherit TimeZone. This only affects objects that don't have the internal slots</span>

---

### Wrong rounding for fractional Duration strings ([#1907](https://github.com/tc39/proposal-temporal/issues/1754))

- Was floor, should have been trunc
```js
Temporal.Duration.from('PT1.03125H').toString()
  // "PT1H1M52.5S" (no change)
Temporal.Duration.from('-PT1.03125H').toString()
  // Intended:                       "-PT1H1M52.5S"
  // According to current spec text: "-PT1H2M53.5S"
```

---

### Missing calendar annotations in ISO strings ([#1950](https://github.com/tc39/proposal-temporal/pull/1950))

- Some calendar annotations forgotten in the ISO 8601 grammar
- PlainTime strings like `15:13:45[u-ca=iso8601]` should be allowed
  - Necessary if we ever want to include time calendars in a future proposal without breaking the web
- PlainYearMonth and PlainMonthDay strings like `2021-12-14[u-ca=iso8601]` should be allowed
  - Necessary for round-tripping via toString()

---

### Consistent units handling in PlainDate ([#1955](https://github.com/tc39/proposal-temporal/pull/1955))

- In `since()` and `until()` methods, value for omitted `largestUnit` should adapt if `smallestUnit` is given
- Omitted line in the spec text made this not happen for PlainDate
- Make PlainDate consistent with other types

```js
date1 = Temporal.PlainDate.from('1970-01-01');
date2 = Temporal.Now.plainDateISO()
date1.until(date2, { smallestUnit: 'year' })
  // Intended: Duration of { years: 51 }
  // According to current spec text: throws RangeError
  // (because default largestUnit < smallestUnit)
```

---

### Make fractionalSecondDigits consistent ([#1956](https://github.com/tc39/proposal-temporal/pull/1956))

- In `Intl` methods, `fractionalSecondDigits` always displays no more and no less than that number of digits
- Make Duration.toString consistent with this, as intended

```js
duration = Temporal.Duration.from({ years: 3 });
duration.toString({ fractionalSecondDigits: 2 });
  // Intended: "P3YT0.00S"
  // According to current spec text: "P3Y"
```

---

### Mistake in grammar of ISO 8601 strings ([#1957](https://github.com/tc39/proposal-temporal/pull/1957))

- When parsing `2020-12-14T07:45:24.12345-00:00:00.321[+11:11:11.54321]`, _TimeFractionalPart_ could refer either to `12345` or `54321` according to the ISO 8601 grammar in the proposal
- Disambiguate this by referring to separate productions

---

### Modulo vs. remainder, round 3 ([#1947](https://github.com/tc39/proposal-temporal/pull/1947))

- I thought for sure we had eliminated all of this type of bug, but an implementor found another one!
- Affects creating Instant and ZonedDateTime from negative epoch nanoseconds
- Was originally written correctly, then I "fixed" it 😬

---

### Typos that were normative 😱

- Fix algorithms that don't work as described in the current spec text due to typos
- List of pull requests:
  - [#1926](https://github.com/tc39/proposal-temporal/pull/1926)
  - [#1931](https://github.com/tc39/proposal-temporal/pull/1931)
  - [#1946](https://github.com/tc39/proposal-temporal/pull/1946)

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented

---

<!-- _class: invert lead -->

# Thanks!
