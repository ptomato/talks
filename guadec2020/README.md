---
theme: gaia
style: |
  /*@import url('https://fonts.googleapis.com/css2?family=Cantarell:ital,wght@0,400;0,700;1,400;1,700&display=swap');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');*/
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.14.0/css/all.min.css');
  .twocolumn { display: flex; }
  .col { flex: 1 }
  section {
    background-color: white;
    background-image: linear-gradient(90deg, rgba(216, 216, 216, .03) 50%,
                                      transparent 50%),
      linear-gradient(90deg, rgba(216, 216, 216, .06) 50%, transparent 50%),
      linear-gradient(90deg, transparent 50%, rgba(216, 216, 216, .08) 50%),
      linear-gradient(90deg, transparent 50%, rgba(216, 216, 216, .1) 50%);
    background-size: 26px, 58px, 74px, 106px;
    color: black;
    font-family: Cantarell;
    letter-spacing: 0;
  }
  section:not(.gaia):not(.invert) h1, section:not(.gaia):not(.invert) h2 {
    color: #0288d1;
  }
  code {
    background-color: rgb(216, 216, 216);
    border-radius: 5px;
    color: black;
    font-family: Hack
  }
  pre code {
    background-color: black;
    color: white;
  }
  .hljs-number, .hljs-params { color: rgb(71, 155, 204); }
  .time {
    background-color: black;
    color: white;
    font-size: 150%;
    padding: 0.2em;
    text-decoration: underline 2px red;
  }
  .credit {
    font-size: 80%;
    position: absolute;
    bottom: 5%;
    left: 70%;
  }
  .hat::before {
    content: '🗣';
    margin: 0 0.25em;
  }
  .hat { font-weight: bold; }
---

<!-- _class: gaia -->
<style scoped>
  h1 {
    line-height: 1.1;
    margin-top: 4em !important;
  }
  h2 {
    font-weight: normal;
    line-height: 1.1;
    margin-top: 0 !important;
  }
</style>

# What's new with JavaScript in GNOME

## The 2020 edition

**Philip Chimento** • <i class="fab fa-rocketchat"></i> pchimento • <i class="fab fa-gitlab"></i>&thinsp;<i class="fab fa-github"></i> ptomato • <i class="fab fa-twitter"></i> @therealptomato
GUADEC Online, July 25, 2020

<!--
  Introduce self
-->

---

# Introduction: What we will talk about

- Same thing we talk about every year!
- **What's new** in GJS?
  - Including a bit about a lesser-known corner: `Intl`
- **What's next** in GJS?
- We need some **help!** Here's what you can do

<!--
  So this talk is for people who write code for the GNOME platform in the JavaScript programming language.
  GNOME has its own JavaScript engine, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  The first half will be about what's new with GJS in GNOME 3.36 and 3.38 and how it can benefit you.
  The second half will be about what improvements you can expect to see coming up in GNOME 3.40.
  The third half will be about what we need help with and how you can get involved!
-->

---

# Introduction

- <span class="hat">Shout-outs</span> are a thank-you to someone who contributed a feature
- Presentation is full of [links](https://www.youtube.com/watch?v=dQw4w9WgXcQ) if you want to click through later
- If you want to follow along: **ptomato.name/talks/guadec2020**

<!--
  This presentation is also meant to be a resource that you can consult later. So I will be running pretty quickly through things like the new APIs, because they're not really that interesting to talk about; but you can go back and read them later and click through the links.

  If you see a shout-out icon it's because someone made a substantial contribution that we should thank them for!
-->

---

<!-- _class: invert lead -->

# What's new in JavaScript land for GNOME 3.36 and 3.38?

---

# Developer experience

- [gjs-docs](https://gjs-docs.gnome.org/) is now way easier to maintain and contribute to <span class="hat">Meg Ford</span>
- Documentation added for GNOME Shell internals <span class="hat">Andy Holmes</span>

![drop-shadow](gjsdocs.png)

<!--
  The developer experience is an important part of GJS, because if you have a platform for people to write code on, then you need to make sure they can write that code easily and without annoyance.
  One of the most important parts of the developer experience is the gjs-docs website.
  We now have two maintainers of this website.
  Meg gave this website a new home on GitLab and did some OpenShift magic to make it auto-deploy when a merge request is merged.
  Now, if you're missing a documentation set, you can just add it, build the Docker container, check that it works, and send us a merge request.
  Andy added lots of documentation sets; including ones for the internal libraries of GNOME Shell like St and the internal Clutter, so that it's more useful for shell extension developers.
-->

---

# Crash fixes

<!--
  We have a few known crashes in GJS.
  Crashes are bad because GNOME Shell includes GJS, so if it's possible to crash GJS by running bad JavaScript code, then it brings down the entire session.
-->

---

<!-- _class: lead gaia -->

## It shouldn't be possible to crash GNOME Shell just by writing JS code.

<!--
  This is our operating principle for fixing crashes in GJS.
  It means that even if you're using something unsupported, it should not crash.
-->

---

# Crash fixes

- [9 crashing bugs still open](https://gitlab.gnome.org/GNOME/gjs/-/issues?label_name%5B%5D=1.+Crash)
- 6 fixed, 5 new since last GUADEC 📈
- Improvements to type safety <span class="hat">Marco Trevisan</span>

<!--
  We fixed 6 crashes in the past year, and 5 new ones have been reported, so, one less. up and to the right I guess?
  Marco went through and introduced type safety in many parts of the codebase. Some of these we're still working on getting merged. Type safety helps us avoid crashes in the future by making certain kinds of errors into compile-time errors rather than crashes.
-->

---

# Performance fixes

- Looking up properties that may come from C:
  - e.g. `myWindow.present()`
  - Don't look up `myWindow.customProperty`
  - Negative lookup cache <span class="hat">Daniel Van Vugt</span>

<!--
  Now I want to talk about some performance fixes.
  Property lookup is a very hot code path.
  It's one of the things that the JavaScript engine optimizes very tightly, if you are dealing with regular JavaScript objects.
  But with objects from GNOME libraries, we always have to check if the property might come from C, which is a bit slower.
  But in the opposite case, once you know that `myWindow.customProperty` doesn't come from C, you never have to look it up again.
  Daniel implemented a negative lookup cache.
-->

---

# Performance fixes

- Calling C functions
  - Translating arguments is slow
  - e.g. `void gtk_c_func(int arg1, double arg2)` → `Gtk.c_func(number, number)`
  - Argument cache <span class="hat">Giovanni Campagna</span>

<!--
  Another hot path is calling C functions through gobject-introspection.
  Every time you call this C function you have to take a JavaScript number and convert it into the correct C type. This happened via a giant switch statement and it was slow.
  Giovanni wrote an argument cache that does the giant switch statement only once, the first time the function is encountered, and caches function pointers to the functions that do the actual conversion.
  You might ask where does he get the time to do this while working on Almond! Well he actually wrote it many years ago, but for various reasons it was never merged.
  I have been adapting these patches so that they work with modern GJS and they are almost ready to merge now.
-->

---

# GObject properties

- Adding GObject properties to your class was always really verbose
- Easy to do notifications incorrectly, for example
- Now you can leave out the boring part:

```js
// Just delete all this!
get my_property() {
  return this._myProperty;
}
set my_property(value) {
  if (this._myProperty === value)
    return;
  this._myProperty = value;
  this.notify('my-property');
}
```

<!--
  GObject properties. It was always really tedious to define these because you had to write getters and setters with a bunch of boilerplate. Now if you don't need anything special from your getters and setters, you can just leave them out.
  (If you do need special logic, then you still have to write them yourself.)
-->

---

# New edition of JS language

- JavaScript is gaining features every year
- No longer the language that everyone loved to hate
- Which of these new features can you use in GJS?

<!--
  It used to be that JavaScript was pretty universally terrible. Now it's just like any other programming language although maybe its dark corners are weirder than most.

  I'm going to run quickly through the new features that we got this year when we upgraded the JavaScript engine.
  This will go fast, but if you want to explore these features more, there is actually an appendix to this slide deck, so if you look at the PDF later, flip to the end, there will be more information.
-->

---

# New JS language features in 3.36

- `String.trimStart()`, `String.trimEnd()`
- `String.matchAll()`
- `Array.flat()`
- `Array.flatMap()`
- `Object.fromEntries()`
- `globalThis`
- `BigInt`, `BigInt64Array`, `BigUint64Array`
- String generics removed
- New `Intl` features

<!--
  So, a couple of new string, array, and object methods;
  the global object is now called "globalThis" which is standardized across Javascript platforms (before, it was called "window" which we inherited from Firefox);
  we have arbitrary precision integers, yay;
  and then two other things that I will cover in a bit more detail.
-->

---

# String generics removed

<div class="twocolumn">
<div class="col">

- Nonstandard way of calling String methods on a non-string object
- Use [moz68tool](https://gitlab.gnome.org/ptomato/moz60tool/-/blob/master/moz68tool) to scan your code for them
- Don't use array generics either, Mozilla will remove them soon
</div>
<div class="col">

:-1:
```js
let num = 15;
String.endsWith(num, 5);
```
:+1:
```js
String.prototype.endsWith.call(num, 5);
String(num).endsWith(5);
`${num}`.endsWith(5);
// ⇒ true
```
</div>
</div>

<!--
  This is something that Mozilla removed from Firefox. It was nonstandard and no other browsers had ever implemented it, and it was never going to become a standard. So, when we upgraded our JavaScript engine, we had no choice but that it stopped working in GJS as well.
  Luckily we have a tool that can scan your code for this usage, and alert you that you have to fix it. (The idea for having such a tool came from an audience question at an earlier GUADEC!)
-->

---

# New `Intl` features

<div class="twocolumn">
<div class="col">

`Intl.RelativeTimeFormat`
- Useful for user interfaces
</div>
<div class="col">

```js
rtf = new Intl.RelativeTimeFormat('en',
  {style: 'narrow'})
rtf.format(-1, 'day')
// ⇒ "1 day ago"

rtf = new Intl.RelativeTimeFormat('es',
  {numeric: 'auto'})
rtf.format(2, 'day')
// ⇒ "pasado mañana"
```
</div>
</div>

<!--
  A new addition to the Intl object is RelativeTimeFormat.
  This is great for user interfaces where you want to show how long ago an item was modified, for example.
-->

---

<!-- _class: invert lead -->

# A small digression about `Intl`

<!--
  So that was it for the new JavaScript language features.
  While we're on the subject of Intl, I want to make a pitch for why you should use it in your app or extension, as well as GNOME Shell.
-->

---

# `Intl`

- A global object
- Does internationalization stuff for you
- Some things awkward to do with `gettext`. `Intl` makes them easy
- Some things `gettext` does well. `Intl` mostly doesn't do those

<!--
  Intl is a global object that does internationalization stuff.
  "Internationalization" doesn't mean fetching translations for UI messages like gettext does.
  That you should still do with gettext.
  But Intl is great for formatting all sorts of things in a way that's aware of the user's locale.
-->

---

# `Intl`

- `Intl.Collator` - locale-sensitive sorting
- `Intl.DateTimeFormat` - printing dates and times (`"Fri Jul 17"`)
- `Intl.NumberFormat` - printing numbers (`"€ 0,00"`)
- `Intl.RelativeTimeFormat` - printing relative times (`"3 days ago"`)
- More in future editions of JS!

<!--
  You can do sorting, date and time formatting, number formatting, and as I mentioned a few slides ago, formatting relative times.

  So, why you should use Intl.
  Imagine you're a translator and you see...
-->

---

# Why you should use `Intl`

```js
/* Translators: this is the month name and day number
 followed by a time string in 24h format.
 i.e. "May 25, 14:30" */
format = N_("%B %-d, %H\u2236%M");
```

- How does the translator know what to put there for their locale?

<!--
  ..this.
  This is actually from from gnome-shell.
  Not only does the translator have to figure out what the correct short date and time format is for their locale, and how to express it in this cryptic strftime format.
  They also have to punctuate it correctly, because in their translation UI this "U+2236 RATIO MARK" might look at first glance like a regular colon.
-->

---

<!-- _class: lead gaia -->

# “Well why don't they just read the manual for strftime?”

---

<!-- _class: lead gaia -->

![drop-shadow](canteven.gif)

<!--
  Sorry I'm not going to make some overworked translation volunteers stop what they're doing and decipher the strftime codebook!

  And even if they did... now imagine you're the developer and you're integrating the new translations...
-->

---

# Why you should use `Intl`

```py
#. Translators: this is the month name and day number
#. followed by a time string in 24h format.
#. i.e. "May 25, 14:30"
#: js/misc/util.js:255
#, no-c-format
msgid "%B %-d, %H∶%M"
msgstr "%j, %R %p"  😱😱😱
```

<!--
  ...and you see this.
  Quick, is `"%j, %R %p"` a valid strftime string?
  The other question is how do you know that what the translator puts in there is correct?
  If you see this in a translation commit, you can read the strftime manual yourself to check it, but you can only guess or ask the translator if that was what they really meant!
-->

---

# Why you should use `Intl`

<div class="twocolumn">
<div class="col">

- More readable, even if longer
- Less burden on GNOME translators
</div>
<div class="col">

```js
format = new Intl.DateTimeFormat(myLocale, {
  month: 'short',
  day: 'numeric',
  hour: '2-digit',
  minute: '2-digit',
  hourCycle: 'h24',
})
format.format(Date.now())
// ⇒ "Jul. 17, 18:01" in my locale
```
</div>
</div>

<!--
  If you use Intl, then the code will look like this.
  It's certainly longer but, at least to me, it looks more readable.
  And most importantly, it doesn't need to be translated.
-->

---

# Why you should use `Intl`

- Who _does_ translate those, anyway?
  - [ICU project](http://site.icu-project.org/home)
  - Unicode [Common Locale Data Repository (CLDR)](http://cldr.unicode.org/)
  - They have the power of major browser vendors behind them

<!--
  Well you might ask, certainly someone has to translate that. Who is it? The answer is that these translations are provided by ICU and Unicode, which are projects with major corporate support behind them.
  So don't burden our volunteers to repeat stuff that's already paid for by large companies and freely available!
-->

---

# Why you should use `Intl`

- What if I don't like the ICU/CLDR's translations in my UI?
  - You can still customize them using `formatToParts()`
  - Supported by most `Intl` formatters

<!--
  You might say that this gives up our control of how we want dates and times to be displayed.
  And in some cases that is right.
-->

---

# Why you should use `Intl`

- What if I don't like the ICU/CLDR's translations in my UI?
  - You can still customize them using `formatToParts()`
  - Supported by most `Intl` formatters

## <span style="display: block; text-align: center"><span class="time">12:00</span> <span class="time">12∶00</span></span>

<!--
  An example of when you might want to do this is that ICU's translations use a colon character instead of the typographically correct ratio mark.
  In GNOME we do care about the little details like this.
  Intl formatters have a formatToParts method that gives you tagged segments of the string and allows you to customize them before pasting them together and presenting the string to your user.
-->

---

<!-- _class: invert lead -->

# What's next in GJS?

<!--
  OK, the next part of the presentation will be about what's coming up this year in GJS.
-->

---

# What's upcoming in 2020–2021?

- Progress towards removing the Big Hammer
- Progress towards ES Modules
- Following edition of JS language (based on Firefox 78)

<!--
  I'm going to talk about the Big Hammer, Modules, and the next upgrade of the JavaScript engine.
-->

---

# Big Hammer

- What is the Big Hammer?
- Why does it need to be removed?
- Why isn't it removed yet?

<!-- What is the Big Hammer? -->

---

<!-- _class: gaia lead -->

![width:800px drop-shadow](hammer.gif)

<!-- It's not this. -->

---

# What is the Big Hammer?

- I gave a [talk about this](https://www.youtube.com/watch?v=ZPZGAUPe2pU) at GUADEC last year
- Some objects accidentally survive a garbage collection
- The “Big Hammer” detects this and runs an extra garbage collection

![drop-shadow](garbage.png)

<!--
  The Big Hammer was actually the topic of the talk I gave at GUADEC last year in Thessaloniki, so if you want more in-depth information then there's a video you can watch!
  The short story. Javascript is a garbage-collected language, so it's supposed to delete memory automatically when the programmer is done using it. Because all of our GNOME platform libraries are writeen in C and because of the way we have to interface these two programming languages, sometimes we don't delete all the memory in one garbage collection. The Big Hammer was a solution introduced by Georges that detects when this happens, and then runs an extra garbage collection a bit later.
-->

---

# Why does it need to be removed?

- When memory is constantly being used and discarded, extra garbage collections happen quite often
- Bad for performance

<!--
  The Big Hammer improved the memory usage of GNOME Shell for a lot of people, so why do we need to remove it?
  Well, when you are constantly using memory, like during normal usage of GNOME Shell, these extra garbage collections are basically happening as fast as we let them. (They are rate-limited to every 10 seconds).
  This is not good for performance, and it occasionally interrupts animations.
-->

---

# Why isn't it removed yet?

- We have a fix and it (mostly) works
- Except for a few strange cases... <span class="hat">Evan Welsh</span>

<!--
  We do have a fix though, and I told the story of this fix at GUADEC last year as well!
  With the fix, we are deleting the extra memory, and these constant extra garbage collections are not needed.
  There are a few cases where the fix doesn't work, but Evan has proposed an extra piece that should take care of these problems.
  So why don't we go ahead with the fix?
-->

---

# Why isn't it removed yet?

- Ironically, the fix leads to higher average memory usage
- Public relations catastrophe! :grimacing:

<!--
  The frustrating part is, with the fix, the average memory usage is actually higher because we are not running extra garbage collections every few seconds anymore.
  This memory is not leaked, certainly it all goes away eventually when the next garbage collection happens. It's just that without the constant garbage collections, GNOME Shell occupies more space by default, if the space is available.
  This is not even different from how GNOME Shell worked before the Big Hammer. But now people's expectations for low memory usage have been set by the Big Hammer.
  Can you imagine the explosion that would happen on Reddit if we released this fix as-is and people saw the number go up in System Monitor? (Even though that's an inaccurate way to measure the memory usage - don't do that)
-->

---

# Why isn't it removed yet?

- We need to tune the garbage collector to work better for our use case
- We need help quantifying acceptable memory usage

<!--
  Here's what we need to do to get this working.
  The garbage collector (which is the same one that is used in your Firefox browser) has a lot of controls that you can adjust, and we need to adjust these carefully to fit our use case. Which is something that we should have done a long time ago, but the controls are actually pretty confusing and it's hard to know exactly the effects that they will have.
  In order to do this - we need to be able to accurately measure the performance and memory usage of gnome-shell, apps, etc. We can't customize the garbage collector if we have no idea whether our changes make things better or worse. I personally don't have experience measuring gnome-shell and don't know what to look out for! so this is where we need help.
-->

---

<!-- _class: lead gaia -->

![drop-shadow](i-have-no-idea-what-im-typing.jpg)

---

# Why isn't it removed yet?

- We need to tune the garbage collector to work better for our use case
- We need help quantifying acceptable memory usage
- We need to figure out some solution that fits:
  - Shell
  - Apps
  - Random scripts that don't even run a main loop

<!--
  Finally, GJS is used in a lot of different situations! gnome-shell and extensions need to be basically always running, apps run for a while but can be opened and closed, and we also have scripts that run for a short time, maybe not even needing garbage collection at all. We have to come up with some solution that works for all three.
-->

---

# ES Modules

- What are ES modules?
- When can we use them?

<!--
  Next thing you can look out for in the coming year is ES Modules. I'll explain what those are. So, you know how, generally, programming languages let you split your source code into several files?
  That wasn't possible in Javascript until a few years ago.
  Why did JavaScript not have something that is so fundamental to other programming languages?
-->

---

<!-- _class: gaia -->

# “The typical JavaScript program is only one line.”

<!--
  They used to say that the typical JavaScript program was only one line.
  Back in the beginning days of the web it really was true that JavaScript was mostly used for form validation.
-->

---

<!-- _class: gaia -->

# “The typical JavaScript program is only one line.”

```html
<input type="text" id="phone-number"
       onchange="if ($('phone-number').val().length !== 10) alert('Invalid phone number!')">
```

<!--
  You might remember overly strict form validation code like this, from the early days of the web, popping up an alert if you didn't put your phone number in exactly like the site expected!
-->

---

# What are ES modules?

- GJS had its own module system: `imports`

:-1:
```js
const System = imports.system;  // loads built-in System module
const {Gtk} = imports.gi;  // loads binding for Gtk
imports.searchPath.unshift('.');
const MyModule = imports.src.myModule;  // loads ./src/myModule.js
```

<!--
  A lot of non-browser platforms that used JavaScript made up something to address this shortcoming.
  GNOME did that as well, of course.
  We got our own homemade module system which has served well for 12 years.
-->

---

# What are ES modules?

- Now we have one solution for this across JavaScript platforms
- Standardized in ECMAScript 2015: "ES Modules"

:+1:
```js
import System from 'system';
import Gtk from 'gi://Gtk';
import { MyClass } from './src/myModule.js';
```

<!--
  But now JavaScript has its own solution, ES Modules, where "ES" stands for "ECMAScript".
  It's better to use the same thing across all platforms.
  ES Modules are slightly nicer, as well. They allow things like asynchronous module loading.
-->

---

# When can we use ES modules?

- Currently working on a way to use the two module systems side by side <span class="hat">Evan Welsh</span>
- Technically, ES modules are a slightly different syntax than regular JavaScript

<!--
  We hope to make these available in GNOME 3.40.
-->

---

# Next edition of JS

- Based on Firefox 78 <span class="hat">Evan Welsh</span>

<!--
  And finally, this year we have another JavaScript engine upgrade coming up.
  There's a chance that Evan and I might get this done here at GUADEC during the hallway track!
  But otherwise it will be in GNOME 3.40.
-->

---

# Next edition of JS

- `??` operator
- `?.` operator
- Static class fields
- Numeric separators (`1_000_000`)
- `String.replaceAll()`
- `Promise.allSettled()`
- `Intl.Locale`
- `Intl.ListFormat`

<!--
  Here are the cool things we get when that happens.
-->

---

<!-- _class: invert lead -->

# Help!

<!--
  Now here's what we need help with.
  Why do we need help? GJS underpins a lot of the GNOME desktop experience yet it only has a few people working on it, most of them not paid, and none of them full time.
-->

---

# Improving the developer experience

- For GNOME Shell
- For apps
- For shell extensions

<!--
  I think the developer experience is probably where GJS and its ecosystem lag the most behind.
  I think about the developer experience separately for three separate groups, because they all do very different things with GJS.
-->

---

# Developer experience: shell extensions

- Shell extensions seen as "separate" from GNOME
- One of the biggest sources of community unhappiness
- Sri gave a talk on Friday on this topic
- Join the unconference session on Sunday 18:00 UTC!

<!--
  You may have seen Sri's talk on Rebooting Extensions.
  The developer experience for shell extensions is probably the worst of these three groups.
  I'll be at the session on Sunday to listen about what we can do, and I'll gladly help someone who wants to tackle one of these ideas.
-->

---

# Developer experience: Shell and apps

- Despite some improvements, developer tools are still not very good
- Here are some ideas
- These projects are not too large
- Can be picked up without knowledge of all of the internals of GJS

<!--
  For the other two groups I have some projects in mind that would be ideal to get started with.
-->

---

# Improve console output

```text
gjs> ({a:5, b:3, foo:true})
[object Object]

gjs> [undefined, null]
,

gjs> 😬😬😬
```

[Issue #104](https://gitlab.gnome.org/GNOME/gjs/issues/104) — Better console interpreter output

<!--
  One thing that trips people up is how annoying and confusing the console output is in GJS.
  You can see some of the problems here.
  This goes for the console in GNOME Shell Looking Glass as well.
  NodeJS does this much better and we could mostly just steal their implementation!
-->

---

# Improve debugger

- Debugger is quite bare-bones
- Doesn't handle multiple source files very well
- Most code is spread over multiple source files

- [Issue #207](https://gitlab.gnome.org/GNOME/gjs/issues/207) — Source locations and `list` command
- [Issue #208](https://gitlab.gnome.org/GNOME/gjs/) — `backtrace full` command

<!--
  The other thing is that the debugger isn't used very often. That's too bad because it's cool that we have a debugger! but it's just too cumbersome to use it on multiple files. And as I mentioned when we talked about modules, most code has multiple files these days.
-->

---

# What else do we need help with?

- Become a code reviewer for GJS
- Create a good workflow for people who want to get started contributing to GJS
- Create a good process for measuring performance and memory usage in GNOME Shell

<!--
  We need code reviewers! Being a code reviewer can be a great way to get acquainted with the code base if you want to start contributing.

  Unlike most GNOME projects we don't have a good workflow for new contributors to get started. It's difficult to get set up to build GJS. With most GNOME apps you just download the flatpak manifest and get started. But GJS is a system component so there isn't a flatpak manifest. We have JHbuild but it's frustrating; we have toolbox but it only works well on certain distributions. (And cannot be used to test gnome-shell.)

  And then as I mentioned when we talked about the Big Hammer, we need to figure out how to get good measurements of performance so that we can customize the garbage collector for our needs, and get rid of these garbage collection problems once and for all.

  So now is a great time to start!
-->

---

<!-- _class: lead invert -->

# Let's hack!

<!--
  If you saw a project that you thought was interesting, let's use this time!
  Talk to me and I can help you get started during the hallway track.
-->

---

<!-- _class: gaia -->

# Thanks

- <span class="hat">GJS contributors from 3.36 and 3.38</span>
- Background pattern by [Randy Merrill](https://leaverou.github.io/css3patterns/#cicada-stripes) — MIT license

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

---

<!-- _class: gaia -->

![bg](computercat.jpg)

# <span style="text-shadow: -1px -1px 0 black, 1px 1px 0 black, -1px 1px 0 black, 1px -1px 0 black">Questions?</span>

<span class="credit">Image: [IRCat](https://pixabay.com/users/IRCat-10981235) from [Pixabay](https://pixabay.com/)</span>

<!--
  I put a JavaScript Cat in my presentation every year so here is a picture of JavaScript Cat feeling tired of collecting garbage.
-->

---

<!-- _class: invert lead -->

# Appendix

---

# New String features

<div class="twocolumn">
<div class="col">

`String.trimStart()`, `String.trimEnd()`
- New standardized methods for trimming whitespace
- replaces the old nonstandard `trimLeft`, `trimRight`
</div>
<div class="col">

:+1:
```js
'   foo  '.trimStart()
// ⇒ "foo  "
'   foo  '.trimEnd()
// ⇒ "   foo"
```
:-1:
```js
'   foo  '.trimLeft()
'   foo  '.trimRight()
```
</div>
</div>

---

# New String features

<div class="twocolumn">
<div class="col">

`String.matchAll()`
- Easier access to regular expression capture groups
- Easier iterating through multiple matches in a string
</div>
<div class="col">

```js
const s = 'GNOME 3.36 and 3.38';
const pat = /(\d+)\.(\d+)/g;
for (let [v, major, minor] of s.matchAll(pat)) {
  // On first iteration:
  //   v = '3.36'
  //   major = '3'
  //   minor = '36'
  // On second iteration:
  //   v = '3.38'
  //   major = '3'
  //   minor = '38'
}
```
</div>
</div>

---

# New Array features

<div class="twocolumn">
<div class="col">

`Array.flat()`
- Well known from lodash and friends
</div>
<div class="col">

```js
[1, 2, [3, 4]].flat()
// ⇒ [1, 2, 3, 4]
```
</div>
</div>

---

# New Array features

<div class="twocolumn">
<div class="col">

`Array.flatMap()`
- A sort of reverse `Array.filter()`
- You can add elements while iterating functional-style
</div>
<div class="col">

```js
function dashComponents(strings) {
  return strings.flatMap(s => s.split('-'));
}

dashComponents(['foo-bar', 'a-b-c'])
// ⇒ ['foo', 'bar', 'a', 'b', 'c']
```
</div>
</div>


---

# New Object features

<div class="twocolumn">
<div class="col">

`Object.fromEntries()`
- Create an object from iterable key-value pairs
</div>
<div class="col">

```js
o = Object.fromEntries(['a', 1], ['b', 2])

o.a
// ⇒ 1
o.b
// ⇒ 2
```
</div>
</div>

---

# New environment features

<div class="twocolumn">
<div class="col">

`globalThis`
- Ever find it weird that GJS's global object is named `window`?
</div>
<div class="col">

:+1:
```js
if (typeof globalThis.foo === 'undefined')
```
:-1:
```js
if (typeof window.foo === 'undefined')
```
</div>
</div>

<!--
  You're not alone if you found it weird that "window" is the name of the global object.
  That's a relic from the time when Javascript only ran in browsers.
  Javascript has now standardized on calling the global object "globalThis" in all environments.
-->

---

# BigInt

<div class="twocolumn">
<div class="col">

- Large numbers in JS are inexact
- BigInt is a new type in JS, arbitrary-precision integers
- BigInt literals are a number suffixed with `n`
</div>
<div class="col">

```js
Number.MAX_SAFE_INTEGER
// ⇒ 9007199254740991
9007199254740992 === 9007199254740993
// ⇒ true
BigInt(Number.MAX_SAFE_INTEGER)
// ⇒ 9007199254740991n
9007199254740992n === 9007199254740993n
// ⇒ false
```
</div>
</div>

---

# When you should use BigInt

- If you didn't already need to use it? Probably ignore it for now
- In the future, 64-bit APIs will accept BigInt as well as numbers
- Looking for a way to change the type of constants like `GLib.MAXINT64` without breaking existing code <span class="hat">Evan Welsh</span>

<!--
  You will probably want to use BigInt if you're doing 64-bit stuff in JavaScript, but given that large 64-bit numbers have always been inaccurate in JavaScript, nobody is probably doing that!
  It will become more useful when we figure out a way to backwards compatibly pass BigInts to gobject-introspection APIs.
-->

---

# BigInt

<div class="twocolumn">
<div class="col">

- Also two new typed array types for BigInts that fit in 64 bits
</div>
<div class="col">

```js
let s = new BigInt64Array([-1n, -2n]);
let u = new BigUint64Array([1n, 2n]);
```
</div>
</div>


---

<!-- _class: gaia -->

---

<!-- _class: gaia lead -->

![drop-shadow](spidermonkey.svg)

<!-- Bonus slide -->
