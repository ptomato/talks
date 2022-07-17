---
theme: gaia
paginate: true
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.1.1/css/all.min.css');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  code { font-family: Hack; }
  section { font-family: Cantarell, sans-serif; letter-spacing: 0; }
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
_class: gaia lead
_footer: Photo by <a href="https://www.pexels.com/@lucaspezeta">Lucas Pezeta</a> from <a href="https://www.pexels.com/photo/tabby-cat-sticking-its-tongue-out-2930204/">Pexels</a>
-->

# What's new with JavaScript in GNOME

![bg right](wink.jpg)

## The 2022 edition

**Philip Chimento**
<i class="fa-solid fa-message"></i> @ptomato:gnome.org
<i class="fab fa-gitlab"></i>&thinsp;<i class="fab fa-github"></i> ptomato
<i class="fab fa-twitter"></i> @therealptomato
GUADEC, July 20, 2022

<!--
  Hi! My name is Philip Chimento. I've been a regular GNOME contributor for about 9 years. I used to like tabs, now I like spaces. You can probably also see that I like cat pictures. At my day job, I work on JavaScript engines at Igalia.

  Today I'll be talking about what's new with JavaScript in GNOME. This is the 2022 episode of a talk that has become somewhat of a tradition at GUADEC. Unfortunately I can't be here in person so as you can see I'm giving the talk remotely. I hope you all are enjoying Mexico.
-->

---

# Introduction: What we will talk about

- Same thing we talk about every year! (but shorter)
- **What's new and what's next** in GJS?
- We need some **help!** Here's what you can do

<!--
  As every year, this talk is primarily aimed at people who write code for the GNOME platform in the JavaScript programming language, whether that is GNOME Shell, apps, shell extensions, or even command line scripts.
  GNOME has its own JavaScript engine, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  The difference from last year is that I asked for a shorter presentation slot.
  In past years I talked a lot in the "what's new and what's next" sections about the various ways to modernize your code, but nowadays we are running a pretty modern JavaScript in GNOME, with no modernization "backlog".
  So while I'll still give a tour of what's new and what's next, I'll speed through that part of the presentation pretty quickly. I won't leave it out - lots of people contributed lots of cool things and that deserves some celebration - but 

  The rest of the presentation will be about what we need help with. 
  I talk every year about the "Big Hammer" bug and we need some help to finally squash it.
  But there are also smaller projects at every level that you can get involved with.
-->

---

# Introduction

- Presentation is full of [links](https://www.youtube.com/watch?v=dQw4w9WgXcQ) if you want to click through later
- If you want to follow along: **ptomato.name/talks/guadec2022**

<!--
  This presentation is also meant to be a resource that you can consult later. So I will be running pretty quickly through things like the new APIs, because they're not really that interesting to talk about; but you can go back and read them later and click through the links.
-->

---

<!-- _class: invert lead -->

# What's new in JavaScript land for GNOME 42 and 43?

<!--
    Part One. What's new for GNOME 42 and the soon-to-be-released GNOME 43?
-->

㊷ ㊸ ㊹

<!--
    By the way, I'll use these little circled 42 and 43 icons to show things that you can expect to see in GNOME 42 and 43. And I'll doff my hat icon to the person who contributed it.
-->

- Enumerability ㊷
- Cairo.Context.prototype.textExtents ㊷
- setTimeout/setInterval ㊷
- pretty print ㊸
- DBus Async methods ㊸
- ActionMap.add_action_entries ㊸

---

# BigInts ㊷

```js
GLib.Variant.new_int64(2n ** 62n)
GLib.MAXINT64_BIGINT
```

---

# Performance improvements

- In particular, improvements to the performance of promises ㊷

---

# JS engine upgrades

### `at()`: Python-style indexing for [arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at), [strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/at), and [typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/at) ㊷

```js
const arr = [1, 2, 3, 4, 5, 6, 7];
arr.at(0)  // 1
arr.at(-1)  // 7
arr.at(999)  // undefined
'my string'.at(-7)  // ' '

arr[arr.length - 1]  // no longer necessary!
```

<!--
    We get the at() method for arrays and strings, which lets you do nicer indexing, passing -1 to get the last item, and so forth. So you don't have to calculate the length and risk off-by-one errors.
-->

---

# JS engine upgrades

### [`Promise.any()`: First successful sub-promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any) ㊷

```js
const cancellable = new Gio.Cancellable();
const fastestServer = await Promise.any([
    checkServer(EAST, cancellable),
    checkServer(WEST, cancellable),
    waitSeconds(30),
]);
if (!fastestServer) {
    cancellable.cancel();
    notify('No server found within 30 seconds');
}
```

<!--
    Last year we got Promise.all(), which waits until all of its sub-promises have resolved. Now we also get Promise.any(), which is kind of the opposite. It waits for any one of the sub-promises to succeed, and then resolves with that promise's value, ignoring the resolutions of the remaining promises. That sounds complicated, but this example might make it clearer. This sample code sends requests to two servers EAST and WEST, and resolves the promise with whichever server responds faster. But if neither server responds within 30 seconds, it'll give up.

    It's different from Promise.race(), where the promise resolves with the first sub-promise's resolution, even if that's a failure. In this example, Promise.race() wouldn't be the right thing to use, because if checkServer EAST was offline and returned an error quickly, then we'd get that error as the result of the promise, ignoring the result from checkServer WEST even though WEST might still be online.
-->

---

# JS engine upgrades

### `??=`, `&&=`, `||=` [operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment) ㊷

Short-circuiting assignment operators
- `a ??= b` - assign `a = b` if `a` is `null` or `undefined`
- `a ||= b` - assign `a = b` if `a` is falsey
- `a &&= b` - assign `a = b` if `a` is truthy

<!--
    We also get a bunch of new operators, that can be used to assign default values to a variable. They short-circuit, so the right hand side is not evaluated if the left-hand side already determines that the assignment doesn't take place.
-->

---

- `export * as ... from ...` ㊷
- Regular expression `d` flag ㊷
- `static { ... }` blocks ㊷
- Error cause ㊷
- WeakRef ㊷
- Top-level await ㊷
- Object.hasOwn() ㊸
- Internationalization ㊷ ㊸

---

# JS engine upgrades

### [`#private` class fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) ㊷

```js
class MyClass {
    #value;

    constructor(value) {
        this.#value = value;
    }

    someMethod() {
        doSomethingWith(this.#value);
    }
}
```

<!--
    In GNOME 42 we upgraded the JavaScript engine to the one from Firefox 91.

    Evan has been working on upgrading the JavaScript engine to the next long-term-support release from Firefox. Luckily, there are no backwards incompatibilities that you have to remove from your code, this time, that we know of, yet!

    I'll talk a bit about the new language features this will bring us. One thing that we get is private class fields. If you have a JavaScript class, you can now have truly encapsulated fields that cannot be accessed outside the class.

    At first this didn't work with GObject classes but now it does
-->

---

# Goodbye `_init()` ㊷

- It's no longer necessary to use `_init()` to construct a GObject
- Just use `constructor()` like normal objects!
- Class fields (public and private) now work with GObjects too
- See also [example app]

```js
class MyObject extends GObject.Object {
    #value;

    constructor(props = {}) {
        super(props);
        this.#value = this.#computeValue();
    }

    #computeValue() { ... }
}
```

---

# Documentation and Examples

- 🎩 Extensions Rebooted

<!--
    Finally I want to give a special shout out to the Extensions Rebooted initiative which apart from all the other work they've been doing that's important for the GNOME Desktop, have also been gradually improving the documentation of GJS.
-->

---

<!-- _class: invert lead -->

# How can _you_ help?

<!--
    Part Three. Want more exciting stuff to happen? Here's how we could use your help!
-->

---

# Help define best practices for the GNOME JS ecosystem

- [Sample app](https://github.com/gcampax/gtk-js-app) has been updated
- For an experiment with even newer stuff, see the [GJS Bloatpad](https://gitlab.gnome.org/ptomato/bloatpad)
- [Writing native Linux desktop apps with JavaScript](https://www.youtube.com/watch?v=YqmmK16rIm4) at LAS 2021
- Talk to us in [#javascript:gnome.org](https://matrix.to/#javascript:gnome.org) about things confusing you!

<!--
    First-time app and extension writers often copy code in order to get started quickly, and this is a perfectly normal thing to do! That's part of what free software is all about: the freedom to study software. So we should have good code and good practices available for people to learn from.

    I made some improvements recently on updating the GJS sample app, since some Outreachy applicants were running into problems with it being outdated this year. Also for my talk at the Linux Application Summit I made a somewhat more complicated sample app that integrates some tools from the NodeJS ecosystem, just to see if that was possible, and maybe some of these should become best practices as well. Evan is also exploring the integration between GJS and TypeScript and he is planning a Birds-of-a-Feather session on this topic on Friday at 1610 hours UTC.

    If you've started using GJS and you find something that is confusing, or a bad example, come talk to us in our Matrix channel. The thing you're asking about may or may not be a mistake or a bug, but usually there's at least a good answer to the question "where would have been a good place for you to find this information?" that will move things forward for the next person to have the same question.
-->

---

# Translate GNOME Platform Getting Started into JS

- [Getting Started](https://developer.gnome.org/documentation/tutorials/beginners/getting_started.html)
- Available in C and Python

---

# Native async operations

[Annotations](https://gitlab.gnome.org/GNOME/gobject-introspection/-/merge_requests/278):
```xml
<method name="load_contents_async" c:identifier="g_file_load_contents_async"
        glib:finish-func="g_file_load_contents_finish"
        glib:sync-func="g_file_load_contents">
```

```js
// Opt-in per method no longer needed:
// Gio._promisify(Gio.File.prototype, 'load_contents_async');
const [contents] = await file.load_contents_async(/* cancel = */ null);
```


<!--
    First off I'd like to talk about the work of Veena, GJS's Outreachy intern from the May 2021 round. She will also be giving a lightning talk about this, so if you want to know more, here's another reminder to go see the intern lightning talk session on Friday at 1500 hours UTC!

    Veena is working on adding annotations to gobject-introspection, such as finish-func and sync-func, that allow us to tie together these pairs of functions automatically.

    Previously we had to tie together these functions manually with Gio.promisify, in order to use async style programming. In the future, we'll be able to leave that out, like it shows here in the commented out code. Promisify was the work of Avi, our Outreachy intern from 2018, and I'm excited that it's being continued.

    The annotations are getting near to being ready in gobject-introspection. The corresponding changes in GJS are a stretch goal for the internship, so those may happen for GNOME 41 or later.
-->

---

# The [Big Hammer](https://gitlab.gnome.org/GNOME/gjs/-/issues/217), why isn't it removed?

Well-defined problems are nearly all solved. Squishy problems remain.

We need:
- ㊷ [better memory accounting](https://gitlab.gnome.org/GNOME/gjs/-/issues/52)
- to quantify acceptable memory usage for Shell, & [tune GC accordingly](https://gitlab.gnome.org/GNOME/gjs/-/issues/43)
- to figure out some solution that fits both Shell and apps
- a communication plan

<!--
    Finally I'll talk about the Big Hammer which has, you may have guessed, not yet been removed. I've talked about this at two previous GUADECs so I won't go over it yet again, but if you are curious, check out my talk from GUADEC 2019.

    At this point, we have almost run out of well-defined problems. The only one remaining is to implement better accounting of memory associated with objects, so that the malloc bytes in the JavaScript engine's statistics correspond better to the actual size of the GObjects that we have. This will also give us more accurate numbers in Sysprof as I mentioned earlier. Hopefully this will land in GNOME 42, but if someone would like to fix it earlier than I do, that would be welcome. There's also a GLib bug blocking it which would be great to have help with.

    After that, there are only squishy, demotivating problems left to solve where it's not clear what success looks like, and this is what we most need help with. If we just enable the fix and remove the Big Hammer, it will look like the memory usage of GNOME Shell will go up drastically and it will be a public relations disaster with all sorts of articles published about memory leaks. But the cause won't be a memory leak; it's just that we won't be running the garbage collector every 10 seconds anymore, so more garbage will accumulate, and the memory usage will go up. But not as high as it was before the Big Hammer, and that's the point! We don't actually want to collect garbage every few seconds. What we do want is to figure out the right tradeoff, for both GNOME Shell and apps, and figure out how to set expectations for users who have been awaiting this fix for a long time.

    Can you help collect data on this, or help communicate it? Let me know.
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

  I put a Shady JavaScript Cat in my presentation every year so here is a picture of JavaScript Cat feeling tired of solving squishy, unsatisfying problems.
-->
