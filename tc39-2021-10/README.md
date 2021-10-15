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

<!--
_class: invert lead
_footer: DRAFT: tests tracking footers will be removed before presenting
-->

# ⌚ **Temporal**

**Philip Chimento** (Igalia, in partnership with Bloomberg)  
**Justin Grant** (invited expert for Temporal)  
TC39 October 2021

---

# Temporal update

- Similar to last time!
- Asking for consensus on several minor normative changes
  - Changes suggested by implementors ("adjustments")
  - Changes originally intended by the champions but incorrectly expressed in the spec text ("bugs")

---

<!-- _class: invert lead -->

# Adjustments

---

<!-- _footer: ❌ tests -->

### No sub-minute time zone offsets (PR [#1871](https://github.com/tc39/proposal-temporal/pull/1871))

- IETF working group for standardizing extensions to ISO string serialization format requested that we drop this extension
- Ujjwal mentioned this in the August plenary
- Change to spec text is ready now

---

### No sub-minute time zone offsets (cont'd)

- Change ZonedDateTime.toString and Instant.toString to output time zone offsets only with minutes precision
- Change ZonedDateTime.from to accept HH:MM precision for non-minute-offset time zones, even with `{ offset: 'reject' }`.
- No change to `offset` property of ZonedDateTime, or in property bags, or to TimeZone.getOffsetStringFor.

---

### No sub-minute time zone offsets (cont'd)

```js
timeZone = Temporal.TimeZone.from('Africa/Monrovia');
zdt = Temporal.PlainDate.from('1972-01-01').toZonedDateTime(timeZone);

// Before:
zdt.toString() === '1972-01-01T00:00:00-00:44:30[Africa/Monrovia]'
zdt.toInstant().toString({ timeZone }) === '1972-01-01T00:00:00-00:44:30'
zdt.offset === '-00:44:30'
timeZone.getOffsetStringFor(zdt.toInstant()) === '-00:44:30'
Temporal.ZonedDateTime.from('1972-01-01T00:00:00-00:44:30[Africa/Monrovia]').equals(zdt)
Temporal.ZonedDateTime.from('1972-01-01T00:00:00-00:45[Africa/Monrovia]') // throws

// After:
zdt.toString() === '1972-01-01T00:00:00-00:45[Africa/Monrovia]'
zdt.toInstant().toString({ timeZone }) === '1972-01-01T00:00:00-00:45'
zdt.offset === '-00:44:30'
timeZone.getOffsetStringFor(zdt.toInstant()) === '-00:44:30'
Temporal.ZonedDateTime.from('1972-01-01T00:00:00-00:44:30[Africa/Monrovia]').equals(zdt)
Temporal.ZonedDateTime.from('1972-01-01T00:00:00-00:45[Africa/Monrovia]').equals(zdt)
```

---

<!-- _footer: ❌ tests -->

### `YYYY-MM-DDThh:mmZ` as PlainDate string ([#1874](https://github.com/tc39/proposal-temporal/pull/1874))

```js
Temporal.PlainDate.from('2020-10-25T11:00Z')
  // Before: 2020-10-25
  // Proposed change: throw
```

- Accepting this kind of string in Plain types poses a risk when deserializing from DBs
- Some DBs attach local-time-zone semantics to this kind of string
- Could result in off-by-one-day bugs that affect users
- On the other hand, this makes some other use cases like extracting part of an ISO string harder

---

<!-- _footer: ❌ tests -->

### Options bag where an option is required ([#1875](https://github.com/tc39/proposal-temporal/pull/1875))

(placeholder)

---

<!-- _footer: ❌ tests -->

### `new Duration()` throws on non-integer ([#1872](https://github.com/tc39/proposal-temporal/pull/1872))

- `new Temporal.Duration(0, 0, 0, 0, 1.1)` should not silently drop the 0.1 hour
- Brings constructor in line with other ways to create a Duration

```js
// Unchanged:
Temporal.Duration.from('PT1.1H')  // 1 hour 6 minutes; valid ISO string
Temporal.Duration.from({ hours: 1.1 })  // throws RangeError because not exact

// Before:
new Temporal.Duration(0, 0, 0, 0, 1.1)  // 1 hour (!)

// After:
new Temporal.Duration(0, 0, 0, 0, 1)  // throws RangeError because not exact
```

---

<!-- _footer: ❌ tests -->

### `relativeTo` PlainDate/ZonedDateTime ([#1873](https://github.com/tc39/proposal-temporal/pull/1873))

- Several operations have a `relativeTo` option
- Previously treated as PlainDateTime or ZonedDateTime
- The time component was only used if it was a ZonedDateTime
- Simplifies things slightly for implementors

---

### `relativeTo` PlainDate/ZonedDateTime (cont'd)

```js
// Before:
class MyCalendar extends Temporal.Calendar {
  constructor() { super('iso8601'); }
  dateAdd(date, duration, options) {
    console.log(date[Symbol.toStringTag]);
    return super.dateAdd(date, duration, options);
  }
}
const relativeTo = Temporal.Now.plainDateTime(new MyCalendar());
Temporal.Duration.from({ days: 7 }).round({ largestUnit: 'weeks', relativeTo });
  // Before: logs 2x "Temporal.PlainDateTime"
  // After: logs 2x "Temporal.PlainDate"
```

---

<!-- _footer: ✅ tests -->

### Consistent order of operations in ZonedDateTime with() (PR [#1865](https://github.com/tc39/proposal-temporal/pull/1865))

- Fixes an inconsistency in the order of observable operations
- Allows implementors to be slightly more efficient

---

### Consistent order of operations in ZonedDateTime with() (cont'd)

```js
class C extends Temporal.Calendar {
  constructor() { super('iso8601'); }
  mergeFields(f1, f2) {
    console.log('boo!');
    return super.mergeFields(f1, f2);
  }
}
const dateTime = Temporal.Now.zonedDateTime(new C());
Object.defineProperty(dateTime, "offset", { value: undefined });
dateTime.with({ year: 2022 });
// Before: logs boo!, then throws TypeError
// After: throws TypeError
```

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: ✅ tests -->

### Totally wrong PlainTime property bag ([#1862](https://github.com/tc39/proposal-temporal/pull/1862))

```js
Temporal.PlainTime.from({ hour: 19, minute: 39, second: 9 });
// Intended: PlainTime representing 19:39:09
// Actual, according to current spec text: throws TypeError
```

- PlainTime property bags unintentionally had to have all 6 properties!

---

<!-- _footer: ❌ tests -->

### Duration string with fraction ([#1759](https://github.com/tc39/proposal-temporal/pull/1759))

- Off-by-one string indexing error in the spec text

```js
Temporal.Duration.from('PT0.1S').milliseconds
// Intended: 100
// Actual, according to current spec text: 10
```

---

<!-- _footer: ❌ tests -->

### Time zone offset string with fraction ([#1830](https://github.com/tc39/proposal-temporal/pull/1830))

- Another off-by-one string indexing error

```js
Temporal.TimeZone.from('+00:00:00.1').getOffsetStringFor(Temporal.Now.instant())
// Intended: "+00:00:00.1"
// Actual, according to current spec text: "+00:00"
```
---

<!-- _footer: ❌ tests -->

### Time zone offset string sign ([#1833](https://github.com/tc39/proposal-temporal/pull/1833))

- While we're on the subject of time zone offsets...

```js
Temporal.TimeZone.from('-07:30').getOffsetStringFor(Temporal.Now.instant())
// Intended: "-07:30"
// Actual, according to current spec text: "--07:-30"
```

---

<!-- _footer: ❌ tests -->

### Bug in Duration string serialization ([#1860](https://github.com/tc39/proposal-temporal/pull/1860))

- Failed to account for the case of 0 decimal digits

```js
d = Temporal.Duration.from({ seconds: 5 });
d.toString({ fractionalSecondDigits: 0 })
  // Intended: "PT5S"
  // Actual, according to current spec text: "PT5.000000000S"
```

---

<!-- _footer: ❌ tests -->

### Wrong rounding mode ([#1777](https://github.com/tc39/proposal-temporal/pull/1777))

- A repeated line in the spec text undid the effect of the first line

```js
april = Temporal.PlainYearMonth.from('2021-04');
october = Temporal.PlainYearMonth.from('2021-10');
october.since(april, { smallestUnit: 'year', roundingMode: 'ceil' }).years
// Intended: 1
// Actual, according to current spec text: 0
october.since(april, { smallestUnit: 'year', roundingMode: 'floor' }).years
// Intended: 0
// Actual, according to current spec text: 1
```

---

<!-- _footer: ❌ tests -->

### Remove `getOffsetNanosecondsFor` fallback ([#1829](https://github.com/tc39/proposal-temporal/pull/1829))

- An earlier version of the proposal had fallbacks like this
- This one remained unintentionally
- Implementor feedback: this wastes memory

```js
timeZone = new Temporal.TimeZone('UTC');
timeZone.getOffsetNanosecondsFor = undefined;
timeZone.getOffsetStringFor(Temporal.Now.instant())
// Before: "+00:00"
// After: throws TypeError
```

---

<!-- _footer: ❌ tests -->

### Mistake in grammar of ISO 8601 strings ([#1796](https://github.com/tc39/proposal-temporal/pull/1796))

- Affects strings with a time zone offset with fractional seconds, e.g. `2020-10-25T07:45:24.123-00:00:00.321`
- Grammar was ambiguous as to which fraction should be used for the time (.123 or .321)

---

<!-- _footer: ❌ tests -->

### Typo in UnbalanceDurationRelative ([#1780](https://github.com/tc39/proposal-temporal/pull/1780))

- Fix an algorithm that doesn't work as described in the current spec text due to a typo

---

<!-- _class: invert lead -->

# Asking for consensus

On the normative PRs discussed in the previous slides
