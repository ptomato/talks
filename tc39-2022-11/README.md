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
TC39 November-December 2022

---

# Temporal progress update

- Implementation continues
- Number of concerns raised by implementors decreasing
- Brief progress update on remaining issues in this presentation
- A few normative PRs for consensus in this plenary

<!--
This update will be a bit different from the ones in previous meetings.
I have some normative PRs to present for consensus, as usual, but I'd also like to take a bit of extra time this time...
-->

---

<!--
_class: lead
-->

# Tour of remaining issues

<!--
    ...to take you on a tour of the remaining normative changes that we would potentially make to the proposal before Stage 4. This number is 
-->

---

## IETF standardization progress [#1450](https://github.com/tc39/proposal-temporal/issues/1450)

- IETF document has passed working-group last call
- Currently in IESG review; changes unlikely, but possible
- An official RFC number will remove the blocker from our side

<!--
    To recap, we are standardizing calendar and time zone annotation syntax in the IETF and agreed at stage 3 to block shipping until there is a standard we can normatively reference.
    The IETF chartered a working group which completed this document and it's now passed working-group last call.
    It's now undergoing IESG = Internet Engineering Steering Group review before publication.
-->

---

## Optimizability of built-in calendars and time zones [#1808](https://github.com/tc39/proposal-temporal/issues/1808) (1/4)

**Summary:** Normative changes that make it easier for implementations to determine when it's OK to avoid creating a `Temporal.Calendar` or `Temporal.TimeZone` object and just use the built-in behaviour.

---

## Optimizability [#1808](https://github.com/tc39/proposal-temporal/issues/1808) (2/4)

- In previous presentations I listed [#1808](https://github.com/tc39/proposal-temporal/issues/1808) as one of the "big" blockers
- No longer; we've now arrived at a solution
- Normative change not ready yet; still discussing a few edge cases
- Neatly solves [#1588](https://github.com/tc39/proposal-temporal/issues/1588) and [#2104](https://github.com/tc39/proposal-temporal/issues/2104) as well

---

## Optimizability [#1808](https://github.com/tc39/proposal-temporal/issues/1808) (3/4)

```js
// Basically the problem is this:
const d = Temporal.PlainDate.from({ year: 2022, month: 11, day: 30, calendar: 'gregory' });
d.calendar  // ⇒ Property value is a Temporal.Calendar instance
// Every time some calendar calculation is done, e.g...
const d2 = d.add({ months: 1 });
// ...the calendar methods have to be looked up on that instance and called
// even though we just want the behaviour of the built-in calendar

// But on the other hand, in the case of a custom calendar object...
const d3 = d.withCalendar(myCustomCalendar);
const d4 = d3.add({ months: 1 });
// ... we do actually want those lookups to happen
```

---

## Optimizability [#1808](https://github.com/tc39/proposal-temporal/issues/1808) (4/4)

Proposed solution:
- Two paths depending on whether object's [[Calendar]] internal slot contains a string (built-in calendar) or an object (custom calendar)
- `calendar` property replaced by string-valued `calendarId` property
- Separate API to get a `Temporal.Calendar` instance, which will not be the same instance each time in the built-in case
- Same thing for `Temporal.TimeZone`, without loss of generality

More details in the issue thread; hope to present spec text next time

---

## More integration of TimeZone and Calendar with Intl [#2005](https://github.com/tc39/proposal-temporal/issues/2005)

- TG2 has expressed a wish for Temporal to include closer integration of `Temporal.TimeZone` and `Temporal.Calendar` in ECMA-402
- Nontrivial amount of work, but likely "mechanical"
- Not likely to require discussions
- Work in progress

---

## Extra calendar fields in ECMA-262 [#2169](https://github.com/tc39/proposal-temporal/issues/2169)

- Disagreement about what 402 may add to objects defined in 262
- Agenda item scheduled to discuss during this plenary
- Once resolved, fixing the Temporal spec text either way is trivial

---

## Domain of Duration arithmetic [#2195](https://github.com/tc39/proposal-temporal/issues/2195) (1/2)

- In a previous update we changed `Temporal.Duration`'s internal slots to store ℝ(𝔽(_value_)) "float64-representable integers" in response to implementor concern about Duration storage size
- Concerns still exist about the domain in which arithmetic is performed with these f64-representable integers: ℝ or 𝔽?
- Champions feel that the status quo is the "least worst" solution
- Open to alternatives

---

## Domain of Duration arithmetic [#2195](https://github.com/tc39/proposal-temporal/issues/2195) (2/2)

Plan:
- Prepare a brief motivating current choices and detailing already-considered alternatives
- Invite new alternatives from implementors
- Fallback is to retain the status quo

---

## Calendar in PlainTime conversion [#2221](https://github.com/tc39/proposal-temporal/issues/2221)

- `Temporal.PlainTime` has no calendar but we want to keep the option open to introduce one in the future
```js
const dt = Temporal.Now.zonedDateTime({ calendar: 'hebrew' });
dt.toPlainTime()  // Currently, drops the calendar. What about in the future?
```

- Low in severity; use cases for PlainTime calendars not clear
- Also small in scope, but...
- Must find how to keep web-compatible without affecting DX now
- Alternatively, decide that PlainTime will never have a calendar

---

## Unnecessary calls to user code [#2247](https://github.com/tc39/proposal-temporal/issues/2247) & [#2289](https://github.com/tc39/proposal-temporal/issues/2289)

- Implementation feedback has turned up several cases of calling redundantly into user code
- Until now, we've been fixing these case-by-case
- Best solved by auditing the whole proposal
- Work in progress

---

## MakeDay/MakeDate/TimeFromYear out of range [#2315](https://github.com/tc39/proposal-temporal/issues/2315)

- [MakeDay](https://tc39.es/ecma262/#sec-makeday), [MakeDate](https://tc39.es/ecma262/#sec-makedate), and [TimeFromYear](https://tc39.es/ecma262/#eqn-TimeFromYear) can overflow to infinity
- This is even already a problem in ECMA-262 (see [ecma262#1087](https://github.com/tc39/ecma262/issues/1087))
- At the least, requires an audit of call sites of these operations
- Possible solution is to make these calculate in ℝ
- Another is to avoid reusing them in Temporal
- Some work and discussions in progress

---

# When is it ready to ship?

- "Ready to ship" definition being discussed in this plenary
- Need the IETF to publish an RFC, so they can complete their review
- Probably makes sense to ship after:
  - Landing the "optimizability" change ([#1808](https://github.com/tc39/proposal-temporal/issues/1808))
  - Changes to Duration arithmetic domain, if any ([#2195](https://github.com/tc39/proposal-temporal/issues/2195))
- All other potential normative changes are minor
- Assuming no more big ones are discovered by implementations

<!--
    Aside from those two, or one, as the case may be, the champions feel that all the other normative changes that I described are at the level of "bugfix" and unlikely to break the web if changed after Temporal is exposed to the web.
-->

---

# What is needed for Stage 4?

- test262 nearing completion
- Implementations known to be near completion: GraalJS, LibJS, V8
- Implementations in progress: Boa, JSC, SpiderMonkey
- Pull request into ECMA-262 would be giant; stage it in sections?

<!--
    test262 coverage is probably sufficient if including the tests in the staging section. But the staging section doesn't count towards stage 4 requirements, so this needs some more work, but it is bounded.

    In order to avoid the problem that we had at Stage 3 where I had the impression that the reviewers and editors found the proposal too large to review each part of it with full attention, I'm wondering if we can stage the spec text in sections, where the editors can review each section once the previous section is agreed upon. We can discuss the process more closer to the time, but it's something we're already thinking about.
-->

---

<!-- _class: lead -->

# Questions
### (about this section)

---

<!-- _class: lead -->

# Normative PRs

---

## Handling of non-numeric inputs [#2438](https://github.com/tc39/proposal-temporal/pull/2438)

```js
new Temporal.Duration(NaN); new Temporal.PlainTime(NaN);
new Temporal.Duration("foo"); new Temporal.PlainTime("foo");
new Temporal.Duration(/regex/); new Temporal.PlainTime(/regex/);
new Temporal.Duration(Temporal); new Temporal.PlainTime(Temporal);
// Before: zero-length duration, midnight time 😱
// After: RangeError
```

- Align better with Web IDL's conversions for `double` and `[EnforceRange] long long` types
- Positive support for this change in informal poll a few months ago

---

## Order of observable property access [#2437](https://github.com/tc39/proposal-temporal/pull/2437)

- In general, access properties of property bags in alphabetical order
  - More detailed design principle described in the [issue]((https://github.com/tc39/proposal-temporal/issues/2254))
- Align with `Intl.DurationFormat` proposal
  - See corresponding [normative change](https://github.com/tc39/proposal-intl-duration-format/pull/129) there

---

## Fast-fail calendar⇔time zone conversion [#2433](https://github.com/tc39/proposal-temporal/pull/2433)

- Throw when passing an object with `Temporal.Calendar` brand to an API that expects a time zone object, and vice versa
- Mainly to prevent falling into this trap:

```js
// (Note: arguments 2 and 3 are in the wrong order here)
new Temporal.ZonedDateTime(0n, calendar, timeZone);
// Before: silently create an object that fails later operations
// After: RangeError
```

---

## IXDTF annotations and UTC offsets [#2428](https://github.com/tc39/proposal-temporal/pull/2428)

- Leftover from the changes presented last time regarding IXDTF annotations format
  - (because we had some open questions about UTC offset syntax in ISO 8601)
- Allow MM-DD[annotations] and YYYY-MM[annotations]
- Disallow YYYY-MM-DDZ and YYYY-MM-DD±UU:UU

---

## `yearOfWeek` API [#2425](https://github.com/tc39/proposal-temporal/pull/2425)

- Problem identified in the wild: 202**2**-01-01 is day 6 of ISO week 52 of 202**1**
```js
const d = Temporal.PlainDate.from('2022-01-01');
console.log(`${d} is day ${d.dayOfWeek} of ISO week ${d.weekOfYear} of ${d.year}`);
  // → "2022-01-01 is day 6 of ISO week 52 of 2022" (wrong!)

// New API:
console.log(`${d} is day ${d.dayOfWeek} of ISO week ${d.weekOfYear} of ${d.yearOfWeek}`);
  // → "2022-01-01 is day 6 of ISO week 52 of 2021"
```

---

## Don't run user code in `.id` getter [#2411](https://github.com/tc39/proposal-temporal/pull/2411)

- Fix unintentional leftover from an earlier change
```js
const c = Temporal.Calendar.from('iso8601');
c.id  // => "iso8601" (unchanged)
  // Before: [[Get]] and call @@toPrimitive, and possibly toString and valueOf on c
  // After: No user code calls

const t = Temporal.TimeZone.from('UTC');
t.id  // => "UTC" (unchanged)
  // Same, no more user code calls
```

---

## Fix rounding in epoch time getters [#2424](https://github.com/tc39/proposal-temporal/pull/2424)

- Instant and ZonedDateTime's `.epochSeconds`, `.epochMilliseconds`, `.epochMicroseconds` getters round in the wrong direction pre-1970
- For epoch times, "rounding up" is towards the end of time, "down" is towards the Big Bang

```js
const i = new Temporal.Instant(-999999n)  // → 1969-12-31T23:59:59.999000001Z
i.round({smallestUnit:'milliseconds', roundingMode:'trunc'})  // → 1969-12-31T23:59:59.999Z
Temporal.Instant.fromEpochMilliseconds(i.epochMilliseconds)
  // Before: 1970-01-01T00:00:00Z
  // After: 1969-12-31T23:59:59.999Z (should match the round() result)
```

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented
