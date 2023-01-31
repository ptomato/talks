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
  - After, new issues not considered unless instances of the spec not working

---

# Final push (2)

- All known discussions on normative issues resolved
- Most of those PRs presenting today with 2 to 3 PRs expected in March plenary
- After March, the champions plan to pause work (unless implementers report more issues) until two implementations are complete and we can ask for Stage 4

---

<!-- _class: lead -->

# <span style="color:white;">Nearly done!</span>

![bg](finish.png)

<!--
https://www.nicepng.com/downpng/u2t4r5t4w7y3i1q8_finish-marathon-finish-line-clipart/
-->

---

## IETF standardization progress [#1450](https://github.com/tc39/proposal-temporal/issues/1450)

- IESG review asked for a section on security concerns
- This section has been written and the ball is back in IESG's court
- Reminder: an official RFC number will remove the blocker from our side

<!--
-->

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
- Need the IETF to publish the RFC in any case

<!--
-->

---

<!-- _class: lead -->

# Questions
### (about this section)

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
    fields(list) { return [...list, 'era']; }
}
Temporal.PlainDate.from({ era: 'foo', year: 2023, month: 1, day: 30, calendar: new C("iso8601") });
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
later.since(earlier, { smallestUnit: "nanoseconds" }).nanoseconds
  // Current spec text: 504
  // Intended: 500
```

---

### Stricter validation of Calendar/TimeZone method return values (PR [#2456](https://github.com/tc39/proposal-temporal/pull/2456))

- Return value always validated when calling into user code
- Inconsistent: sometimes throw on wrong type, sometimes convert
- Choice is to consistently throw on wrong type
- This is no longer accepted:

```js
class C extends Temporal.Calendar {
    daysInWeek() { return "7"; }
}
Temporal.Now.plainDate(new C("iso8601")).daysInWeek;
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

<!-- _class: invert lead -->

# Adjustments
to the proposal

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

### Clarify meaning of `daysInMonth` (PR [#2484](https://github.com/tc39/proposal-temporal/pull/2484))

- Previously, `daysInMonth` had two meanings:
  - **Count** of days in the month (we've chosen this one)
  - **1-based index** of the last day of the month
- Normally the same, but transition from Julian to Gregorian calendar skipped some calendar days
- This is not currently a CLDR calendar supported by `Intl`
- However, CLDR plans to add it in the future
- The distinction requires a change to PlainYearMonth arithmetic
```js
julianGregorian.yearMonthFromFields({ year: 1582, month: 10 }).daysInMonth === 21
```

---

### Clarify non-ISO-calendar methods (PRs [#2474](https://github.com/tc39/proposal-temporal/pull/2474) and [#2475](https://github.com/tc39/proposal-temporal/pull/2475))

- Address questions raised during implementation
- Definition of `mergeFields` for non-ISO-calendar is ambiguous
- Definitions of `yearMonthFromFields` and `monthDayFromFields` were missing a step

---

### Remove fallback `fields` and `mergeFields` (PR [#2467](https://github.com/tc39/proposal-temporal/pull/2467))

- An earlier design had fallbacks for more Calendar methods
- If builtin calendars always call intrinsics (see later slide), e.g. `delete Temporal.Calendar.prototype.fields` isn't a concern
- Guidance for custom calendars that don't extend `Temporal.Calendar` is to always implement all methods
  - <small>(All methods required anyway in the builtins-always-call-intrinsics world)</small>
- Won't affect the vast majority of code

---

### Extra calendar fields in ECMA-262 (PR [#2442](https://github.com/tc39/proposal-temporal/pull/2442))

- Previous plenary resolved that 402 may add properties to objects defined in 262
- This PR is the result of that
- Doesn't affect implementations

---

### Consistency in user-observable operations (PR [#2478](https://github.com/tc39/proposal-temporal/pull/2478))

- General principle: Perform user-observable validation operations on the receiver before any arguments
- There were a few places this wasn't the case
  - `with()` methods
  - PlainYearMonth `add()`, `subtract()`
- Unlikely to make any difference unless specifically looking for it with `Proxy`

---

### Optimizability of built-in calendars and time zones

#### Summary

- Largest remaining change expected
- Addresses request from implementers to avoid creating `Temporal.Calendar` or `Temporal.TimeZone` objects...
  - ...when Temporal objects use only built-in timezones/calendars ([#1808](https://github.com/tc39/proposal-temporal/issues/1808))
  - ...when constructing `Temporal.PlainTime` objects ([#1588](https://github.com/tc39/proposal-temporal/issues/1588))

Pull request: [#2482](https://github.com/tc39/proposal-temporal/pull/2482)

---

### Optimizability

#### Problem statement

```js
const d = Temporal.PlainDate.from({ year: 2022, month: 11, day: 30, calendar: 'gregory' });
d.calendar  // ⇒ a Temporal.Calendar instance

// Many Temporal operations...
const d2 = d.add({ months: 1 });
// ...require a Calendar instance and calling (perhaps patched) observable methods
// even though we just want built-in calendar behaviour.

// But in the case of a custom calendar object...
const d3 = d.withCalendar(myCustomCalendar);
const d4 = d3.add({ months: 1 });
// ... we do actually want those observable calls to happen
```

---

### Optimizability

#### Proposed (1/3): Types w/ calendars (`Plain*`, `ZonedDateTime`)

- [[Calendar]] internal slot stores either:
  - string (e.g. `"iso8601"`, `"gregory"`): for built-in behaviour that only calls intrinsics
  - object: for custom behaviour with observable lookups & calls
- Replace `.calendar` getter with `.calendarId` getter
  - Returns the string slot value, or Gets `.id` on the slot object
- Add `.getCalendar()` method
  - Returns the object slot value, or creates new `Temporal.Calendar`

---

### Optimizability

#### Proposed (2/3): Time zones in `ZonedDateTime`

- ZonedDateTime [[TimeZone]] internal slot can store a string or object, just like [[Calendar]] in previous slide
- Replace `ZonedDateTime.p.timeZone` getter with `ZonedDateTime.p.timeZoneId` getter
- Add `ZonedDateTime.p.getTimeZone()` method

---

### Optimizability

#### Proposed (3/3): Other changes

- Rename `Temporal.Now.timeZone()` → `Temporal.Now.timeZoneId()`
  - Returns a string to encourage fast path use
- Remove calendars altogether from `Temporal.PlainTime`
- Change time zone & calendar comparison semantics
  - New behavior: SameValue with `.id` fallback
- Remove extra reads of `timeZone` property in `ZonedDateTime.p.with()`

---

### Optimizability

#### Values accepted by `Calendar.from`/`TimeZone.from`

- Previously could pass a property bag
  - e.g. `Temporal.Calendar.from({ day, month, year, calendar })`
  - Distinguished from custom calendar object by `calendar` prop
  - Now breaks "string = fast builtin, object = slow custom" principle
- New behaviour
  - Objects only accepted _if they completely implement the protocol_
  - Separate PR: [#2485](https://github.com/tc39/proposal-temporal/pull/2485)

---

### Optimizability

#### Values accepted by other `from`

- Property bags for creating other Temporal types, e.g.
```js
Temporal.ZonedDateTime.from({ year, month, day, calendar, timeZone })
```
- `calendar`/`timeZone` properties can be string or object here
- No change from status quo
- Considered distinguishing `calendarId` (only string) and `calendar` (only object, or both) but seems unnecessary at this time

---

### Optimizability

#### Fallout from this change

- This is one of the limited areas where we'll consider adjustments between now and the next plenary
- Based on implementation concerns, experience from users of polyfills, or unexpected consequences to developer experience

---

### Spelling of `calendarId`/`timeZoneId`

- Objection in GitHub: `calendarId` and `timeZoneId` are not OK
- Alternatives considered but not chosen:
  - `calendarID`/`timeZoneID`: Violates [W3C casing rules](https://w3ctag.github.io/design-principles/#casing-rules). We aren't bound by those, but also don't want to confuse devs who are used to `getElementById`.
  - `calendarCode`/`timeZoneCode`: OK, but much more churn than necessary (to change existing `id` properties to `code`)
  - Single `timeZone: string | TimeZoneProtocol` property: We were unsure if polymorphic properties are OK

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
