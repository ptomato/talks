---
marp: true
theme: gaia
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.14.0/css/all.min.css');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  @import url('https://fonts.googleapis.com/css2?family=Rubik:ital,wght@0,400;0,500;0,700;1,400;1,700&display=swap');
  code { font-family: Hack; }
  section { font-family: Rubik, sans-serif; letter-spacing: 0; }
  section.lead.invert { text-shadow: 0 0 10px black, 0 0 20px black, 0 0 30px black; }
  marp-pre code, marp-pre { background-color: #042029; }
  .hljs-string { color: #8ae234; }
  .hljs-number { color: #729fcf; }
  .hljs-params { color: #e9b96e; font-style: italic; }
  .hljs-built_in { color: #fce94f; font-weight: bold; }
  .hljs-keyword { color: #fcaf3e; font-weight: bold; }
  .hljs-attr { color: #e9b96e; }
  .hljs-variable { color: red; font-weight: bold; }
  /* .hljs-comment, .hljs-regexp, .hljs-symbol */
---

<!--
_class: invert lead
_footer: Photo by <a href="https://unsplash.com/@cblanco_31">Carlos Blanco</a> on <a href="https://unsplash.com/s/photos/gastown-clock">Unsplash</a>
-->

# **Temporal**

## Modern dates and times in JavaScript

**Philip Chimento** • <i class="fab fa-github"></i> ptomato • <i class="fab fa-twitter"></i> @therealptomato  
Igalia, in partnership with Bloomberg  

![bg](gastown-clock.jpg)

<!--
My name is Philip Chimento. I work as a JavaScript engine developer at Igalia.

I'm speaking today about the Temporal proposal, which adds modern date and time handling to JavaScript. I'm part of a group working on this proposal, and my participation is part of a partnership between Igalia and Bloomberg.

In this presentation, I'll give a tour through the API, and show you what you can do with Temporal by means of an easy, medium, and complicated programming task.
-->

---

# What is Temporal?

- A proposal for enhancing JavaScript
- A new, built-in, date-time library
- Strongly typed
- Immutable
- Strong support for internationalization

<!--
Temporal is an API that has been proposed to become part of JavaScript. It'll be available in browsers soon. It will add a built-in library for dates and times to JavaScript, like many other programming languages already have.

Temporal is built-in to the browser or JS engine. That's important because many developers include a dependency on Moment or some similar library in their app to achieve the same thing that you could with Temporal. But, depending on how much locale data you include, this may add a payload of anywhere from a dozen to two hundred kilobytes to the app.

Temporal is strongly typed. It means that for different kinds of data, such as a calendar date with or without an associated time, there are different types to represent them, instead of one type fits all.

Temporal objects are immutable. That means that you don't have to worry about code that you don't own, modifying your objects without your knowledge.

Temporal is designed to work together with JavaScript's internationalization facilities, and provides things that a lot of other libraries don't, such as non-Gregorian calendars.
-->

---

<!-- _class: lead gaia -->

# :thinking: But we already have `Date`!

<!--
A fair question for any proposal is, do we really need this? JavaScript already has the global Date object. It's not perfect, but it's done OK for all these years.
-->

---

# `Date` is _not_ a good API

Source: [Maggie Johnson-Pint](https://maggiepint.com/2017/04/09/fixing-javascript-date-getting-started/)

1. No support for time zones other than the user’s local time and UTC
2. Parser behavior so unreliable it is unusable
3. `Date` object is mutable
4. DST behavior is unpredictable
5. Computation APIs are unwieldy
6. No support for non-Gregorian calendars

<!--
I disagree! We do actually have a good list of Date's deficiencies. This here is a list of problems with Date that were identified way back at the beginning, that Temporal aims to solve. You can read about them in the linked blog post by Maggie Johnson-Pint, one of the Moment developers and also one of the original architects of Temporal. She lists them "in order of relative disgust." I won't go into all of these. Some of them, such as the unwieldy APIs, could be fixed. Others, like the mutability, cannot be fixed without breaking the web, because so much code out there already relies on the existing behaviour.

JavaScript's Date was based on a Date class from Java, which Java deprecated in 1997 and replaced by something better. A quarter century later, we are doing the same thing in JavaScript.
-->

---

<!--
_class: lead invert
_footer: Photo by <a href="https://unsplash.com/@pratiksha_mohanty">Pratiksha Mohanty</a> on <a href="https://unsplash.com/s/photos/spices">Unsplash</a>
-->

# The selection of types

![bg](spices.jpg)

<!--
I mentioned strong typing. For the next few slides I'm going to give you a tour through these types that Temporal introduces.
-->

---

<!-- _footer: \\*disregarding leap seconds, though -->

# `Temporal.Instant`

- An exact moment in time\*
- No time zone, no daylight saving
- No calendar
- Data model: elapsed nanoseconds since midnight UTC, Jan. 1, 1970

<!--
The first type you should know about is Instant. It represents what we call an exact time, an instantaneous point on the timeline, with nanosecond resolution. There is no calendar, so no months, weeks, years, or days; no time zone, so no daylight saving adjustments. Just purely an ever-increasing number.
-->

---

# Plain types

- ‘Wall’ time
- No time zone, no daylight saving
- Several types in this family with progressively less information

<div class="mermaid">
%%{init: {'themeVariables': {'fontFamily': 'Rubik', 'fontSize': 24}}}%%
graph LR
  PDT(PlainDateTime) ==> PD(PlainDate)
  PDT ==> PT(PlainTime)
  PD ==> PYM(PlainYearMonth)
  PD ==> PMD(PlainMonthDay)
</div>

<!-- mermaid.js -->
<script src="https://unpkg.com/mermaid@8.10.2/dist/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true});</script>

<!--
Next we have a family of types called the "Plain" types. These represent a date on your wall calendar and a time on your wall clock, which we call "wall" time for short.
They represent your local date and time independent of time zone, and so there are no daylight saving adjustments.
But unlike Instant, they are not exact moments in time. A wall time like "November 18th 2024 at two-thirty" occurs at several different exact times around the world.
-->

---

# Plain types

- Represent the information you have
- Avoid buggy pattern of filling in 0 or UTC for missing info
- Do appropriate calculations based on type

<!--
Why do we have this family of types with progressively less information?
It's so that you can correctly represent the information that you have. For example, with the old Date, if you wanted to represent a date without a time, you might use midnight in the user's local time zone. But there are actually days in certain time zones where midnight is skipped due to daylight saving time. This can cause hard-to-track-down bugs. For example, midnight on November 4th, 2018, didn't exist in Brazil!
-->

---

# Plain types

- `Temporal.PlainDate`: “The conference session on November 18th, 2024”
- `Temporal.PlainTime`: “The break is at 12:05”
- `Temporal.PlainYearMonth`: “The November 2024 board meeting”
- `Temporal.PlainMonthDay`: “My birthday is December 15th”

<!--
In Temporal, we have these types that don't carry all the information:

PlainDate is a day without a specific clock time. It's a very common use case!

PlainTime is a clock time, not on any specific day.

PlainYearMonth is a month without a specific day. You could think of using it to refer to an event that happens occasionally, like I have there "the September 2021 board meeting". It also corresponds to HTML input type month.

PlainMonthDay is a calendar date, but without a specific year. You could think of it as referring to a birthday or anniversary.
-->

---

# `Temporal.ZonedDateTime`

- Like `Temporal.Instant`, an exact time
- But also with a calendar and time zone
- Correctly accounts for the time zone's daylight saving rules
- “The conference session is on Monday, November 18th, 2024, at 11 AM, US Eastern Standard Time”

<!--
Completing the family of types is another exact time type, ZonedDateTime. Just like Instant, this type represents an exact moment in time. But it is also coupled to a location with time zone rules, because it includes a time zone. The time zone means that this type does account for daylight saving time changes (and other changes in the time zone.) It also has a calendar, which means that unlike Instant it has a year, month, and day. So it's an exact time that can conveniently give you a wall time as well.

We say that ZonedDateTime represents a calendar event that happened, or will happen, at a place on Earth.
-->

---

# `Temporal.Duration`

- returned from other types' `since()` and `until()` methods
- passed to their `add()` and `subtract()` methods

<!--
There is one more type that's part of Temporal and doesn't represent a date or a time. Duration is used in arithmetic with the other types, and has some methods of its own for rounding and converting between units.
-->

---

# Type relationships

<svg viewBox="113 540 4042 1484" width="100%" height="100%" fill="none" stroke-width="3.8" style="background-color: transparent;" xmlns="http://www.w3.org/2000/svg">
  <style>
    text {
      font-family: 'Rubik', sans-serif;
      font-size: 70px;
      font-weight: 500;
    }
  </style>
  <rect x="116" y="544" width="841" height="1350" fill="#FFFCE4" stroke="#B7B7B7" rx="20" />
  <text x="218" y="658" style="font-size: 80px; fill: #737373; white-space: pre;" >Exact Time Types</text>
  <rect x="1055" y="544" width="2445" height="1350" fill="#FFFCE4" stroke="#B7B7B7" rx="20" />
  <path stroke="#595959" d="m 1755,1374 h 186 V 929 h 186"/>
  <text x="1568" y="658" style="font-size: 80px; fill: #737373; white-space: pre;" >Calendar Date / Wall-Clock Time Types</text>
  <path stroke="#595959" d="M 530 1280 V 1020"/>
  <path stroke="#595959" d="m -511,-966 v -138 l -1,-124 c -3,-121 1,-14 -1,-14"/>
  <rect x="250" y="1250" fill="#F0EDF7" width="560" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="430" y="1370">Instant</text>
  <path stroke="#595959" d="m 1755,1374 h 186 v 221 h 186"/>
  <rect x="1278" y="1280" fill="#F0EDF7" width="560" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="1318" y="1390">PlainDateTime</text>
  <path stroke="#595959" d="m 2515,928 h 171 V 824 h 170"/>
  <rect x="2768" y="730" fill="#F0EDF7" width="590" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="2823" y="845">PlainMonthDay</text>
  <path stroke="#595959" d="m 2515,928 h 171 v 220 h 170"/>
  <rect x="2048" y="840" fill="#F0EDF7" width="560" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="2173" y="955">PlainDate</text>
  <rect x="2768" y="1060" fill="#F0EDF7" width="590" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="2808" y="1175">PlainYearMonth</text>
  <rect x="2048" y="1500" fill="#F0EDF7" width="560" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="2163" y="1615">PlainTime</text>
  <rect x="250" y="835" fill="#F0EDF7" width="1590" height="180" stroke="#B4A7D6" rx="35"/>
  <text style="fill: #000; white-space: pre;" x="750" y="950">ZonedDateTime</text>
  <text x="1148" y="1840" style="font-size: 72px; fill: #737373; white-space: pre;" >These have a calendar (except PlainTime) &amp; know wall-clock time</text>
  <path d="m 156,1648 h 705 v 152 H 156 Z"/>
  <text x="200" y="1740" style="font-size: 80px; fill: #737373; white-space: pre;" >These types know</text>
  <text x="215" y="1840" style="font-size: 80px; fill: #737373; white-space: pre;" >time since Epoch</text>
  <path stroke="#595959" d="M 1560,1275 V 1015"/>
</svg>

<!--
This diagram is taken from the Temporal documentation. It shows the relationships between the types. You can see there are the two types on the left that know the exact time, and the other types on the right that know the wall time. ZonedDateTime spans both categories because it knows both the exact time and the wall time.
-->

---

<!--
_class: invert lead
_footer: Photo by <a href="https://unsplash.com/@lordarcadius">Vipul Jha</a> on <a href="https://unsplash.com/s/photos/code">Unsplash</a>
-->

# Show me the code!

![bg](keyboard.jpg)

<!--
Now that you've been introduced to the cast of characters in Temporal, it's time to show how to accomplish a programming task. I've picked three tasks to walk through: an easy one, a medium one, and a complicated one.
-->

---

<!-- _class: lead -->

# An easy one

Get a Unix timestamp in ms

<!--
The easy task is to get the current time as a Unix timestamp in milliseconds. This is the #1 top voted question for the old JavaScript Date object on Stack Overflow, so naturally we'll want to be able to do the same thing in Temporal.

First we consider what type we have to use. A timestamp represents an exact time without regard to time zones, so we use Instant.

The next thing, maybe a meta-thing, to consider, is do we really want a timestamp in milliseconds or will the Instant object itself work just as well? Going forward, as Temporal gets wider adoption, we expect that numerical timestamps are going to be mainly necessary for interoperation, and not so much for JavaScript applications, where Instant will be used. But for the sake of this example let's say that we do need a numerical timestamp.

We know what type we need, and next we need to know how to fill it with the current time.
-->

---

# `Temporal.Now`

- A namespace with functions that give you Temporal objects representing the _current_ date, time, or time zone
- "ISO" means "using the ISO 8601 calendar"

```js
Temporal.Now.instant()
Temporal.Now.timeZoneId()
Temporal.Now.zonedDateTimeISO()
Temporal.Now.plainDateTimeISO()
Temporal.Now.plainDateISO()
Temporal.Now.plainTimeISO()
```

<!--
We do this with the functions in the "Now" namespace. Any of these functions will give you the current date, time, and/or time zone, as one of the Temporal types. The "ISO" in the name means that the date is in the standard ISO calendar. I'll talk more about calendars later.

Since we need the current time as an Instant, we'll use the top one from this list.

Next we'll need to figure out how to get a number of milliseconds since the Unix epoch from the Instant.
-->

---

# HOW DO I? Date and time info

- Each Temporal object has read-only properties
- <small>`year`, `month`, `monthCode`, `day`, `hour`, `minute`, `second`, `millisecond`, `microsecond`, `nanosecond`, `calendar`, `timeZone`, `offset`, `era`, `eraYear`, `dayOfWeek`, `dayOfYear`, `weekOfYear`, `daysInWeek`, `daysInMonth`, `daysInYear`, `monthsInYear`, `inLeapYear`, `hoursInDay`, `startOfDay`, `offsetNanoseconds`, `epochMilliseconds`, `epochNanoseconds`</small>
- Only ZonedDateTime has all these at once, the others have subsets

<!--
Temporal types have read-only properties with which we can get this sort of information. This is a full list of these properties. Not every type has each property; only the ones that make sense for that type. So, for example, "year" is only on types that have calendars, "offset" only exists on ZonedDateTime, and the "epochNanosconds" and similar properties are only on the exact time types.

In our case we need the epochMilliseconds property.
-->

---

# Get a Unix timestamp in ms

```js
> Temporal.Now.instant().epochMilliseconds
1627682409993
```

<!--
Putting the previous slides together into this line of code: we get the result of Temporal.Now.instant, and examine its epochMilliseconds property to obtain a Unix timestamp in milliseconds.
-->

---

<!-- _class: lead -->

# A medium one

What is the date one month from today?

<!--
Now for the medium task. The question we have to answer is, what is the date one month from today?

We already saw in the previous task how to figure out what the date is today, so let's skip that for the moment (although I'll talk more about how that interacts with calendars in a moment.) How do we add one month to a date?
-->

---

# HOW DO I? Arithmetic

- `add()`/`subtract()` take a `Temporal.Duration` and return a new Temporal object of the same type, that far in the future or the past
- `since()`/`until()` take another Temporal object of the same type and return a `Temporal.Duration` representing the amount of time that elapsed from one to the other

```js
const monthLater = date.add({ months: 1 })
```

<!--
When I mentioned Duration a few slides ago, I mentioned arithmetic methods. This is where we will need to use one of those arithmetic methods. We have add and subtract for adding or subtracting a Duration. We also have since and until that determine the elapsed time between two Temporal objects.

In our case we use the add method, and pass it a property bag that automatically gets converted to a Duration, representing one month, like in this line of code.

Now this 'date' variable, where does it come from? We know we need to use one of the Now methods. Specifically, Now.plainDateISO().
-->

---

# Calendar support

- The standard "machine" calendar is the ISO 8601 calendar
- "Human" readable calendars are common in user-facing use cases
- Much of the world uses the Gregorian calendar
- Can print non-ISO dates with `toLocaleString()`, but that is not good enough

<!--
So, a little bit about calendars. The ISO calendar can be thought of as the "machine" calendar. How it works is standardized, with no regional or language dependence.

The Gregorian calendar is a human calendar that is used in a large part of the world with minor variations, but it is similar enough to the ISO calendar that you can switch between the two without having to do any mental calculations. 

However, although a large part of the world uses the Gregorian calendar, that does not mean that the whole world uses it. Some regions don't; and in some regions, people use the Gregorian calendar for business purposes, and another one for religious purposes.

Already without Temporal, you can print an old JavaScript Date object in a non-ISO, non-Gregorian calendar if the user wants it. That's good enough for some applications, but not for answering this question, because how long one month is depends on which month you're talking about, in which calendar.
-->

---

# What is the date, one month from today?

```js
> today = new Date('2024-10-09');
> today.toLocaleString('en', { calendar: 'hebrew', dateStyle: 'long', timeZone: 'UTC' });
'7 Tishri 5785'

> nextMonth = new Date(today);
> nextMonth.setUTCMonth(nextMonth.getUTCMonth() + 1);
> nextMonth.toLocaleString('en', { calendar: 'hebrew', dateStyle: 'long', timeZone: 'UTC' });
'8 Heshvan 5785'  // WRONG!
```

<!--
Here's an example of how that could go wrong using the old Date. Let's say today's date in the Gregorian calendar is October 9th, and one month later is November 9th. But in the Hebrew calendar, these two dates are not one month apart this year. They are one month and one day apart. So it is not enough just to add an ISO calendar month and print the dates in another calendar; you have to actually consider the calendar when performing the addition.

The lesson here is when working with dates that the user will see, is to use the user's calendar, not the machine ISO calendar.
-->

---

# What is the date, one month from today?

```js
const calendar = ... // choose the appropriate calendar
const today = Temporal.Now.plainDateISO.withCalendar(calendar);
console.log(today.add({ months: 1 }).toLocaleString());
// example output in my locale: "2024-12-18"
```

<!--
Putting it all together, the proper way to get the date one month from today is to define exactly what "month" you mean by choosing the calendar!

You can get the calendar from the user's preferences in your app, or from their locale using the resolvedOptions method of Intl.DateTimeFormat (though beware; this does not always correspond to what the user actually wants!)

Then, you can get today's date in that calendar using Now.plainDateISO, add one month to it using the add method, and format it for the user with the toLocaleString method. Here's an example of what it looks like in my locale: 2024 dash 12 dash 18.
-->

---

<!-- _class: lead -->

# A complicated one

What times are these conference sessions for me?

![](sessions.png)

<!--
Now for the complicated task. We need to answer the question, using the information on this conference website, what times do these sessions happen for me? Which sessions can I attend, when I'm not asleep?

(I would've loved to use the JSNation US website for this, but it conveniently already puts the sessions in your browser's local time zone. Well done!)

This question turns out to be surprisingly hard to answer, if we _don't_ use a computer, because it requires a lot of mental gymnastics with time zones and subtracting or adding hours. Most people, myself included, just put "11 o'clock AEST" into a search engine and hope that the search engine correctly guesses what I want. Temporal's strongly typed approach is perfect for solving this problem.
-->

---

# What time is the conference for me?

What we know:
- The two calendar dates that the sessions occur on (July 23 & 30, 2025)
- The start times of the sessions and their time zones (11 AM AEST, noon BST, 1 PM EDT)
- My time zone
- The hours (in my local time) at which I am willing to be at my computer

<!--
Here are the facts that we know. We know from the website the two calendar dates of the conference: July 23rd and 30th. I got the screenshot from a past conference but let's say for this example the dates are in 2025.

There are three sessions on each conference day. We know the local start times of each of the three sessions, and what time zones those local start times are given in.

I also know my time zone, and I know the hours in my local time zone during which I'm willing to be online watching a conference.
-->

---

# What time is the conference for me?

- For each session date,
  - For each session,
    - Calculate the exact time of that session start and end on that date;
    - If the local time of the session in my time zone falls within the hours that I'm willing to be at my computer,
      - Print the local time of the session in my time zone.

<!--
Here's pseudocode of what we need to do. For each date and each session, we know the wall time and the time zone, which allows us to figure out the exact time that the session starts and ends. Then we need to go from exact time back to wall time in my time zone to check whether I'm awake or not. If I am, then we print out that session.
-->

---

## What time is the conference for me?

```js
const dates = [
  Temporal.PlainDate.from('2025-07-23'),
  Temporal.PlainDate.from('2025-07-30')
];
const sessions = [
  { timeZone: 'Australia/Brisbane', hour: 11 },
  { timeZone: 'Europe/London', hour: 12 },
  { timeZone: 'America/New_York', hour: 13 }
];
const sessionLength = Temporal.Duration.from({ hours: 4 });
const myTimeZone = Temporal.Now.timeZoneId();
```

<!--
Here's the first part of the code, where we set things up. We have the calendar dates, the session time zones and start hours, the length of the sessions, and my time zone.

For this calculation it's OK to use the ISO calendar for the dates because I'm reckoning my awake times in the ISO calendar as well.

There are a few things to explain here. One of them is how I determined these time zone names! On the website we had Australia Eastern Standard Time, British Summer Time, and Eastern Standard Time (with North America implied in that last one.) But those are human names, not machine identifiers. These strings with the slashes, like Australia-slash-Brisbane, are the official identifiers of the IANA time zone database, and that's how you refer to time zones in Temporal. If you don't know the IANA identifier for a particular time zone, you can usually find it out on Wikipedia. Sometimes the human abbreviations are ambiguous, but the identifiers are always precise.

The next question is why are we using these from() methods to create our Temporal objects?
-->

---

# HOW DO I? Construction

- Constructors are low-level
- Accept numerical arguments for the date/time data

```js
new Temporal.PlainTime(13, 37)  // ⇒ 13:37:00
new Temporal.Instant(946684860000000000n)  // ⇒ 2000-01-01T00:01:00Z
new Temporal.PlainDate(2019, 6, 24);  // ⇒ 2019-06-24
```

<!--
As you might expect, you could also use a Temporal type's constructor to create an instance of it, but the constructors are intended for low-level use where you have all the data in exactly the right format, like in these examples.
-->

---

# HOW DO I? Construction

- `from()` static methods are high-level and friendly
- Accept property bags, ISO strings, instances
- Recommended in most cases

```js
Temporal.PlainYearMonth.from({ year: 2019, month: 6 });
Temporal.PlainYearMonth.from('2019-06-24T15:43:27');
Temporal.PlainYearMonth.from(myPlainDate);
```

<!--
On the other hand, there are these factory methods named from(), which are high-level and accept many more kinds of data. In particular, they are often more readable. In most cases I'd recommend using from() to create a Temporal object.
-->

---

# HOW DO I? Comparison

- `equals()` lets you know whether two Temporal objects of the same type are exactly equal
- `Temporal.{type}.compare(obj1, obj2)` is for sorting and ordering

```js
dates.sort(Temporal.PlainDate.compare);
```

<!--
The other piece of data that I mentioned we know is what times I'm willing to be online for the conference. I want to write a little function that takes a wall-clock time and returns true if the time is within my online hours.

For that we have to do comparisons. Each type has a compare static method that we use for this. It's a static method so that you can use it as an argument to the sort method of Arrays.

If instead you just want the common case of checking whether two Temporal objects are equal, use the equals method.
-->

---

## What time is the conference for me?

```js
const myCoffeeTime = Temporal.PlainTime.from('08:00');
const myBedtime = Temporal.PlainTime.from('23:00');

function iCanBeOnlineAt(time) {
  return Temporal.PlainTime.compare(myCoffeeTime, time) <= 0 &&
         Temporal.PlainTime.compare(time, myBedtime) <= 0;
}
```

<!--
Now back to writing code. Here is the function that tells whether I can be online. We are using PlainTime.compare to implement it.

Here I _am_ taking advantage of the fact that my online hours do not cross midnight... we'd have to be slightly more complicated to support the case where they did.
-->

---

## What time is the conference for me?

```js
const formatter = new Intl.DateTimeFormat(/* your preferred format here */);
console.log('Put the conference in your calendar:');
dates.forEach((date) => {
  sessions.forEach(({ timeZone, hour }) => {
    const sessionStart = date.toZonedDateTime({ timeZone, plainTime: { hour } });
    const mySessionStart = sessionStart.withTimeZone(myTimeZone);
    const mySessionEnd = mySessionStart.add(sessionLength);
    if (iCanBeOnlineAt(mySessionStart) && iCanBeOnlineAt(mySessionEnd)) {
      console.log(formatter.formatRange(
        mySessionStart.toPlainDateTime(), mySessionEnd.toPlainDateTime()));
    }
  });
});
```

<!--
And now this code is how the actual calculation works, that I previously outlined in pseudocode.

We loop through each conference date and the info for each session, and convert it to the exact time sessionStart, as a ZonedDateTime. Then we use the withTimeZone method to get a new ZonedDateTime object for the start of the session in my time zone, instead of the conference session's time zone. Then we add the session length using the add method that we're already familiar with, to get the end of the session in my time zone. These we pass to our comparison function, and if they are during my online times, we print out the start and the end of the session using formatRange.

If you're watching a recording of this talk, you might want to pause it at this point to examine the code in more detail.

I'll quickly go into details about the new methods that we've seen here.
-->

---

# HOW DO I? Conversion

- Conversion methods can remove or add information
- Argument: any information that needs to be added
- Some types contain all the information of some other types

```js
const date = Temporal.PlainDate.from('2006-08-24');
const time = Temporal.PlainTime.from('15:23:30.003');
date.toPlainDateTime(time);  // ⇒ 2006-08-24T15:23:30.003
date.toPlainMonthDay();  // ⇒ 08-24
```

<!--
The toZonedDateTime method that we saw is an example of one of the methods for conversion between Temporal types. To convert from one Temporal type to another, you either need to add information or drop information. If you drop information, like the year when you go from PlainDate to PlainMonthDay, the method doesn't take any argument. On the other hand, if you add information, like the clock time when you go from PlainDate to PlainDateTime, the method will take the required information as an argument. In our case we converted from PlainDate to ZonedDateTime, so we had to add both a time zone and a time, which the toZonedDateTime method takes in a property bag.
-->

---

# HOW DO I? Modification

- `with()` methods
- Argument: a property bag with some components of the same type
- Returns a new Temporal object with the provided fields replaced
- Remember, Temporal objects are immutable!
- Separate `withCalendar()` and `withTimeZone()` methods

```js
const date = Temporal.PlainDate.from('2006-01-24');
// What's the last day of this month?
date.with({ day: date.daysInMonth });  // ⇒ 2006-01-31
date.withCalendar('islamic-civil');  // ⇒ 1426-12-24 AH
```

<!--
The other new method you saw was withTimeZone. This is a good time to get into how to modify a Temporal object. Technically you can't! Temporal objects are immutable. But you can get a new object of the same type with one or more components replaced. This is what the with method is for. It takes a property bag with one or more components to replace, and returns a new object.

Here you can see an example: we take a date, and replace the day with the last day of the month.

We have separate methods for withCalendar and withTimeZone, because these two can't be provided at the same time as any other properties. It would be ambiguous which property you'd have to replace first.
-->

---

## What time is the conference for me?

```
Put the conference in your calendar:
Tuesday, July 22, 2025, 6:00–10:00 p.m.
Wednesday, July 23, 2025, 10:00 a.m.–2:00 p.m.
Tuesday, July 29, 2025, 6:00–10:00 p.m.
Wednesday, July 30, 2025, 10:00 a.m.–2:00 p.m.
```

<!--
Take that code, put it all together, run it, and here is the nicely formatted answer! (This is the answer in my time zone, with my locale's formatting, with dateStyle full and timeStyle short. Your answer will likely be different.)

This is a seemingly simple question answered in more lines of code than you might expect, but I seriously would not have wanted to even try to write this code with the old Date object. It would've been possible, by carefully making sure the Dates were all in UTC, and converting accordingly by manually adding or subtracting the right number of hours, but really susceptible to bugs.
-->

---

# Learning more

- https://tc39.es/proposal-temporal/docs/ (to appear on MDN)

<!--
I hope you've found this a useful tour of Temporal. And if you have the old Date object in your code base, I hope you're excited to replace it with Temporal! You can check out the documentation at this link. At some point it will graduate and you'll be able to find it on MDN instead, along with the rest of JavaScript's built-ins.
-->

---

<!-- _class: invert -->

<!-- Thank you for your attention. -->
