---
theme: gaia
paginate: true
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.1.1/css/all.min.css');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  code { font-family: Hack; }
  section { font-family: Cantarell, sans-serif; letter-spacing: 0; }
  section.lead.invert { text-shadow: 0 0 10px black, 0 0 20px black; }
  section.smaller-type li, ul ul li { font-size: 85% }
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
_class: gaia lead
_footer: Photo by <a href="https://www.pexels.com/@nihalkarkala/">Nihal Karkala</a> from <a href="https://www.pexels.com/photo/cute-furry-cat-wearing-sunglasses-7944876/">Pexels</a>

Photo by Nihal Karkala: 
-->

# What's new with JavaScript in GNOME

![bg right](shades.jpg)

## The 2022 edition

**Philip Chimento**
<i class="fa-solid fa-message"></i> @ptomato:gnome.org
<i class="fab fa-gitlab"></i>&thinsp;<i class="fab fa-github"></i> ptomato
<i class="fab fa-twitter"></i> @therealptomato
GUADEC, July 20, 2022

<!--
  Hi! My name is Philip Chimento. I've been a regular GNOME contributor for about 10 years. In that time I've written apps and now I maintain our JavaScript engine. In a stunning coincidence, at my day job at Igalia, I also work on JavaScript engines.

  Unfortunately I can't be here in person so as you can see I'm giving the talk remotely. I hope you all are enjoying Mexico.
-->

---

# Introduction: What I will talk about

- Same thing I talk about every year! (but shorter)
- **What's new and what's next** in GJS?
- We need some **help!** Here's what you can do

<!--
  Today I'll be talking about what's new with JavaScript in GNOME. I've given similar talks at previous GUADECs and this is the 2022 episode, so if you've been here before you probably know what to expect.

  As every year, this talk is primarily aimed at people who write code for the GNOME platform in the JavaScript programming language, whether that is GNOME Shell, apps, shell extensions, or even command line scripts.
  GNOME has its own JavaScript engine, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  The difference from last year is that I asked for a shorter presentation slot. In past years I talked a lot in the "what's new and what's next" sections about the various ways to modernize your code, but nowadays we are running a pretty modern JavaScript in GNOME, with no modernization "backlog". So while I'll still give a tour of what's new and what's next, I'll speed through that part of the presentation pretty quickly. I won't leave it out - lots of people contributed lots of cool things and that deserves celebration! But we have a lot more visibility of these improvements these days, with better documentation, a more active community of GJS developers, and initiatives like This Week in GNOME, so a yearly GUADEC talk is no longer the only channel where people find out about these things.

  So, the rest of the presentation will be about what we need help with. As every year, I'll talk about the "Big Hammer" bug and what help we need to finally squash it. But there are also projects at every other level that you can get involved with.
-->

---

<!--
 _footer: Photo by <a href="https://www.pexels.com/@karolina-grabowska/">Karolina Grabowska</a> from <a href="https://www.pexels.com/photo/white-and-brown-cat-lying-beside-a-laptop-and-toys-5468268/">Pexels</a>
-->

![bg right](mousecat.jpg)

# Introduction

- Presentation is full of links if you want to click through later
- If you want to follow along: **ptomato.name/talks/guadec2022**

<!--
  This presentation is also meant to be a resource that you can consult later. So I will be running pretty quickly through things like the new APIs, because they're not really that interesting to talk about; but you can go back and read them later and click through the links.

  The slides are already available on my web space so if you want to click on the links NOW, you can already go there and follow along with the presentation.
-->

---

<!-- _class: invert lead -->

# What's new in JavaScript land for GNOME ㊷ and ㊸?

<!--
    Part One. What's new for GNOME 42 and the soon-to-be-released GNOME 43?
    By the way, I'll use these little circled 42 and 43 icons to show things that you can expect to see in GNOME 42 and 43. And when you see a hat emoji, that's me tipping my hat to the person who contributed it.
-->

---

# BigInts ㊷

- You can now pass BigInt values to introspected functions that would otherwise take 64-bit integers in C
- Return values we're [not sure](https://gitlab.gnome.org/GNOME/gjs/-/issues/271) what to do with yet... (e.g. `GLib.get_monotonic_time()`)
- :tophat: Marco Trevisan

```js
GLib.Variant.new_int64(2n ** 62n)
GLib.MAXINT64_BIGINT
```

<!--
    In GNOME 42 Marco gave us the ability to pass BigInts to functions which in C would take a 64-bit integer. We've had BigInts for quite a while in GJS but they didn't integrate with the GNOME platform. At the same time, we had the problem that you can't pass the full range of 64-bit integers to C because in JavaScript, numbers are floating point, so not precise above a certain value. Now you can use BigInt for these cases.

    How to return 64-bit integers we're not really sure how to do it yet. Since these functions have always returned regular numbers before BigInt existed, it'd break people's code to have them start returning BigInts. We have several ideas on that linked thread, please chime in if your code would use this feature.
-->

---

# Promise performance improvements ㊷

- `Promise` resolution can no longer get starved out when the main loop is busy with events
- :tophat: Evan Welsh, Marco Trevisan, and Philip Withnall

<!--
    Also in GNOME 42 we got a three-person collaboration to write a new system for scheduling Promise resolution, for when you're writing asynchronous code. Evan wrote a GSource, Marco added some things, and Philip did the code review so that I didn't have to become a GSource expert.

    One thing I like about this collaboration is that it was a success story of how GUADEC helps to connect people. I presented this task in my talk, and Philip volunteered to do the code review already then. Every little bit helps!
-->

---

# `setTimeout`/`setInterval` ㊷

- `setTimeout()` and `setInterval()` just like in the browser
- Uses GLib main loop
- Familiarity for people coming from web development
- :tophat: Evan Welsh

<!--
    Another GNOME 42 hit from Evan was to implement global setTimeout and setInterval functions just like in the browser, but integrated with the GLib main loop. Just like the global console object and TextEncoder that Evan did for GNOME 41, this is a familiarity boost for newcomers who might be familiar with web development.
-->

---

# Pretty print ㊸

- Better output of objects at the interactive console prompt
- No more `[object Object]`
- Improved output also extends to the `log()` and `logError()` functions
- Soon in the debugger too
- :tophat: Nasah Kuma

```
gjs> ({a:1, b:2})
{ a: 1, b: 2 }

(used to print "[object Object]")
```

<!--
    Former Outreachy intern Nasah returns with an implementation of a long-standing issue: the output in the interactive interpreter used to be not great for objects, it would print the dreaded "object Object" and then you have to resort to workarounds such as Object.keys or JSON.stringify. This is no longer the case! Now you get a usefully pretty-printed object.

    You also get this in log() and logError() which are often used for debugging. Not in print() because that's often used for user-visible output.
-->

---

# DBus async methods ㊸

- Generated DBus proxy classes have sync and async methods for each DBus method
- e.g. for method `org.freedesktop.login1.Manager.Inhibit` you'd get `InhibitSync()` and `InhibitRemote()` methods
- The "Remote" method takes a callback
- Now we have an "Async" method (e.g. `InhibitAsync()`) that returns a `Promise`
- :tophat: Sergio Costas

<!--
    This cycle we get from Sergio, a convenience for async calls of DBus methods when you generate a DBus proxy class. It used to be that you had to use a callback, and additionally the method was weirdly named "Remote". "Remote" still exists, so your old code will continue to work, but now there is also an "Async" method and it returns a Promise so you can write code in a more modern style, with async/await.
-->

---

# JS engine upgrades

GJS's underlying JavaScript engine is the one from Firefox, but embedded.

- Tracks Firefox's long-term support version (currently Firefox ESR 102)
- Brings in newer editions of the JS language, and security updates
- GNOME ㊷: upgraded from 78 to 91
- GNOME ㊸: will upgrade to 102

<!--
    I almost always have a section on what JS language features are new, when we upgrade the verion of the underlying JavaScript engine to one from a newer Firefox. This year we upgraded to Firefox 91 in 
    GNOME 42, and we're planning to upgrade to Firefox 102 in GNOME 43.

    This used to take up the bulk of my presentations, but more recently, this has been somewhat less interesting, since Firefox no longer has a backlog of specified JS language features to implement. So now the new language features we get are still pretty new and not that widely used yet.

    Also, Firefox used to have lots of nonstandard extensions in its JavaScript, but those are I think all gone now. I used to have to warn people about all sorts of code that wouldn't work anymore. Luckily, this time around there are no backwards incompatibilities that you have to remove from your code, this time, that we know of, yet!

    By the way, why two upgrades in one year?!
    Firefox's release cycle for long-term support versions is now a bit less than a year, so the point in the GNOME release cycle when we can upgrade keeps changing.
-->

---

# JS engine upgrades

- `at()`: Python-style indexing for [arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at), [strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/at), and [typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/at) ㊷
- `??=`, `&&=`, `||=` [operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment) ㊷
- `export * as ... from ...` ㊷
- [Regular expression `d` flag](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/hasIndices) ㊷
- [`static { ... }` blocks](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Class_static_initialization_blocks) ㊷
- (continued...)

<!--
    Here's the first half of what we get:
    - the AT method, so we can finally easily get the last element of an array or string with at(-1)
    - the logical assignment operators, these are really useful for dealing with objects used as options bags
    - export-star-from, this is useful when aggregating several modules into one export
    - the D flag for regular expressions, use this when you want to find exactly where in the string each match group occurred
    - static blocks, these can be useful for things that you might otherwise do in a ClassInit method
-->

---

# JS engine upgrades

- [`Error.cause`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause) ㊷
- [`WeakRef`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef) ㊷
- [`Promise.any()`: First successful sub-promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any) ㊷
- Top-level `await` ㊷
- [`Object.hasOwn()` shorthand](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn) ㊸
- Internationalization ㊷ ㊸

<!--
    ...and there's more!
    - Error cause is useful when you want to rethrow an error but tack another error on top of it
    - WeakRef may be useful for breaking reference cycles although I haven't really tried it out yet
    - Promise.any is handy when you have several async operations and you want to wait for one of them to succeed, but you don't care which one
    - toplevel await gives you a bit cleaner code when you are importing modules (although you have to be careful to use it in a way that doesn't impact your startup time)
    - Object.hasOwn is just a nice shorthand so that you don't have to do Object.prototype.hasOwnProperty.call
    - there's a lot of new stuff in internationalization that I think apps and GNOME shell would benefit from but not a lot of people realize it exists. I don't have enough time in this talk to go into all of it, but maybe I will submit a talk just on that topic next year, especially if we get Temporal.
-->

---

# JS engine upgrades

### [`#private` class members](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) ㊷

```js
class MyClass {
    #value;
    #counter = 0;

    constructor(value) {
        this.#value = value;
    }

    #somePrivateMethod() {
        doSomethingWith(this.#value);
        this.#counter++;
    }
}
```

<!--
    The last and probably most visible thing that we get is private class members. We always used underscore to indicate private-by-convention. If you have a JavaScript class, you can now have truly encapsulated fields that cannot be accessed outside the class.

    At first this didn't work with GObject classes but now it does thanks to Evan. This brought in some other improvements as well, which I'll share in the next slide...
-->

---

# Goodbye `_init()` ㊷

- It's no longer necessary to use `_init()` to construct a GObject
- Just use `constructor()` like normal objects!
- :tophat: Evan Welsh

```js
class MyObject extends GObject.Object {
    #value;

    constructor(props = {}) {
        super(props);
        this.#value = this.#computeValue();
    }

    // ...
```

<!--
    While Evan was fixing GObject classes to work with class fields, he also fixed a long-standing annoyance: now you don't have to use underscore-init as the constructor anymore in GObject classes. You can write a constructor just like normal classes.
    So, a GObject class now looks like this - I've updated my example app to take advantage of this.    
-->

---

<!-- _class: invert lead -->

# How can _you_ help?

<!--
    Now we come to the second part of the presentation.

    Want more exciting stuff to happen? Here's how we could use your help!
-->

---

# Help define best practices

- We have a lot of best-practices material
- Talk to us in [#javascript:gnome.org](https://matrix.to/#javascript:gnome.org) about things confusing you!
- That's where you can find reviewers if you want to update these things

### This is a great place to contribute even if you're just beginning with GJS!

<!--
    First-time app and extension writers often copy code in order to get started quickly, and this is a perfectly normal thing to do! That's part of what free software is all about: the freedom to study software. So we should have good code and good practices available for people to learn from.

    If you've started using GJS and you find something that is confusing, or a bad example, come talk to us in our Matrix channel. The thing you're asking about may or may not be a mistake or a bug, or it may be outdated, but usually there's at least a good answer to the question "where would have been a good place for you to find this information?" that will move things forward for the next person to have the same question. Then if you make a merge request out of the information that would've helped you, that's incredibly helpful for the maintainers and everyone else too.

    Compared to a few years ago the situation has really improved. The "Extensions Rebooted" initiative has really breathed new life into the GJS community and we all owe them a big debt of thanks. We actually have several resources now, where we used to have none. I'll talk about them in the next slide.
-->

---

# Current JS best-practices material

- **[gtk-js-app](https://github.com/gcampax/gtk-js-app)**: example app, GTK 3, old-style imports
- **[Bloatpad](https://gitlab.gnome.org/ptomato/bloatpad)** (and accompanying [talk](http://ptomato.name/talks/las2021/)): example app, GTK 4, ES modules
- **[gjs.guide](https://gjs.guide/guides/)**: docs and tutorials collection for apps and extensions
- **[GTK4+GJS Book](https://rmnvgr.gitlab.io/gtk4-gjs-book/)**: example app + tutorial, ES modules
- **[GNOME Platform Demos](https://gitlab.gnome.org/Archive/gnome-devel-docs/-/tree/master/platform-demos/C)**: several example apps
  - Old, removed from web, but still some good stuff

<!--
    We have several example apps, and a couple of websites. gtk-js-app is a bit older but was updated not too long ago, it's still a decent example of a GTK 3 app although it still uses old-style imports. Bloatpad is from my talk last year at Linux Application Summit and I just recently updated it for GNOME 42. We also have a brand new example app from Romain in the GTK4+GJS Book. GJS Guide is really active these days with many guides being written.

    I used to think it was urgent that we consolidate all this material into one place, but I've kind of learned since then that it makes more sense to let people make whatever they are enthusiastic to make, and just make sure that there are plenty of links, and that if something is really old and contains out-of-date advice, it gets removed from the web.

    In the case of the GNOME Platform Demos, I always thought those were good stuff, but they are very old and the best practices have changed in the meantime. A good project might be to see which of those example apps are still relevant, rewrite them for the modern day, and move them into one of the maintained guides.

    A good way to help in general is to help edit all this material; keep it updated and even more importantly, look at it with fresh eyes. I can say from experience that when you author a guide like this, it's easy to suffer from the "curse of knowledge" where you have forgotten what is and isn't obvious to a newcomer. Helping with this is something that newcomers can uniquely do, and learn a lot in the process.
-->

---

# Best-practices material not in JS

- [GNOME Platform Getting Started](https://developer.gnome.org/documentation/tutorials/beginners/getting_started.html)
  - Available in C and Python
- [GNOME Platform Tutorials](https://developer.gnome.org/documentation/tutorials.html)
  - Some already available in JS, others not yet
- [HowDoI](https://wiki.gnome.org/HowDoI)
  - Old, but some good stuff in there
  - Useful bits should be ported to GNOME Platform Tutorials

<!--
    There are also some resources that exist in other languages but not in JavaScript. Notably the GNOME Platform Getting Started is kind of our flagship tutorial now, and is only available in C and Python. This would be a great and also impactful project for someone, to port the tutorial to JavaScript.

    On the same site there are more tutorials. Some of them exist in JavaScript, but not all. Ideally all of these tutorials and code snippets should be available in GJS!

    A bit older is How Do I. A lot of the content there has already been moved to GNOME Platform Tutorials, but not all. How Do I was pretty much all C code, so there's some porting to do there as well.
-->

---

# Native async operations

- Previously, focus of Outreachy 2018 (Avi Zajac) and 2021 (Veena Nagar)
- Lots of work done already

<!--
    Now how about an impactful coding project. For years we have quested after the holy grail of native async operations. It's been the topic of two Outreachy internships, Avi and Veena worked on it and improved the situation, but we need to 
    I will show you what it is in the next slide.
    As an example I'll use some simple code that reads the contents of one file and writes it to another file, both asynchronously.
-->

---

# Native async operations (2)

```js
// Old way: (PYRAMID OF DOOM!)
file.load_contents_async(/* cancel = */ null, (obj, result) => {
    try {
        const [, contents] = file.load_contents_finish(result);
    } catch (e) {
        logError(e);
        return;
    }
    file2.replace_contents_bytes_async(contents, /* etag = */ null, /* makeBackup = */ false,
        /* cancel = */ null, (obj, result) => {
            try {
                file.replace_contents_finish(result);
            } catch (e) {
                logError(e);
            }
        });
});
```

<!--
    This is the old way of doing things. Isn't it horrendous. It's not called the pyramid of doom for nothing. Imagine if you needed to chain even more async operations!

    Each operation has an async start function that takes a callback parameter, and a finish function that must be called inside the callback.

    Strictly speaking the try-catches aren't necessary here because when an exception is thrown inside a callback it'll get logged anyway when control returns to the main loop, since there's nowhere else to throw it. But we might fix that at some point if it's possible and doesn't break too much existing code.
-->

---

# Native async operations (3)

```js
// Current way:
Gio._promisify(Gio.File.prototype, 'load_contents_async');
Gio._promisify(Gio.File.prototype, 'replace_contents_bytes_async',
    'replace_contents_finish');
const [contents] = await file.load_contents_async(/* cancel = */ null);
await file2.replace_contents_bytes_async(contents, /* etag = */ null,
    /* makeBackup = */ false, /* cancel = */ null);
```

<!--
    This is what the current situation looks like. We opt in to making these two methods return Promises using an unofficial function, underscore-promisify. Once you do that, you can use await with them, and not pass callback parameters. This was originally released as a "technology preview" but it stuck around.
-->

---

# Native async operations (4)

```js
// End goal: (per-method opt-in no longer needed)
// Gio._promisify(Gio.File.prototype, 'load_contents_async');
// Gio._promisify(Gio.File.prototype, 'replace_contents_bytes_async',
//     'replace_contents_finish');
const [contents] = await file.load_contents_async(/* cancel = */ null);
await file2.replace_contents_bytes_async(contents, /* etag = */ null,
    /* makeBackup = */ false, /* cancel = */ null);
```

<!--
    The last bit we need to do is to just make this automatic, without the opt-in. So we can just delete these underscore-promisify lines. And we are halfway there.
-->

---

# Native async operations (5):

GIR annotations:
```xml
<method name="load_contents_async" c:identifier="g_file_load_contents_async"
        glib:finish-func="g_file_load_contents_finish"
        glib:sync-func="g_file_load_contents">
```

- We have a [merge request](https://gitlab.gnome.org/GNOME/gobject-introspection/-/merge_requests/278) implementing these
- But it needs matching gobject-introspection C API
- Then needs an implementation in GJS
- Good task if you know some C

<!--
    What the underscore-promisify does is tie the start and finish functions together, and it relies on you the programmer to provide that information. In order for it to be done automatically, we need gobject-introspection annotations to provide the information. As of last year we have a merge request that does this, and can put that information into the GIR files. However, before it can be merged, it still needs a C API.
    Then finally we'd need to implement it in GJS. 

    This would be a good medium-sized project for someone who knows C, and really impactful. There are any number of people who can help plan it out or review it, myself included.
-->

---

# About that [Big Hammer](https://gitlab.gnome.org/GNOME/gjs/-/issues/217)... :hammer:

**Please** help get this over the finish line.

We need help with:
- Help to **measure the impact** on GNOME Shell so we can tune the garbage collector
- A **communication plan**

(there are also some coding tasks, but they'll get done: [glib#623](https://gitlab.gnome.org/GNOME/glib/-/issues/623), [gjs#52](https://gitlab.gnome.org/GNOME/gjs/-/issues/52))

<!--
    Speaking of impactful.
    Well, I've milked four GUADEC talks out of this Big Hammer issue, so it's really time to get the fix landed.
    If you want all the details, watch my talk at GUADEC 2019, but I'll give a quick summary.
    We had an issue with the garbage collector in JS holding on to objects longer than it should have.
    We have a workaround, affectionately called the Big Hammer, that runs the garbage collector every 10 seconds to clean up these objects. However, this is obviously bad for performance. We have had a fix mostly finished for several years! But the problem is that if we merge it, then GNOME Shell's memory usage will go up again because we are no longer OVER-collecting garbage. Users do watch this stuff, and so if we do that, it will be a public relations disaster with all sorts of articles published complaining about memory leaks that are not actually memory leaks.

    So, there are only squishy, demotivating problems left to solve where it's not clear what success looks like, and we really need some help with these. What we really need is a way to measure and quantify the effect of the fix on GNOME Shell's performace, that takes both CPU time and memory usage into account, and is not just someone running Top - so that we can correctly and effectively tune the garbage collector's parameters for a long-lived process like GNOME Shell.

    We also need help with a communication plan, so we can figure out how to set expectations for users who have been awaiting this fix for a long time.

    Can you help quantify this, or help communicate it? PLEASE help.
-->

---

<!-- _class: gaia -->

# Thanks

GJS contributors from 42 and 43

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

<!--
    On that note, I'd like to end by acknowledging everyone who helped in any way with GJS in GNOME 42 and 43!

    Here's the license; you may reuse bits of this presentation as-is, with attribution, and not for commercial use.

    Now it's time for...
-->

---

<!--
_class: invert gaia lead
_footer: Image: <a href="https://pixabay.com/users/IRCat-10981235">IRCat</a> from <a href="https://pixabay.com/">Pixabay</a>
-->

![bg](computercat.jpg)

# Questions?

# &zwj;

<!--
  ...questions.
-->
