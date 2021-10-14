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
_footer: DRAFT: items with "❌ spec text" will be removed before agenda deadline
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
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

<!-- _footer: ✅ spec text ❌ tests -->

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

<!-- _footer: ❌ spec text ❌ tests [#1832](https://github.com/tc39/proposal-temporal/issues/1832) -->

### `new Duration()` throws on non-integer (#TODO)

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

<!-- _footer: ❌ spec text ❌ tests [#1821](https://github.com/tc39/proposal-temporal/issues/1821), [#1782](https://github.com/tc39/proposal-temporal/issues/1782) -->

### `relativeTo` PlainDate/ZonedDateTime (#TODO)

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

<!-- _footer: ❌ spec text ❌ tests [#1763](https://github.com/tc39/proposal-temporal/issues/1763) -->

### Make fractionalSecondDigits consistent (#TODO)

- In toString methods, `fractionalSecondDigits` is ignored if seconds not displayed
- Make Temporal.Duration consistent with this

```js
duration = Temporal.Duration.from({ years: 3 });
duration.toString({ fractionalSecondDigits: 2 });
  // Before: 'P3YT0.00S'
  // After: 'P3Y'
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Consistent order of operations in ZDT with() (PR [#1865](https://github.com/tc39/proposal-temporal/pull/1865))

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
- Fixes an inconsistency in the order of user-visible operations
- Allows implementors to be slightly more efficient

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: ✅ spec text ✅ tests -->

### Totally wrong PlainTime property bag ([#1862](https://github.com/tc39/proposal-temporal/pull/1862))

```js
Temporal.PlainTime.from({ hour: 19, minute: 39, second: 9 });
// Intended: PlainTime representing 19:39:09
// Actual, according to current spec text: throws TypeError
```

- PlainTime property bags unintentionally had to have all 6 properties!

---

<!-- _footer: ✅ spec text ❌ tests -->

### Duration string with fraction ([#1759](https://github.com/tc39/proposal-temporal/pull/1759))

- Off-by-one string indexing error in the spec text

```js
Temporal.Duration.from('PT0.1S').milliseconds
// Intended: 100
// Actual, according to current spec text: 10
```

---

<!-- _footer: ✅ spec text ❌ tests -->

### Time zone offset string with fraction ([#1830](https://github.com/tc39/proposal-temporal/pull/1830))

- Another off-by-one string indexing error

```js
Temporal.TimeZone.from('+00:00:00.1').getOffsetStringFor(Temporal.Now.instant())
// Intended: "+00:00:00.1"
// Actual, according to current spec text: "+00:00"
```
---

<!-- _footer: ✅ spec text ❌ tests -->

### Time zone offset string sign ([#1833](https://github.com/tc39/proposal-temporal/pull/1833))

- While we're on the subject of time zone offsets...

```js
Temporal.TimeZone.from('-07:30').getOffsetStringFor(Temporal.Now.instant())
// Intended: "-07:30"
// Actual, according to current spec text: "--07:-30"
```

---

<!-- _footer: ❌ spec text ❌ tests [#1791](https://github.com/tc39/proposal-temporal/issues/1791) -->

### Bug in Duration comparison (#TODO)

- Incorrect comparison with time zone offset shifts

```js
// Note: 2020-11-01 is a 25-hour day in America/Vancouver time zone
relativeTo = Temporal.ZonedDateTime.from('2019-11-01T00:00-07:00[America/Vancouver]');
d = Temporal.Duration.from({ years: 1, days: 1 })

Temporal.Duration.compare(d, { years: 1, hours: 25 }, { relativeTo })
  // Correct answer: 0
  // According to current spec text: -1
```

---

<!-- _footer: ✅ spec text ❌ tests -->

### Bug in Duration string serialization ([#1860](https://github.com/tc39/proposal-temporal/pull/1860))

- Failed to account for the case of 0 decimal digits

```js
d = Temporal.Duration.from({ seconds: 5 });
d.toString({ fractionalSecondDigits: 0 })
  // Intended: "PT5S"
  // Actual, according to current spec text: "PT5.000000000S"
```

---

<!-- _footer: ✅ spec text ❌ tests -->

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

<!-- _footer: ✅ spec text ❌ tests -->

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

TODO: check similar fallback for CalendarMergeFields?

---

<!-- _footer: ❌ spec text ❌ tests [#1864](https://github.com/tc39/proposal-temporal/issues/1864) -->

### Consistent units handling in PlainDate (#TODO)

- In `since()` and `until()` methods,

```js
date1 = Temporal.PlainDate.from('1970-01-01');
date2 = Temporal.Now.plainDateISO()
date1.until(date2, { smallestUnit: 'year' })
// Before: throws RangeError (because default largestUnit < smallestUnit)
// After: Duration of { years: 51 }
```

---

<!-- _footer: ❌ spec text ❌ tests [#1685](https://github.com/tc39/proposal-temporal/issues/1685) -->

### Consistent default options (#TODO)

- When Temporal invokes a calendar operation with the default options:
  - Sometimes passed `undefined` as the options argument
  - Sometimes passed `Object.create(null)` as the options argument
- This should be consistent, because it is observable in userland calendars

---

<!-- _footer: ❌ spec text ❌ tests [#1805](https://github.com/tc39/proposal-temporal/issues/1805) -->

### Mistake in grammar of time zone names (#TODO)

- Special `Etc/GMT` time zones not correctly parsed

```js
Temporal.TimeZone.from('2000-01-01T00:00-07:00[Etc/GMT+7]').id
// Intended: "Etc/GMT+7"
// Actual, according to current spec text: "-07:00"
```

---

<!-- _footer: ✅ spec text ❌ tests -->

### Mistake in grammar of ISO 8601 strings ([#1796](https://github.com/tc39/proposal-temporal/pull/1796))

- Affects strings with a time zone offset with fractional seconds, e.g. `2020-10-25T07:45:24.123-00:00:00.321`
- Grammar was ambiguous as to which fraction should be used for the time (.123 or .321)
- **FIXME:** further fix `2020-10-25T07:45:24.123-00:00:00.321[+11:11:11.1]` if there is time before the agenda deadline ([#1797](https://github.com/tc39/proposal-temporal/issues/1797))

---

<!-- _footer: ✅ spec text ❌ tests -->

### Typos that were normative 😱

- Fix algorithms that don't work as described in the current spec text due to typos
- List of pull requests:
  - [#1780](https://github.com/tc39/proposal-temporal/pull/1780)

---

<!-- _class: invert lead -->

# Asking for consensus

On the normative PRs discussed in the previous slides
