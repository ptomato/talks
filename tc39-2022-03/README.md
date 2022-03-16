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

---

<!-- _class: invert lead -->

# Adjustments

---

<!-- _footer: ✅ spec text ❌ tests -->

### Mathematical values in Duration (PR [#2094](https://github.com/tc39/proposal-temporal/pull/2094))

- General principle: internal slots should store MVs
  - Avoids subtle bugs
- Implementor feedback:
  - Values of `Temporal.Duration` are unbounded integers
  - Storing 10 BigInts far less performant than 10 Numbers

---

### Mathematical values in Duration (2)

- Note that Duration fields can now be the _intersection_ of:
  - mathematical integers
  - Number values
- Specifically, _not_ NaN, ±∞, and −0
- This was implicit before, and [sloppily defined](https://github.com/tc39/proposal-temporal/issues/1715). Now it is explicit.

---

### Mathematical values in Duration (3)

```js
const pos = Temporal.Duration.from({ months: 1 });
const neg = d1.negated();
const blank = new Temporal.Duration();
// Before:
Object.is(pos.years, neg.years) // => false
Object.is(blank.years, blank.negated().years) // => false
// After:
Object.is(pos.years, neg.years) // => true
Object.is(blank.years, blank.negated().years) // => true
```

---

<!-- _footer: ✅ spec text ✅ tests -->

### Remove non-roundtrippable serialization format (PR [#2035](https://github.com/tc39/proposal-temporal/pull/2035))

---

<!-- _footer: ✅ spec text ✅ tests -->

### Consistent default options (PR [#2028](https://github.com/tc39/proposal-temporal/pull/2028))

- When Temporal invokes a calendar operation with the default options:
  - Sometimes passed `undefined` as the options argument
  - Sometimes passed `Object.create(null)` as the options argument
- This should be consistent, because it is observable in userland calendars

---

<!-- _footer: ✅ spec text ❌ tests -->

### Consistent expanded-year output (PR [#2090](https://github.com/tc39/proposal-temporal/pull/2090))

DRAFT, not decided yet

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: ✅ spec text ✅ tests -->

### Invalid sign in PlainYearMonth.subtract (PR [#2002](https://github.com/tc39/proposal-temporal/pull/2002))

---

<!-- _footer: ✅ spec text ❌ tests -->

### Faulty mixing of calendar domains in PlainYearMonth arithmetic (PR [#2003](https://github.com/tc39/proposal-temporal/pull/2003))

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

### PlainMonthDay not handled in Intl formatRange (PR [#2043](https://github.com/tc39/proposal-temporal/pull/2043))

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

### Typos that were normative 😱

- Fix algorithms that don't work as described in the current spec text due to typos
- List of pull requests:
  - [PR #2000](https://github.com/tc39/proposal-temporal/pull/2000) ✅ spec text ✅ tests

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
