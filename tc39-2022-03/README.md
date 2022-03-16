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
_footer: <strong>DRAFT</strong>, to be finalized by 2022-03-18
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 March 2022

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
  - 26 → 19 → 12 → 9 (+ maybe 1)

---

# IETF update

This slide will be filled in after the IETF meeting of 19–25 March!

---

<!-- _class: invert lead -->

# Adjustments

---

<!-- _footer: ✅ spec text ❌ tests -->

### Mathematical values in Duration (PR [#2094](https://github.com/tc39/proposal-temporal/pull/2094))

- General principle: internal slots should store MVs
  - Avoid subtle bugs
  - Avoid "speccing IEEE arithmetic"
- Implementor feedback:
  - Values of `Temporal.Duration` are unbounded integers
  - Storing 10 BigInts far less performant than 10 Numbers

---

### Mathematical values in Duration (2)

- Note that Duration fields can now store the _intersection_ of:
  - mathematical integers
  - Number values
- Specifically _not_ NaN, ±∞, −0, non-integers, integers not exactly representable in a double
- This was implicit before, and [sloppily defined](https://github.com/tc39/proposal-temporal/issues/1715). Now it is explicit.

---

### Mathematical values in Duration (3)

```js
blank = new Temporal.Duration(0);
Object.is(blank.years, blank.negated().years)
  // Before: false (blank.negated().years is -0)
  // After: true

max = Temporal.Duration.from({ nanoseconds: Number.MAX_SAFE_INTEGER });
more = max.add({ nanoseconds: 1 });
evenmore = max.add({ nanoseconds: 2 });
evenmore.nanoseconds - more.nanoseconds  // 0
evenmore.subtract(more).nanoseconds
  // Before: 1
  // After: 0
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Consistent default options (PR [#2028](https://github.com/tc39/proposal-temporal/pull/2028))

- When Temporal invokes a calendar operation with the default options, it passes as the options argument:
  - Sometimes `undefined`
  - Sometimes `Object.create(null)`
- This should be consistent, because it is observable in userland calendars

---

### Consistent default options (2)

```js
original = Temporal.Calendar.prototype.yearMonthFromFields;
Temporal.Calendar.prototype.yearMonthFromFields = function (fields, options) {
    console.log(options);
    return original.call(this, fields, options);
}

ym = Temporal.PlainYearMonth.from("2022-03")
  // calls calendar.yearMonthFromFields() internally
  // Before: logs Object with null prototype
  // After: logs undefined
```
(this affects several places, not just `PlainYearMonth.from`)

---

<!-- _footer: ✅ spec text ❌ tests -->

### Consistent expanded-year output (PR [#2090](https://github.com/tc39/proposal-temporal/pull/2090))

DRAFT, not decided yet

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: ✅ spec text ✅ tests -->

### Wrong sign in PYM.subtract (PR [#2002](https://github.com/tc39/proposal-temporal/pull/2002))

- Sign-flip error in the algorithm for PlainYearMonth.subtract
- Would produce wrong results if implemented exactly as written

---

<!-- _footer: ✅ spec text ❌ tests -->

### Calendar mix-up in PYM arithmetic (PR [#2003](https://github.com/tc39/proposal-temporal/pull/2003))

- Faulty mixing of calendar domains in PlainYearMonth.add and subtract algorithms
- Example below would incorrectly throw, but other examples might return wrong results

```js
m = Temporal.PlainYearMonth.from({ year: 2021, month: 1, calendar: 'chinese' })
  // (internally results in ISO reference date 2021-02-12)
m.daysInMonth  // => 29
m.subtract({ months: 1 })
  // Correct answer: PlainYearMonth in Chinese calendar with ISO reference date 2021-01-13
  // According to current spec text: Throws
```

<!--
    Trying to combine a day in Chinese calendar space with a month and year in ISO calendar space
-->

---

<!-- _footer: ✅ spec text ✅ tests -->

### DST bug in Duration comparison (PR [#2026](https://github.com/tc39/proposal-temporal/pull/2026))

- Incorrect comparison with time zone offset shifts
- Due to not calculating offset shift at start of day

```js
// Note: 2020-11-01 is a 25-hour day in America/Vancouver time zone
relativeTo = Temporal.ZonedDateTime.from('2019-11-01T00:00-07:00[America/Vancouver]');
d = Temporal.Duration.from({ years: 1, days: 1 })

Temporal.Duration.compare(d, { years: 1, hours: 25 }, { relativeTo })
  // Correct answer: 0
  // According to current spec text: -1
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Remove non-roundtrippable serialization ([#2035](https://github.com/tc39/proposal-temporal/pull/2035))

- `toString()` on PlainYearMonth and PlainMonthDay with ISO calendar returned a string that wasn't valid as input to `from()`
- When including the calendar, the ISO reference year / day should always be included as well

```js
new Temporal.PlainYearMonth(2022, 3).toString({ calendarName: 'always' })
  // Intended: '2022-03-01[u-ca=iso8601]' (includes ISO reference day)
  // According to current spec text: '2022-03[u-ca=iso8601]'
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### PMD not handled in DTF.formatRange (PR [#2043](https://github.com/tc39/proposal-temporal/pull/2043))

- Omission such that Intl.DateTimeFormat.formatRange() didn't properly check the types of its arguments when one was a Temporal.PlainMonthDay

```js
dtf = new Intl.DateTimeFormat('en', {calendar: 'iso8601'});
aMonthDay = new Temporal.PlainMonthDay(3, 28);
notAMonthDay = Temporal.Now.instant();
dtf.formatRange(aMonthDay, notAMonthDay)
  // Intended: throws TypeError
  // According to current spec text: some bogus result
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Mistake in grammar of `Etc/GMT±N` time zone names (PR [#2050](https://github.com/tc39/proposal-temporal/pull/2050))

- Special `Etc/GMT` time zones not correctly parsed

```js
Temporal.TimeZone.from('2000-01-01T00:00-07:00[Etc/GMT+7]').id
// Intended: "Etc/GMT+7"
// Actual, according to current spec text: "-07:00"
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Normative typo 😱 [PR #2000](https://github.com/tc39/proposal-temporal/pull/2000)

- Missing "not" in an if-condition

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented

---

# Sneak peek for next plenary

Three major pieces of implementor feedback remaining to address:

- Investigate optimizing the built-in calendar case (issue [#1808](https://github.com/tc39/proposal-temporal/issues/1808))
- Integrate Temporal.Calendar and Temporal.TimeZone into Intl.DateTimeFormat options (issue [#2005](https://github.com/tc39/proposal-temporal/issues/2005))
- Investigate removing [[Calendar]] slot from PlainTime (issue [#1588](https://github.com/tc39/proposal-temporal/issues/1588))

<!--
    I hope to present all of these in June, probably along with a couple of tweaks. Follow along with the issues if you are interested in these topics.
-->
