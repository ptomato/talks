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

# ⌚ **Temporal and TG2**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39-TG2 February 2022

---

## Refresher: Temporal

- Stage 3 TC39 proposal
- Introduces 7 data types for representing dates and times
- As well, `Temporal.Duration` for representing lengths of time
- `Temporal.TimeZone` and `Temporal.Calendar` allow calculations with time zones and calendars

---

## Refresher: Temporal (2)

- `Temporal.Instant`: exact time, no time zone or calendar
- `Temporal.ZonedDateTime`: exact time, with time zone and calendar
- `Temporal.PlainDateTime`: wall-clock time, with calendar
- `Temporal.PlainTime`, `Temporal.PlainDate`, `Temporal.PlainYearMonth`, `Temporal.PlainMonthDay`: wall-clock time / wall-calendar date, with less information

---

## How Temporal interacts with TG2

- An [annex](https://tc39.es/proposal-temporal#sec-temporal-intl) to Temporal lists all modifications that would be made to Ecma-402
- Some open questions that require a decision from TG2

<!--
    I'll discuss both of these in this short presentation.
    It was brought to my attention that not everyone who regularly attends these meetings is aware of the Ecma-402 annex, so I'll start there because I primarily want everyone to know about it.
-->

---

<!-- _class: invert -->

# Ecma-402 annex to Temporal proposal

- https://tc39.es/proposal-temporal#sec-temporal-intl
- Mostly concerns Intl.DateTimeFormat and calendar support

---

## Time Zone Names

Disregard this section; it's going to be [removed](https://github.com/tc39/proposal-temporal/issues/1996).

(Originally the idea was to take IsValidTimeZoneName, CanonicalizeTimeZoneName, and DefaultTimeZone into Ecma-262.)

---

## GetOptionsObject

This abstract operation is proposed to be moved into Ecma-262, so there's no need to keep a separate copy in Ecma-402.

---

## DateTimeFormat

Temporal requires having a different "pattern" for each Temporal type, instead of one pattern for formatting `Date`, so we adapt `Intl.DateTimeFormat` accordingly.

(There is an issue open about storage requirements for `Intl.DateTimeFormat`: [#602](https://github.com/tc39/ecma402/issues/623))

---

## Calendar support

- Without Ecma-402, Temporal only knows the ISO 8601 calendar, specified exactly
- Add `era()` and `eraYear()` methods to `Temporal.Calendar`
- Add `era` and `eraYear` properties to `Plain` types
- Supersede the ISO calendar operations defined in Ecma-262

---

## Calendar support (2)

- Non-ISO calendar operations are specified loosely
- Just how loosely is still a matter of debate
- Pull request: https://github.com/tc39/proposal-temporal/pull/1928

---

<!-- _class: invert -->

# Open questions

These may or may not require discussion / action from TG2.

---

## How to handle out-of-bounds `era`/`eraYear`

- TG2 issue: [#540](https://github.com/tc39/ecma402/issues/540)
- Rough agreement in GitHub issue

---

## Formatting ZonedDateTime's time zone

- Temporal issue: [#2013](https://github.com/tc39/proposal-temporal/issues/2013)
- FormatDateTimePattern will have to gain a time zone argument, so that the ZonedDateTime's time zone overrides the time zone in the [[TimeZone]] internal slot of the `Intl.DateTimeFormat` object

---

### Property-bag zones & calendars in 402

<small>

- Temporal issue: [#2005](https://github.com/tc39/proposal-temporal/issues/2005)
- The [[TimeZone]] and [[Calendar]] internal slots of DateTimeFormat should be Temporal instances, not strings
- Temporal also accepts property-bag time zones and calendars that implement the protocol

</small>

```js
const timeZone = {
    getOffsetNanosecondsFor(instant) { ... },
    getPossibleInstantsFor(dateTime) { ... },
    toString() { return "Etc/My_Zone"; }
};
const dateTime = Temporal.Now.zonedDateTimeISO(timeZone);
dateTime.toLocaleString();  // Should this code succeed or fail? What should be the result?
```

---

# Calendar-specific questions

These will likely need user research from TG2.

- What's the behaviour of date arithmetic around epagomenal days? [p-t#1994](https://github.com/tc39/proposal-temporal/issues/1994)
- What should be the anchor year for the Ethiopic calendar? [#534](https://github.com/tc39/ecma402/issues/534)
- What era should be used for the Hebrew calendar? [#535](https://github.com/tc39/ecma402/issues/535)
- Designing unique identifiers for eras [#541](https://github.com/tc39/ecma402/issues/541)

---

<!-- _class: invert lead -->

# Thanks!
