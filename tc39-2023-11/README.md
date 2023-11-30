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
  .twocol {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
  pre.mermaid { background: transparent; }
  pre.mermaid svg .background { fill: transparent; }
  table { font-size: 70%; }
---

<!-- _class: invert lead -->

<!-- mermaid.js -->
<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
mermaid.initialize({startOnLoad:true});
</script>

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 November 28, 2023

---

## Progress update

- Most approved normative changes merged with test262 coverage
- Working through some remaining review comments from Anba
- One normative change to propose today
  - (from usage experience in community)
- Follow the checklist in issue [#2628](https://github.com/tc39/proposal-temporal/issues/2628) for updates
- **Will give a loud signal when this checklist is complete**

---

## Test conformance as of October 2023

<div class="twocol">
<div>

- SpiderMonkey: 92%
- V8: 74%
- JavaScriptCore: 31%
- LibJS: 24%
- Boa: 13%

</div>
<div>
<pre class="mermaid">
%%{
  init: {
    "xyChart": {
      "xAxis": {
        "labelPadding": 10
      },
      "yAxis": {
        "labelPadding": 10
      }
    },
    "themeVariables": {
      "fontFamily": "Rubik",
      "xyChart": {
        "plotColorPalette": "#a40000"
      }
    }
  }
}%%
xychart-beta horizontal
  x-axis [SM, V8, JSC, LibJS, Boa]
  y-axis "% of test262 passing" 0 --> 100
  bar [92, 74, 31, 24, 13]
</pre>
</div>
</div>

---

## IETF standardization progress ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))

- **Complete.** This is no longer a blocker for shipping unflagged.

---

## Leap days in PYM/PMD.toPlainDate ([#2718](https://github.com/tc39/proposal-temporal/pull/2718))


```js
// What day of the week is your birthday in 2030?
const bd = Temporal.PlainMonthDay.from("12-15");
bd.toPlainDate({ year: 2030 }).dayOfWeek;
```

- Current: throws an exception if your birthday is February 29th
- Design principle: "no data-driven exceptions"
- Instead, `toPlainDate()` should return 2030-02-28
- Can still get the throwing behaviour more verbosely:

```js
const { monthCode, day } = Temporal.PlainMonthDay.from("02-29");
Temporal.PlainDate.from({ year: 2030, monthCode, day }, { overflow: "reject" }).dayOfWeek;
```

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

On normative PR [#2718](https://github.com/tc39/proposal-temporal/pull/2718)

---

# Proposed summary for notes

> The blocker on IETF standardization of the string format has been resolved.
> The champions will give a signal when outstanding changes have been merged, and at that point implementations will be encouraged to continue their work and ship unflagged when ready.
> A normative change to overflow behaviour in PlainYearMonth/PlainMonthDay.p.toPlainDate (PR [#2718](https://github.com/tc39/proposal-temporal/pull/2718)) reached consensus.

---

<!-- _class: lead -->

# Follow-up
### November 30, 2023

---

### Data-driven exceptions??!

- My bad for introducing jargon!
- Will avoid this term in the future
- Overview of what it means
- Then re-present the normative change

<!--
First of all, my apologies for confusing the issue on Tuesday by using the jargon "data-driven exceptions". I thought this was a more widely known term in TC39, but I guess we only use it in Temporal meetings.

We will take an action item to come up with a better name for this design principle to avoid confusion in the future.

I'll give an overview of how data processing is considered in Temporal, to clear up the confusion from yesterday, then at the end I'll present the normative change again.
-->

---

### Principle: No data-driven exceptions (1)

- Date/time data is full of edge cases
  * Buy a yearly subscription on February 29th
  * Send an automated email at 02:30 every day

<!--
The context behind "avoid data-driven exceptions" is that date/time data contains lots of weird edge cases. There are leap days, daylight saving time transitions, but also things like non-Gregorian calendars where there are leap months. 

So it will be common when writing Temporal code that developers will test with the normal cases but forget to test with those weird cases. If the weird cases throw exceptions, then code will work fine in the lab but will break when confronted with actual real-world data in production. Code that breaks when confronted with valid but unusual data is contrary to how most JS APIs work. So we tried to avoid that in Temporal.

This is not a unique problem to Temporal; it represents a real-world problem that all software needs to deal with. If you buy a yearly subscription for a product on Feb 29, you don't get to skip paying the bill until the next leap year! If your automated email system sends an email every day at 2:30AM, it shouldn't skip the day that DST starts. And so on.

Across a very wide range of these real-world use cases, what we observed is that these data-dependent edge cases are handled (by existing software) by defining default behaviors for how to resolve ambiguity automatically. For example: if a year doesn't have a leap day, then a result that would normally be Feb 29 is automatically constrained to Feb 28. If a time of 2:30 AM is requested on a day DST starts, then 3:30AM is used instead. Many of these defaults were inherited from existing APIs, including JS's own Date object.
-->

---

### Principle: No data-driven exceptions (2)

- _Data-dependent ambiguous cases should default to some reasonable behaviour instead of throwing_
  - Favour consistent behaviour, even if not ideal for all use cases
  - Favour real-world usage
* Can opt in to throwing

<!--
As a shorthand among the Temporal champions, "no data-driven exceptions" means that data-dependent ambiguous cases should default to some reasonable behavior instead of throwing. In retrospect this isn't a good name! But what it means is that if you write your code and it doesn't throw for a normal date and time, then it also shouldn't throw BY DEFAULT for a "weird" date or time.

I emphasize BY DEFAULT above because there are cases where developers do want to throw when confronted with weird data. If I want to send an email in the middle of the hour skipped by a DST transition, then it's fine to send that email later that day. But if I'm creating a baby's birth certificate and I for that time, I want to warn the user that the time is invalid! To handle these cases, Temporal always has a way to throw an exception, usually by using an option with the value reject.

Note that most other APIs (like legacy Date) don't offer an option to throw. They just silently fix up the weird data and return the result. So "no data driven exceptions" isn't really a new thing, it's just how all other date/time APIs work. What's new is that Temporal provides "yes data driven exceptions!" options to support the unusual cases where ambiguity is not acceptable.
-->

---

### What is valid data? (1)

- Temporal objects
  - Immutable
  - Always have valid data in internal slots
  - no `new Date(NaN)`

<!--
Temporal objects are immutable and always contain valid data. With the old JavaScript Date you can create a Date object that has NaN in its internal slot, representing an "invalid date". You can't do this with Temporal objects.

In this normative change we are talking about converting from one type of Temporal object to another, so this is the kind of "valid data" we are talking about. For completeness, I'll go over the other kinds data that we consider valid.
-->

---

### What is valid data? (2)

- Property bags
  - A validity domain is defined for each property
  - The property bag is valid if each property is valid individually
  - e.g. `hour`: integer in range [0, 23]
  - e.g. `month`: finite positive integer
  - Valid: `{ calendar: 'hebrew', year: 5784, month: 13, day: 1 }`
  - Valid: `{ calendar: customObj, year: 2000, month: 1, day: 9999 }`
  - Not valid: `{ year: 2000.5, month: 0, day: -1 }`

<!--
Also note that obviously bad data (like a negative hour or zeroth month) will always throw. These are not "weird data", they're just plain invalid. However, values like { month: 13 } might be valid in a non-Gregorian calendar, so positive out-of-range day and month values don't throw by default either. The general principle is that if an input could be valid in some day, some month, some year, some calendar, etc. then we we don't throw by default, even if it's not present in a particular day/month/year/calendar. If we can determine _without_ doing any calendar calculation that any individual value is invalid, like a negative day, then the property bag is not valid data and is not subject to clamping.
It's a messy principle (because dates and times and calendars and time zones are messy!) but it's consistent.
-->

---

### What is valid data? (3)

- Strings: ISO 8601 + IXDTF is clear about valid ranges
  - Valid: `02-29`
  - Valid: `2024-02-29`
  - Not valid: `00-00`, `12-32`, `2030-02-29`
- Ambiguity still possible with time zone transitions
  - Syntactically valid: `2023-03-12T02:30[America/Vancouver]`
  - Not valid: `2023-03-12T99:99[America/Vancouver]`

<!--
Finally, parsing strings must also adhere to the grammar defined in ISO 8601 and IXDTF (the new IETF RFC). If a string input doesn't adhere to those specs, we always throw to be compliant. The flexibility discussed above only applies to cases where we're interpreting number inputs, not string inputs.

Here are some examples of what's valid and what's not. The top one is a valid month-day string. The next one is a valid date string. The third line has some month-day and date strings that are 

ISO 8601 and IXDTF do not include time zone transitions in their definition of validity. That'd be impossible.
The string on the second-to-last line is a nonexistent time because it's in the middle of the hour when we set the clocks forward this spring in my time zone, but it's a valid string and when you convert it to a Temporal object, you can choose with an option whether to clamp it or throw.
The string on the last line is still not valid because 99:99 is just not a time that clocks can display.

-->

---

### APIs that could clamp or reject

<div class="twocol"  style="font-size: 55%">
<div>

#### Clamps by default, option to throw

- PlainDateTime + TimeZone → ZonedDateTime
  ```js
  plainDateTime.toZonedDateTime(timeZone, { disambiguation: 'reject' })
  ```
- Syntactically valid date-time-zone string → ZonedDateTime
  ```js
  Temporal.ZonedDateTime.from('2023-03-12T02:30[America/Vancouver]',
    { disambiguation: 'reject' })
  Temporal.ZonedDateTime.from('2023-03-12T02:30-09:00[America/Vancouver]',
    { offset: 'reject' })
  ```
- Valid date-time string + TimeZone → Instant
  ```js
  timeZone.getInstantFor('2023-02-13T02:30', { disambiguation: 'reject' })
  ```
- Valid property bag → any Temporal type (`from()`)
  ```js
  Temporal.PlainDate.from({ year: 2023, month: 11, day: 31 }, { overflow: 'reject' })
  ```
- Temporal type + {some properties} → Same type (`with()`)
  ```js
  plainDate.with({ day: 31 }, { overflow: 'reject' })
  ```

</div>
<div>

#### Always clamps

- PlainDate + PlainTime + TimeZone → ZonedDateTime (convenience methods)
  ```js
  plainDate.toZonedDateTime({ plainTime, timeZone })
  plainTime.toZonedDateTime({ plainDate, timeZone })
  ```
- Valid property bag → any Temporal type (convenience coercion)
  ```js
  plainDate.until({ year: 2023, month: 11, day: 31 })
  ```

#### Throws

- <span style="color:#a40000;">PlainYearMonth + {day} → PlainDate ⚠️</span>
  ```js
  plainYearMonth.toPlainDate({ day: 31 })
  ```
- <span style="color:#a40000;">PlainMonthDay + {year} → PlainDate ⚠️</span>
  ```js
  plainMonthDay.toPlainDate({ year: 2030 })
  ```

</div>
</div>

<!--
Here's an overview of all the ways that you can convert data in Temporal that could be invalid in the result domain.
This is a pretty short list because most conversions can't fail at all (at least not due to being invalid in the result domain like February 29th 2030), and so don't need to be clamped. An example of a conversion that can't fail is converting from a PlainDateTime to a PlainDate. Every valid PlainDateTime object can be converted to a valid PlainDate.

Most failable conversions clamp by default, but have an option to throw. On the left are examples of what circumstances this occurs in. (Sorry, it's a bit crammed in.)

The ones that don't have an option to throw are because we considered them a convenience conversion, and so they use the default option. The PlainDate + PlainTime + TimeZone to ZonedDateTime methods are just shorthand for first converting to a PlainDateTime and then a ZonedDateTime, so if you want to adjust the behaviour you can specify in which conversion you want the clamping or throwing to occur.
Likewise, any method that accepts a Temporal object also accepts a property bag, and this is a shorthand for calling `from()` on the property bag. If you want the throwing behaviour, you can specify it manually by 

Given this context, let's come back to the normative change. In the current Temporal spec, these red ones don't follow the same default behavior as other Temporal APIs, which is to constrain the output to a valid date if the desired date doesn't exist when the receiver is combined with the input.

Basically we want to empty the "Throws" category into the "Always clamps" category.
-->

---

### PR [#2718](https://github.com/tc39/proposal-temporal/pull/2718)

- Moves "Throws" items into "Always clamps" category
- Motivated by feedback from practitioners
- If we were still designing API at Stage 2, would move into "Clamps by default, option to throw" instead
- Will track adding the option as a possibility for a follow-on proposal

<!--
This is a spec bug that was discovered by a user, and we want to fix it so that the default behavior of all similar Temporal APIs are consistent.

If we didn't make this change, then common use cases like "what day of the week is my birthday next year?" could throw, which would be quite unexpected.

If this proposal were still stage 2, it'd probably be good to provide the option allowing you to choose the throwing behaviour. We don't want to do that at this time, but will track that as a possibility for a follow-on proposal.
-->

---

<!-- _class: invert lead -->

# Consensus?

<!--Can we get consensus for this normative change?-->
