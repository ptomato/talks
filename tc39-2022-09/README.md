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
_footer: DRAFT
-->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 September 2022

---

# Another Temporal update

- We continue to adjust the proposal based on implementor feedback and bug reports
- Fixes continue to become more and more obscure edge cases
- Acknowledge new concerns about mathematical values in Temporal.Duration and are considering how to address
- Good news from the IETF

---

# IETF update: "IXDTF"

- Draft syntax for calendar annotations has reached consensus in IETF SEDATE working group
- Tentatively named **IXDTF** (Internet Extended Date-Time Format)
- Only administrative blockers remain
  * (standards organizations, amirite?)
  * We will continue to push, and alert the plenary when this happens

---

<!-- _class: invert lead -->

# Adjustments
to the proposal

---

### Implement conclusions of IXDTF (PR [#2397](https://github.com/tc39/proposal-temporal/pull/2397))

- IXDTF contains a specification of our proposed calendar annotation, as well as the de-facto standard time zone annotation introduced by Java
- Also allows other (as yet unknown) annotations: 2022-09-13T10:00[Asia/Tokyo]**<span style="color:#4e9a06">\[magic=happens]</span>**
- Introduces a "critical flag" indicating that an annotation must be respected: 2022-09-13T10:00[**<span style="color:#4e9a06">!</span>**<!---->Asia/Tokyo]

---

### Implement conclusions of IXDTF (2)

- Temporal only recognizes time zone and calendar annotations, and ignores unknown ones unless they are marked critical
- Time zone and calendar annotations were already treated as critical anyway, so the critical flag has no effect in Temporal
  - i.e. `2022-09-13[u-ca=hebrew]` would already never return a date in the Gregorian calendar
  - However, flag may have meaning to other consumers of ISO strings

---

### Implement conclusions of IXDTF (3)

- For interoperability with other consumers, add a new value `"critical"` to the `timeZoneName` and `calendarName` options in the various `toString()` functions:

```js
Temporal.Now.zonedDateTimeISO()
  .toString({ calendarName: 'critical', timeZoneName: 'critical' });
// returns:
// "2022-09-12T19:00-07:00[!America/Vancouver][!u-ca=iso8601]"
```

---

### ISO 8601 grammar (PRs [#2394](https://github.com/tc39/proposal-temporal/pull/2394), [#2395](https://github.com/tc39/proposal-temporal/pull/2395), [#2398](https://github.com/tc39/proposal-temporal/pull/2398))

Several other tweaks:
- A string of 8 decimal digits is a valid (but not currently existing) calendar name and also valid YYYYMMDD syntax.
  - <small>Does `Temporal.Calendar.from('20220912')` throw due to no calendar with that name existing, or return the date 2022-09-12's ISO 8601 calendar? We picked the latter</small>
- Make time zone and calendar names case-insensitive, as they currently are in ECMA-402

---

### ISO 8601 grammar (2)

Several other tweaks:
- Allow annotations after MM-DD and YYYY-MM syntax
- Fix bug where interpreting a `relativeTo` string as a PlainDate would accept a `Z` UTC designator (which PlainDate does not)

---

### Tweaks to observable operations (PR [#2377](https://github.com/tc39/proposal-temporal/pull/2377))

- Follow [conventions](https://github.com/tc39/how-we-work/pull/119) for order of observable operations in calendars' `dateFromFields`, `yearMonthFromFields` , `monthDayFromFields` methods
- Affects nothing unless you are going out of your way to spy on operations with Proxy traps or suchlike

---

### Validation of returns from user code (PR [#2387](https://github.com/tc39/proposal-temporal/pull/2387))

- When calling a calendar's `dateAdd()` implemented in user code, throw if we detect an inconsistent result
  - <small>(technical details: in our abstract operation [NanosecondsToDays](https://tc39.es/proposal-temporal/#sec-temporal-nanosecondstodays), validate that the results from [AddZonedDateTime](https://tc39.es/proposal-temporal/#sec-temporal-addzoneddatetime) are self-consistent)</small>

---

### Remove unnecessary property access (PR [#2392](https://github.com/tc39/proposal-temporal/pull/2392))

- Skip an unnecessary observable HasProperty operation when calling `from()` on an object that already has the correct internal slot

```js
const calendarInstance = new Temporal.Calendar('iso8601');
Temporal.Calendar.from(calendarInstance);
// would previously do HasProperty(calendarInstance, 'calendar')

const timeZoneInstance = new Temporal.TimeZone('UTC');
Temporal.TimeZone.from(timeZoneInstance);
// would previously do HasProperty(timeZoneInstance, 'timeZone')
```

---

### Fix rounding in Instant arithmetic (PR #NNNN)

- Latent bug in the spec text, exposed by the adoption of NumberFormat V3's rounding modes
- Result of `Temporal.Instant.prototype.until()` was rounded as if it were a number of epoch nanoseconds, not a Duration
- Ditto for `Temporal.Instant.prototype.since()`

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
