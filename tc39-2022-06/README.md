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
_footer: DRAFT
-->

# ⌚ **Temporal**

**(your name here?)**
Igalia, in partnership with Bloomberg  
TC39 June 2022

---

# Temporal update

- Similar to what we asked for consensus on last time!
- Asking for consensus on several minor normative changes:
  - Changes suggested by implementors ("adjustments")
  - Changes to make spec text accurately reflect the decisions made in the champions group ("bugs")
- No big discussions this time, just fixes

---

# Temporal update

- Will continue turning implementor feedback into presentations like this as bandwidth allows
- Number of normative PRs per plenary meeting still decreasing 📉

---

# IETF update

- (fill out this slide)

---

<!-- _class: invert lead -->

# Adjustments

---

<!-- _footer: Tests: not possible? -->

### Non-402 impls with calendars (PR [#2193](https://github.com/tc39/proposal-temporal/pull/2193))

- Currently, an implementation that does _not_ have `Intl`, is constrained to support only the ISO 8601 calendar.
- Remove this constraint
- But, require that any supported calendars are from the set supported by `Intl`. An implementation becoming 402-capable shouldn't _remove_ functionality

---

<!-- _footer: Tests: ❌ -->

### Order of operations in dateFromFields (PR [#2203](https://github.com/tc39/proposal-temporal/pull/2203))

- Changes the order of thrown exceptions in a really esoteric case
- Why do we care about this? It came up in the reviews for Stage 3 but Philip didn't realize it would be observable
- Makes the spec text a bit nicer
- In case you care about it:

```js
calendar.dateFromFields({ get year() { throw "not called"; } })
// Before: throws "not called"
// After: throws TypeError due to missing `day` property
```

---

<!-- _class: invert lead -->

# Bugs

---

<!-- _footer: Tests: ❌ -->

### Remove erroneous range check (PR [#2189](https://github.com/tc39/proposal-temporal/pull/2189))

- Faulty range check when parsing Instant strings with UTC offsets
- Made some valid strings invalid:

```js
Temporal.Instant.from('-271821-04-19T23:00-01:00').epochSeconds
// Current spec text: throws
// Intended: -86400e8
Temporal.Instant.from('+275760-09-13T01:00+01:00').epochSeconds
// Current spec text: throws
// Intended: 86400e8
```

---

<!-- _footer: Tests: ❌ -->

### Fix mistake in time zone name grammar (PR [#2200](https://github.com/tc39/proposal-temporal/pull/2200))

- Time zone strings with UTC offset time zones not in grammar
- More invalid strings that should've been valid:

```js
Temporal.TimeZone.from('+05:30')
// Current spec text: throws
// Intended: UTC offset time zone of +05:30
Temporal.TimeZone.from('2022-05-23T17:49-07:00[-07:00]')
// Current spec text: throws
// Intended: UTC offset time zone of -07:00
```

---

<!-- _footer: Tests: ✅ -->

### Fix mistake in exact time rounding (PR [#2210](https://github.com/tc39/proposal-temporal/pull/2210))

- Exact times should be rounded as if they are positive numbers, even if they are before the Unix epoch
- In other words, rounding is relative to the Big Bang, not 1970

```js
inst = Temporal.Instant.from('1969-12-31T23:30Z');
inst.round('hour')
// Current spec text: instant at 1969-12-31T23:00Z
// Intended: instant at 1970-01-01T00:00Z
```

---

<!-- _footer: Tests: ❌ -->

### Use null-proto objects in more places (PR [#2219](https://github.com/tc39/proposal-temporal/pull/2219))

- Last time, changed to use null-prototype objects in some places
- Do the same for other property bags where we create an object and then look up properties on it
- Also one place where we missed it last time in PlainYearMonth
- Guards against this kind of shenanigans:

```js
Temporal.PlainDate.from('2022-01-01').with({
  get year() {
    Object.prototype.day = 31;
    return 2023;
  },
});
```

---

<!-- _footer: Tests: ❌ -->

### Fix overflow case in Duration.total (PR #0000)

- Fix an edge case where the result of `.total()` overflows
- Could return ∞ or throw
- ∞ is what you'd get if you calculated it yourself

```js
Temporal.Duration.from({ microseconds: Number.MAX_VALUE }).total('nanoseconds')
// Current spec text: unclear what you should get; fails assertion
// After this change: Infinity
```

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented

---

# Sneak peek for next plenary

Three major pieces of implementor feedback remaining to address:

- Investigate optimizing the built-in calendar case (issue [#1808](https://github.com/tc39/proposal-temporal/issues/1808))
- Integrate Calendar and TimeZone into Intl.DTF options ([#2005](https://github.com/tc39/proposal-temporal/issues/2005))
- Investigate removing [[Calendar]] slot from PlainTime ([#1588](https://github.com/tc39/proposal-temporal/issues/1588))

Finally:

- Implement conclusions of IETF string standardization ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))
- Fix some other minor errors that you probably don't care about

<!--
    I hope to present all of these in July. Follow along with the issues if you are interested in these topics.
-->
