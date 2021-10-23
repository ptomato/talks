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
  section pre { margin: 0.5em 0 }
  section li li { font-size: 85% }
  section li li pre { margin: 0.1em 0 }
  section.smaller-type li { font-size: 90% }
  section.smaller-type li li { font-size: 88% }
  section.smaller-type li pre { margin: 0.1em 0 }
  h3 { margin-bottom: 0.8em }
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
- One normative adjustment requires discussion and may be contentious, so it's at the end

---

<!-- _class: invert lead -->

# Adjustments

---

### No sub-minute time zone offsets (PR [#1871](https://github.com/tc39/proposal-temporal/pull/1871))

- IETF working group for standardizing extensions to ISO string serialization format requested that we drop this extension
- Ujjwal mentioned this in the August plenary
- Change to spec text is ready now

---

### No sub-minute time zone offsets (cont'd)

- Change `ZonedDateTime.p.toString` and `Instant.p.toString` to output time zone offsets only with minutes precision
- Change `ZonedDateTime.from` to accept HH:MM precision for non-minute-offset time zones, even with `{ offset: 'reject' }`.
- No change to `offset` property of `ZonedDateTime`, or in property bags, or to `TimeZone.p.getOffsetStringFor`.

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

### `YYYY-MM-DDThh:mmZ` as PlainDate string (PR [#1874](https://github.com/tc39/proposal-temporal/pull/1874))

```js
Temporal.PlainDate.from('2020-10-25T11:00Z')
  // Before: 2020-10-25
  // Proposed change: throw
```

- Current behavior is risky when deserializing from DBs that sometimes attach local-time-zone semantics to ISO strings
- Can cause off-by-one-day bugs that affect users
- Ignoring this risk is still possible with more explicit, verbose code:

```js
parseDateUnsafe = s => Temporal.Instant.from(s).toZonedDateTime(s).toPlainDate()
```

---

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

### `relativeTo` PlainDate/ZonedDateTime ([#1873](https://github.com/tc39/proposal-temporal/pull/1873))

- Several operations have a `relativeTo` option
- Previously treated as `PlainDateTime` or `ZonedDateTime`
- The time component was only used if it was a `ZonedDateTime`
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

### Consistent order of operations in ZonedDateTime `with()` (PR [#1865](https://github.com/tc39/proposal-temporal/pull/1865))

- Fixes an inconsistency in the order of observable operations
- Allows implementors to be slightly more efficient

---

### Consistent order of operations in ZonedDateTime `with()` (cont'd)

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

### Totally wrong PlainTime property bag ([#1862](https://github.com/tc39/proposal-temporal/pull/1862))

```js
Temporal.PlainTime.from({ hour: 19, minute: 39, second: 9 });
// Intended: PlainTime representing 19:39:09
// Actual, according to current spec text: throws TypeError
```

- PlainTime property bags unintentionally had to have all 6 properties!

---

### Duration string with fraction ([#1759](https://github.com/tc39/proposal-temporal/pull/1759))

- Off-by-one string indexing error in the spec text

```js
Temporal.Duration.from('PT0.1S').milliseconds
// Intended: 100
// Actual, according to current spec text: 10
```

---

### Time zone offset string with fraction ([#1830](https://github.com/tc39/proposal-temporal/pull/1830))

- Another off-by-one string indexing error

```js
Temporal.TimeZone.from('+00:00:00.1').getOffsetStringFor(Temporal.Now.instant())
// Intended: "+00:00:00.1"
// Actual, according to current spec text: "+00:00"
```
---

### Time zone offset string sign ([#1833](https://github.com/tc39/proposal-temporal/pull/1833))

- While we're on the subject of time zone offsets...

```js
Temporal.TimeZone.from('-07:30').getOffsetStringFor(Temporal.Now.instant())
// Intended: "-07:30"
// Actual, according to current spec text: "--07:-30"
```

---

### Bug in Duration string serialization ([#1860](https://github.com/tc39/proposal-temporal/pull/1860))

- Failed to account for the case of 0 decimal digits

```js
d = Temporal.Duration.from({ seconds: 5 });
d.toString({ fractionalSecondDigits: 0 })
  // Intended: "PT5S"
  // Actual, according to current spec text: "PT5.000000000S"
```

---

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

### Mistake in grammar of ISO 8601 strings ([#1796](https://github.com/tc39/proposal-temporal/pull/1796))

- Affects strings with a time zone offset with fractional seconds, e.g. `2020-10-25T07:45:24.123-00:00:00.321`
- Grammar was ambiguous as to which fraction should be used for the time (.123 or .321)

---

### Typo in UnbalanceDurationRelative ([#1780](https://github.com/tc39/proposal-temporal/pull/1780))

- Fix an algorithm that doesn't work as described in the current spec text due to a typo

---

### Bug in PlainDateTime/PlainTime `since` ([#1875](https://github.com/tc39/proposal-temporal/pull/1875))

- Spec reversed arguments
  - Intended: calculate from `this` to `other`
  - Current: calculate from `other` to `this`
- Only affects `{largestUnit: 'years'}` or `{largestUnit: 'months'}` 
- PR changes spec to match polyfill, tests, and docs
- ⏳ This bug was discovered after advancement deadline

---

<!-- _class: invert lead -->

# Asking for consensus

On the normative PRs discussed in the previous slides

---

<!-- _class: invert lead -->

# Discussion

---

<!-- _class: smaller-type -->

### Patterns for optional/required params ([#1756](https://github.com/tc39/proposal-temporal/issues/1756))

- Optional params SHOULD be a property bag to support future extension
  ```js
  JSON.stringify({ x }, undefined, 2) // ❌ SHOULD NOT make new APIs with positional, optional params

  new Intl.DisplayNames(undefined, { type }) // ✅ MAY stay consistent with existing APIs
  ```

- Required params SHOULD NOT be property bags

  - Exception: "primitive | bag" polymorphic param MAY be required
    ```js
    duration.round() // ❌ throws because it's a no-op
    duration.round('day') // ✅
    duration.round({ smallestUnit: 'day' }) // ✅ same as .round('day')
    duration.round({ smallestUnit: 'day', roundingIncrement: 7 }) // ✅ other optional props
    ```
  - Exception: bags for "data" (not options) MAY be required
    ```js
    plainDate.with()  // ❌ throws because it's a no-op
    plainDate.with({ day: 1, month: 2 }) // ✅ required param with optional props
    ```
---

### Param patterns: open issue

- If a primitive corresponds to a bag property, is that property always required in object form?
  - Choice 1: MUST. "Required" should mean it's required everywhere
  - Choice 2: SHOULD, with exceptions for unusual cases (like below)
```js
// "1 of N required" or mutually-exclusive properties
duration.round('day')
duration.round({ smallestUnit: 'day' })
duration.round({ largestUnit: 'month' }) // either smallestUnit or largestUnit is required

// primitives that are aggregations
f(7)
f({ read: true, write: true, execute: true }) // at least one property req'd
```

---

### Strings in place of req'd options bag (PR [#1875](https://github.com/tc39/proposal-temporal/pull/1875))

- Reviewer feedback: options bags should be optional!
- Strings replace req'd options bag; advanced cases can use objects
- Non-breaking change for `*.p.round` and `Duration.p.total`

```js
durationOpts = { largestUnit: 'hours', smallestUnit: 'milliseconds', roundingMode: 'trunc' }
duration = datetime.since('2020-01-01', durationOpts)

duration.round('seconds') // proposed equivalent to { smallestUnit: 'seconds' }
duration.round({ largestUnit: 'milliseconds' }) // smallestUnit OR largestUnit req'd
duration.round({ ...durationOpts, roundingIncrement: 100 }) // same options shape as until() and since()

duration.total('days') // proposed equivalent to { unit: 'days' }
duration.total({ unit: 'months', relativeTo: '2020-01-01[America/Denver]' })
```

---

### Alternatives considered and rejected

```js
// ❌ why let users write code that's guaranteed to be wrong?
pdt.round();

// ❌ splitting options bags on required vs. optional seems wacky, verbose, and hostile
pdt.round({ smallestUnit: 'day' }, { roundingMode: 'ceil' });

// ❌ less bad than above, but still makes it harder for users to learn about
// similarity between round() and until()/since() options
pdt.round('day', { roundingMode: 'ceil' });
```

<!--
---

### DISCUSSION: req'd vs. optional params patterns?

Problem to solve: options objects where properties are required in some APIs or cases but optional in others.

```js
pdt = Temporal.PlainDateTime.from('2021-10-28T10:00');
roundingOpts = { smallestUnit: 'day', roundingMode: 'ceil' };

// `smallestUnit` option is optional for `until` and `since`
notRounded = pdt.since('2021-01-01');
fullDays = pdt.since('2021-01-01', { smallestUnit: 'day' });
partialDays = pdt.since('2021-01-01', roundingOpts);

// `smallestUnit` option is required for `round` 
pdt.round(); // throws (to avoid no-op calls)
closestMidnight = pdt.round({ smallestUnit: 'day' });
nextMidnight = pdt.round(roundingOpts);
```
-->
