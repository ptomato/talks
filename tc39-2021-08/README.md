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
_footer: <span style="color:yellow;">**Status: DRAFT**</span>
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 August/September 2021

<!--
-->

---

# Temporal update

- IETF string format standardization update
- Asking for consensus on several minor normative changes
  - Changes originally intended by the champions but incorrectly expressed in the spec text ("bugs")
  - Changes suggested by implementors ("adjustments")

---

<!-- _class: invert lead -->

# IETF string format standardization

---

TODO

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: ✅ spec text ❌ tests -->

### Double seconds (PR [#1592](https://github.com/tc39/proposal-temporal/pull/1592))

```js
duration = Temporal.Duration.from({ minutes: 5, seconds: 30 })

// Intended:
duration.toString() === 'PT5M30S'
// Actual, according to the current spec text:
duration.toString() === 'PT5M30S30S'
```

- Obviously not the intention

---

<!-- _footer: ✅ spec text ✅ tests -->

### Observable order of Duration checks (PR [#1601](https://github.com/tc39/proposal-temporal/pulls/1601))

```js
called = 0;
observer = { valueOf() { called++; return Infinity; }};
d = new Temporal.Duration(observer, observer); // => throws RangeError

// Intended:
called === 1
// Actual, according to the current spec text:
called === 2
```
- Other Temporal types have the intended behaviour

---

<!-- _footer: ✅ spec text ✅ tests -->

### Accidental observable property access (PR [#1616](https://github.com/tc39/proposal-temporal/pulls/1616))

```js
d = Temporal.Duration.from('P12M');
Object.defineProperty(d, 'months', { get() { return Infinity; }});
d.total({ unit: 'days', relativeTo: Temporal.Now.plainDateTimeISO() });
// Throws according to current spec text. Not intended
```

An accidental <span style="color: darkred;">Get(_duration_, **"months"**)</span> instead of <span style="color: darkred;">_duration_.[[Months]]</span>

---

<!-- _footer: ✅ spec text ✅ tests -->

### Wrong error type (PR [#1646](https://github.com/tc39/proposal-temporal/pulls/1646))

```js
instant = Temporal.Now.instant();
instant.round();  // missing a unit to round to

// Intended: throws TypeError
// Actual, according to current spec text: throws RangeError
```

- `TypeError` is more appropriate here than `RangeError`

---

<!-- _footer: ❌ spec text ❌ tests -->

### (PR [#1638](https://github.com/tc39/proposal-temporal/pull/1638))

TODO

---

<!-- _footer: ✅ spec text ❌ tests -->

### Fix tautological comparison (PR [#1665](https://github.com/tc39/proposal-temporal/pull/1665))

```diff
-1. If [...] and _mid_.[[Year]] is equal to _mid_.[[Year]], then
+1. If [...] and _mid_.[[Year]] is equal to _end_.[[Year]], then
```

- The algorithm described in the current spec text doesn't work due to a typo

---

<!-- _footer: ❌ spec text ❌ tests -->

### Fix wrong variable name (PR TODO)

```diff
-1. Set _fMicroseconds_ to _mils_ modulo 1.
+1. Set _fMicroseconds_ to _mics_ modulo 1.
```

- Another algorithm that doesn't work as described in the current spec text due to a typo

---

<!-- _footer: ❌ spec text ❌ tests -->

### Fix wrong value passed to user code (PR [#1667](https://github.com/tc39/proposal-temporal/pull/1667))

```js
class C extends Temporal.Calendar {
    constructor() { super('iso8601'); }
    dateAdd(date, duration, options) {
        console.log(JSON.stringify(options));
        return super.dateAdd(date, duration, options);
    }
}
plain = Temporal.PlainDateTime.from('2021-03-14T02:30').withCalendar(new C());
plain.toZonedDateTime('America/Vancouver');
// Intended: logs {"overflow":"constrain"}
// Actual, according to current spec text: logs "constrain"
```
- Passes string instead of options object to user code
- Also missing an argument

---

<!-- _footer: ❌ spec text ❌ tests -->

### Return type of `Calendar.mergeFields()` (PR [#1669](https://github.com/tc39/proposal-temporal/pull/1669))

```js
class C extends Temporal.Calendar {
    constructor() { super('iso8601'); }
    mergeFields(fields, additionalFields) {
        return "I'm not supposed to return this";
    }
}
plain = Temporal.Now.plainDate(new C());
plain.with({ day: 1 });
// Intended: throws TypeError
// In current spec text, this fails an assertion
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Totally wrong Duration property bag (PR [#1659](https://github.com/tc39/proposal-temporal/pull/1659))

```js
Temporal.Duration.from({ years: 5 });
// Intended: creates a Temporal.Duration of 5 years
// Actual, according to current spec text: throws RangeError
```

- Duration property bags unintentionally had to have all 10 properties!

---

<!-- _footer: ❌ spec text ❌ tests -->

### Mark options parameters as optional (PR [#1640](https://github.com/tc39/proposal-temporal/pull/1640))

- Affects `length` property of some functions

---

<!-- _class: invert lead -->

# Adjustments

---

<!-- _footer: ❌ spec text ❌ tests -->

### Mathematical values in Duration ([#1604](https://github.com/tc39/proposal-temporal/issues/1604))

- General principle, internal slots should store MVs
- Values of `Temporal.Duration` are unbounded
- Advantage: consistency, avoidance of subtle bugs
- Disadvantages: potentially disruptive for implementors, potentially less performant
- Thanks to André Bargull for noting this is a normative change

---

<!-- _footer: ❌ spec text ❌ tests -->

### No sub-minute time zone offsets ([#1544](https://github.com/tc39/proposal-temporal/issues/1544))

- IETF standardization of the time zone and calendar string annotations blocked on this (FIXME: change)
- 2021-03: Temporal must remain behind a feature flag until these annotations are standardized

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

<!-- _footer: ✅ spec text ❌ tests -->

### Optimization in PlainDateTime calendar accessors (PR [#1613](https://github.com/tc39/proposal-temporal/pull/1613))

- `PlainDateTime` can now be passed to `Calendar` methods.
- Prevents creation of an extra `PlainDate` object.

---

```js
class C extends Temporal.Calendar {
    constructor() { super('iso8601'); }
    year(d) {
        console.log(d[Symbol.toStringTag]);
        return super.year(d);
    }
}
plain = Temporal.Now.plainDateTime(new C());
plain.year
// Before: logs "Temporal.PlainDate"
// After: logs "Temporal.PlainDateTime"
```

---

<!-- _footer: ❌ spec text ❌ tests -->

### Guard against garbage in `Calendar.fields` (PR TODO)

- `Temporal.Calendar.prototype.fields()` expects an iterable as argument
  - Will now throw if the iterable yields any duplicate values
  - Will now throw if the iterable yields any values that are not `'year'`, `'month'`, `'monthCode'`, `'day'`, `'hour'`, `'minute'`, `'second'`, `'millisecond'`, `'microsecond'`, `'nanosecond'`

---

```js
Temporal.Calendar.from('iso8601').fields({
    *[Symbol.iterator]() {
        while(true)
            yield 'garbage';
    }
});
// Before: infinite loop
// After: throws RangeError
```

---

<!-- _footer: ✅ spec text ❌ tests -->

### Don't check options values if unused (PR [#1606](https://github.com/tc39/proposal-temporal/pull/1606))

TODO: is on agenda for next champions meeting
