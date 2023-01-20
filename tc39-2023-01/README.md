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

<!--
_class: invert lead
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 January-February 2023

---

# Temporal progress update

- Implementation continues
- Final push to resolve issues raised during stage 3
- A few normative PRs for consensus in this plenary

<!--
-->

---


# Final push

- What does "final push" mean?
- Goals:
  - All discussions on existing issues resolved
  - By March plenary, no remaining normative changes

---

# Final push (2)

- As of today's plenary, there remain no open discussions
- Two/three (TBD) normative PRs remain to be made
- (Assuming what we present today gets consensus)

<!--
-->

---

## IETF standardization progress [#1450](https://github.com/tc39/proposal-temporal/issues/1450)

- IESG review asked for a section on security concerns
- This section has been written and the ball is back in IESG's court
- Reminder; an official RFC number will remove the blocker from our side

<!--
-->

---

## Validation in mergeFields [#2466](https://github.com/tc39/proposal-temporal/issues/2466)

(summary to be added after getting clarity on this issue in next week's champions meeting)

---

## Unnecessary calls to user code [#2247](https://github.com/tc39/proposal-temporal/issues/2247) & [#2289](https://github.com/tc39/proposal-temporal/issues/2289)

- Implementation feedback has turned up several cases of calling redundantly into user code
- Until now, we've been fixing these case-by-case
- Best solved by auditing the whole proposal
- Work in progress, intend to present in March

---

## MakeDay/MakeDate/TimeFromYear out of range [#2315](https://github.com/tc39/proposal-temporal/issues/2315)

- [MakeDay](https://tc39.es/ecma262/#sec-makeday), [MakeDate](https://tc39.es/ecma262/#sec-makedate), and [TimeFromYear](https://tc39.es/ecma262/#eqn-TimeFromYear) can overflow to infinity
- This is even already a problem in ECMA-262 (see [ecma262#1087](https://github.com/tc39/ecma262/issues/1087))
- Proposed solution is for Temporal to use separate versions of these operations that calculate in ℝ
- Work in progress, intend to present in March

---

# When is it ready to ship unflagged?

- "Ready to ship" definition being discussed in this plenary
- Need the IETF to publish an RFC, so they can complete their review

<!--
-->

---

<!-- _class: lead -->

# Questions
### (about this section)

---

<!-- _class: invert lead -->

# Adjustments
to the proposal

---

### Optimizability of built-in calendars and time zones (1/5)

**Summary:** A large change that solves a long-standing request from implementations, making it easier to determine when it's OK to avoid creating a `Temporal.Calendar` or `Temporal.TimeZone` object and just use the built-in behaviour.

Pull request: [#2482](https://github.com/tc39/proposal-temporal/pull/2482)

---

### Optimizability (2/5)

```js
// Basically the problem is this:
const d = Temporal.PlainDate.from({ year: 2022, month: 11, day: 30, calendar: 'gregory' });
d.calendar  // ⇒ Property value is a Temporal.Calendar instance
// Every time some calendar calculation is done, e.g...
const d2 = d.add({ months: 1 });
// ...the calendar methods have to be looked up on that instance and called
// even though we just want the behaviour of the built-in calendar

// But on the other hand, in the case of a custom calendar object...
const d3 = d.withCalendar(myCustomCalendar);
const d4 = d3.add({ months: 1 });
// ... we do actually want those lookups to happen
```

---

### Optimizability (3/5)

Proposed solution (calendars: Plain types and ZonedDateTime)
- [[Calendar]] internal slot can store
  - a string (`"iso8601"`, `"gregory"`), for built-in behaviour that always calls intrinsics
  - an object, for custom behaviour with observable lookups & calls
- Replace `.calendar` getter with `.calendarId` getter, which returns the string directly or Gets `.id` on the object
- Add `.getCalendar()` method, which returns the object directly or creates a new Temporal.Calendar from the string

---

### Optimizability (4/5)

Proposed solution (time zones):
- ZonedDateTime [[TimeZone]] internal slot can store a string or object, just like [[Calendar]] in previous slide
- Replace `ZonedDateTime.prototype.timeZone` getter with `ZonedDateTime.prototype.timeZoneId` getter
- Add `ZonedDateTime.prototype.getTimeZone()` method

---

### Optimizability (5/5)

Proposed solution (incidentals):
- Rename `Temporal.Now.timeZone()` → `Temporal.Now.timeZoneId()`, always returns a string
- Remove calendars altogether from Temporal.PlainTime
- Remove read of `timeZone` on property bag passed to `ZonedDateTime.p.with()`
- Change time zone and calendar comparison semantics to use `.id`

---

### Values accepted by Calendar.from/TimeZone.from (PR [#2485](https://github.com/tc39/proposal-temporal/pull/2485))

- Following on from the change to builtin calendars and time zones
- Now worth revisiting that from() accepts two kinds of strings (identifiers and ISO strings) and three kinds of objects (Temporal objects, plain objects that are custom calendars, plain objects that are property bags)
- TBD: fill this slide after Thursday's champions meeting

---

### Remove fallback `fields` and `mergeFields` (PR [#2467](https://github.com/tc39/proposal-temporal/pull/2467))

- An earlier design had fallbacks for more Calendar methods
- If builtin calendars always call intrinsics, e.g. `delete Temporal.Calendar.prototype.fields` isn't a concern
- Guidance for custom calendars that don't extend `Temporal.Calendar` is to always implement all methods
- Won't affect the vast majority of code

---

### Extra calendar fields in ECMA-262 (PR [#2442](https://github.com/tc39/proposal-temporal/pull/2442))

- Previous plenary resolved that 402 may add properties to objects defined in 262
- This PR is the result of that
- Doesn't affect implementations

---

### Clarify meaning of `daysInMonth` (PR [#2484](https://github.com/tc39/proposal-temporal/pull/2484))

- Previously, `daysInMonth` had two meanings:
  - Count of days in the month
  - Number of the last day of the month
- Normally the same, but transition from Julian to Gregorian calendar skipped some calendar days
- This is not currently a CLDR calendar supported by `Intl`
- However, CLDR plans to add it in the future
- e.g. `Temporal.PlainYearMonth.from('1582-10').daysInMonth === 21`
- The distinction requires a change to PlainYearMonth arithmetic

---

### Clarify non-ISO-calendar methods (PRs [#2474](https://github.com/tc39/proposal-temporal/pull/2474) and [#2475](https://github.com/tc39/proposal-temporal/pull/2475))

- Address questions raised during implementation
- Definition of `mergeFields` for non-ISO-calendar is ambiguous
- Definitions of `yearMonthFromFields` and `monthDayFromFields` were missing a step

---

### Limit previously 'unlimited' rounding increments (PR [#2480](https://github.com/tc39/proposal-temporal/pull/2480))

- When rounding Durations, `years`, `months`, `weeks`, `days` fields were allowed to be rounded to any positive finite increment
- Limit requested by implementor due to storage concern
- Limit is now `1e9`

```js
Temporal.PlainDate.from('2023-01-05').until('2023-01-06', { roundingIncrement: 1e10 })
  // Before: no problem!
  // After: RangeError
```

---

### Consistency in user-observable operations (PR [#2478](https://github.com/tc39/proposal-temporal/pull/2478))

- General principle: Perform user-observable validation operations on the receiver before any arguments
- There were a few places this wasn't the case
  - `with()` methods
  - PlainYearMonth `add()`, `subtract()`
- Unlikely to make any difference unless specifically looking for it with `Proxy`

---

<!-- _class: invert lead -->

# Bugs

---

### Fix discrepancies in PrepareTemporalFields (PR [#2472](https://github.com/tc39/proposal-temporal/pull/2472))

- Frank pointed out that we don't need two versions of this AO
- In fact, accidental discrepancies had arisen between them
- Avoid a contrived situation where code works in an implementation without 402 and breaks in an implementation with 402:

```js
class C extends Temporal.Calendar {
    constructor() { super('iso8601'); }
    fields(l) { return [...l, 'era']; }
}
Temporal.PlainDate.from({ era: 'foo', year: 2023, month: 1, day: 30, calendar: new C() });
```

---

### Fix time zone formatting in ZonedDateTime (PR [#2479](https://github.com/tc39/proposal-temporal/pull/2479))

- ZonedDateTime's [[TimeZone]] slot unintentionally left unconsidered in locale-sensitive formatting

```js
Temporal.ZonedDateTime.from('2023-01-30T10[Antarctica/McMurdo]').toLocaleString('en')
  // Current spec text: '1/29/2023, 1:00:00 PM PST' (for me, that is)
  // Intended: '1/30/2023, 10:00:00 AM GMT+13'
```

---

### Fix regression in Instant arithmetic (PR [#2477](https://github.com/tc39/proposal-temporal/pull/2477))

- Unintentional side effect of a previous change
- Converted a mathematical value to a Number value too early, causing inexact result

```js
const earlier = new Temporal.ZonedDateTime(1546935756_123_456_789n, "UTC");
const later = new Temporal.ZonedDateTime(1631018380_987_654_289n, "UTC");
later.since(earlier, {smallestUnit: "nanoseconds"}).nanoseconds
  // Current spec text: 504
  // Intended: 500
```

---

### Stricter validation of Calendar/TimeZone method return values (PR [#2456](https://github.com/tc39/proposal-temporal/pull/2456))

- Return value always validated when calling into user code
- Inconsistency: sometimes throw on wrong type, sometimes convert
- Choice is to consistently throw on wrong type
- This is no longer accepted:

```js
class C extends Temporal.Calendar {
    constructor() { super("iso8601"); }
    daysInWeek() { return "7"; }
}
```

---

### "Normative typo" in PlainMonthDay (PR [#2460](https://github.com/tc39/proposal-temporal/pull/2460))

- Wrong variable used in wrong place caused calendar calculation with incorrect year

```js
Temporal.PlainMonthDay.from('2023-01-05[u-ca=hebrew]')
  // Current spec text: PlainMonthDay of 18 Tevet
  // Intended: PlainMonthDay of 12 Tevet
  // (ISO date 5 January 2023 is 12 Tevet 5783 in the Hebrew calendar)
```

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
