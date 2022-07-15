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
TC39 July 2022

---

# Temporal update

- We continue to adjust the proposal based on implementor feedback and bug reports
- Fixes continue to become more and more obscure edge cases
- More to present this time than last time, due to SpiderMonkey implementation advancing
- This month's major bug finder is **André Bargull**, thanks :tada:

---

# Temporal update

- Will continue turning implementor feedback into presentations like this as bandwidth allows

---

# Sneak peek for next plenary

Three major pieces of implementor feedback remaining to address:

- Investigate optimizing the built-in calendar case (issue [#1808](https://github.com/tc39/proposal-temporal/issues/1808))
- Integrate Calendar and TimeZone into Intl.DTF options ([#2005](https://github.com/tc39/proposal-temporal/issues/2005))
- Investigate removing [[Calendar]] slot from PlainTime ([#1588](https://github.com/tc39/proposal-temporal/issues/1588))

(+ more edge cases as they come up)

<!--
    I hope to present all of these in September. Follow along with the issues if you are interested in these topics.
-->

---

# IETF update

- IETF meeting coming up 2022-07-25
- Issue open to implement conclusions ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))

---

<!-- _class: invert lead -->

# Adjustments
to the proposal

---

### Consistency in copying object props (PR [#2245](https://github.com/tc39/proposal-temporal/pull/2245))

- Ensure that all steps that copy properties from one object to another use CopyDataProperties, for consistency with the rest of ECMA-262
- Changes order of operations observably:
  - Before: sequence of [[GetOwnProperty]] then sequence of [[Get]]
  - After: interleaved pairs of [[GetOwnProperty]] + [[Get]]
- Also now copies symbol keys
  - (though Temporal doesn't use them :shrug:)

---

### Better validation of calendar / time zone values

- User-defined calendars and time zones can return all sorts of nonsense from their methods
- In most cases we throw on obvious nonsense
- Anba picked out a few more cases that we should throw on:
  - Time zone offsets are &lt; 24 hours, not &le; (PR [#2260](https://github.com/tc39/proposal-temporal/pull/2260))
  - Time zones shouldn't be able to compute a negative day length (PR [#2261](https://github.com/tc39/proposal-temporal/pull/2261))
  - Various calendar methods (PR [#2265](https://github.com/tc39/proposal-temporal/pull/2265))

---

### Align rounding modes with NFv3 (PR [#2262](https://github.com/tc39/proposal-temporal/pull/2262))

- When Temporal got Stage 3, the NumberFormat v3 proposal had not decided which rounding modes to include
- We tentatively chose `ceil`, `floor`, `trunc`, and `halfExpand` to match the four kinds of rounding that you can do with `Math` methods
- Understanding was that Temporal would align with NumberFormat
- NumberFormat now includes `expand`, `halfCeil`, `halfFloor`, `halfTrunc`, `halfEven` as well

---

### Restrict reference year in PlainMonthDay constructor (PR [#2266](https://github.com/tc39/proposal-temporal/pull/2266))

- There was mistakenly no limit on the reference year<sup>1</sup> of a PlainMonthDay

```js
new Temporal.PlainMonthDay(9, 30, "iso8601", 999_999_999_999_999)
  // Before: creates a PlainMonthDay object with an invalid value in internal slot
  // After: throws RangeError
```

<hr/>

<div style="font-size: 22px; line-height: 1.1;">
<sup>1</sup> "What on earth is the reference year??" PlainMonthDay stores a year in the ISO calendar as part of its internal data model. User code doesn't normally interact with it, but it can be set as an optional constructor argument so that custom calendars can set their own values for it)
</div>

---

### Avoid repeated method [[Get]] ops (PR [#2267](https://github.com/tc39/proposal-temporal/pull/2267))

- In some operations (e.g., rounding a duration relative to a date) the `relativeTo.calendar.dateAdd` property was [[Get]]ted repeatedly.

```js
let gets = 0;
const calendar = new class extends Temporal.Calendar {
  get dateAdd() {
    ++gets;
    return Temporal.Calendar.prototype.dateAdd;
  }
}('iso8601');
const relativeTo = Temporal.Now.plainDate(calendar);
Temporal.Duration.from({ months: 18 }).round({ smallestUnit: 'years', relativeTo });
console.log(gets);
  // Before: 5
  // After: 1
```

---

### Avoid unnecessary observable toString call (PR [#2269](https://github.com/tc39/proposal-temporal/pull/2269))

- In e.g. `zonedDateTime.toString({ calendarName: 'never' })` we don't need to call `toString()` on the calendar object

```js
const calendar = new class extends Temporal.Calendar {
  toString() { throw new Error("don't call me"); }
}('iso8601');
const zonedDateTime = Temporal.Now.zonedDateTime(calendar);
zonedDateTime.toString({ calendarName: 'never' });
  // Before: throws
  // After: string with the current date, time, & time zone, no calendar annotation
```

---

### ISO 8601 grammar (PR [#2284](https://github.com/tc39/proposal-temporal/pull/2284), [#2287](https://github.com/tc39/proposal-temporal/pull/2287), [#2345](https://github.com/tc39/proposal-temporal/pull/2345))

- Clarify `192312[America/New_York]` is a year-month string, not `19:23:12`
- Clarify `1901-04[America/New_York]` is a year-month string, not `19:01` with UTC-4 offset
- Clarify `192312[u-ca=iso8601]` is a year-month string, not `19:23:12`
- Accept and ignore calendar in Instant strings: e.g. `2022-07-18T10:00Z[u-ca=gregory]`
- A few more mistakes in disambiguating HHMMSS from YYYYMM

---

### Support legacy IANA time zone names (PR [#2292](https://github.com/tc39/proposal-temporal/pull/2292))

- Legacy names in the IANA time zone database such as `Etc/GMT0`, `GMT+0`, `PST8PDT`
- As with other IANA 'Link Names' these are canonicalized

```js
new Temporal.TimeZone('Etc/GMT0')
  // Before: throw RangeError
  // After: creates Temporal.TimeZone with ID 'UTC'
```

---

### Validation of integer options values (PR [#2297](https://github.com/tc39/proposal-temporal/pull/2297))

- First truncate, then validate that the value is within range
- This is how e.g. `Number.prototype.toFixed()` works
- Consistently different between ECMA-262 and ECMA-402
- [Discussion open](https://github.com/tc39/ecma402/issues/691) in ECMA-402 to align with 262

```js
new Temporal.PlainTime().toString({ fractionalSecondDigits: 9.1 })
  // Before: throw RangeError (value is > 9)
  // After: "00:00:00.000000000" (9 fractional digits)
```

---

### Handle ±∞ in Duration.p.total (PR [#2298](https://github.com/tc39/proposal-temporal/pull/2298))

- Durations do not tolerate infinities in their components
- However, calculating a total can result in infinity
- Make `total()` method consistent with what you would get if you calculated the total manually
  - (e.g. `Number.MAX_VALUE * 1000` in the example below)

```js
d = Temporal.Duration.from({ microseconds: Number.MAX_VALUE });
d.round({ largestUnit: 'nanoseconds' })
  // throws RangeError (unchanged)
d.total('nanoseconds')
  // Before: fails assertion in spec
  // After: Infinity
```

---

### Remove unnecessary property access (PR [#2316](https://github.com/tc39/proposal-temporal/pull/2316))

- Obscure edge case where iterator methods are called on the input to `calendar.fields()` method even if the method itself is not called because the property isn't present

```js
const calendar = {
  dateFromFields() { throw new Error('do call me'); }
};
Array.prototype[Symbol.iterator] = function () {
  throw new Error("don't call me");
}
Temporal.PlainDate.from({ year: 2022, month: 7, day: 18, calendar })
  // Before: throws "don't call me"
  // After: throws "do call me"
```

---

### Correct duration balancing algorithm (PR [#2344](https://github.com/tc39/proposal-temporal/pull/2344))

- Incorrect translation of the code into specification steps led to durations being balanced relative to the wrong date.

```js
const relativeTo = Temporal.PlainDate.from('1972-02-01');
const duration = Temporal.Duration.from({ years: 3, months: 11, days: 28 });
// (this duration is one day short of 4 years because February 1972 had 29 days)
duration.round({ largestUnit: 'years', relativeTo });
  // Before: result is a duration of 4 years
  // After: result is the same as the input duration
```

---

### ~~Edge case in nested calendar props~~ (PR [#2350](https://github.com/tc39/proposal-temporal/pull/2350))

- (Needs more discussion after all, withdrawn)

---

### Extreme time zone transitions (PR [#2351](https://github.com/tc39/proposal-temporal/pull/2351))

- `Temporal.TimeZone.p.getNextTransition()` at the end of the allowed range for `Temporal.Instant` should not throw
- Consistent with what we do for `Temporal.Now.instant()` if the system clock is past the end of the range
- In the year 275760 will we still have time zone transitions anyway?

```js
const lastTime = Temporal.Instant.fromEpochSeconds(86400_0000_0000);
const tz = Temporal.TimeZone.from('America/Vancouver');
tz.getNextTransition(lastTime);
  // Before: throws RangeError
  // After: null
```

<!--
---

### Remove unnecessary call to user code (PR [#2346](https://github.com/tc39/proposal-temporal/pull/2346))

- Converting the `relativeTo` option from a ZonedDateTime to a PlainDate in RoundDuration involves a call to a time zone method
- Do this only for certain `smallestUnit` values when it is needed, instead of unconditionally

```js
const relativeTo = Temporal.Now.zonedDateTimeISO({
  getOffsetNanosecondsFor() { throw new Error("don't call me"); }
});
Temporal.Duration.from({ seconds: 120 }).round({ smallestUnit: 'minutes', relativeTo });
  // Before: throws "don't call me"
  // After: a duration of 2 minutes
```
-->

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
