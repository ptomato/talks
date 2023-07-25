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
-->

# JavaScript in GNOME


<div style="display: inline-grid; grid-template-columns: 1fr 1fr;">
<div>

## New Features

**Philip Chimento**
<i class="fa-solid fa-message"></i> @ptomato:gnome.org
<i class="fab fa-gitlab"></i> ptomato

</div>
<div>

## Community

**Sonny Piers**
<i class="fa-solid fa-message"></i> @sonny:gnome.org
<i class="fab fa-gitlab"></i> sonny

</div>
</div>

GUADEC, July 27, 2023

<!--
-->

---

<!-- _class: invert lead -->

# New features

---

# Introduction: What I will talk about

- Same thing I talk about every year! (but shorter)
- **What's new and what's next** in GJS?
- **Thanks** to everyone who contributed!

<!--
  Today I'll be talking about what's new and what's next with JavaScript in GNOME. I've given similar talks at previous GUADECs, so if you've been here before you probably know what to expect. What's new this year is that this is only one part of a larger presentation on GJS, highlighting exciting things like the Workbench app.

  This part of the talk is primarily aimed at people who write code for the GNOME platform in the JavaScript programming language, whether that is GNOME Shell, apps, shell extensions, or even command line scripts. GNOME has its own JavaScript engine for all these purposes, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  In past years I talked a lot in the "what's new and what's next" sections about the various ways to modernize your code, but nowadays we are running a pretty modern JavaScript in GNOME already, with no modernization "backlog". The new language features are still pretty exciting, but there's nothing earthshaking like ES6 classes. Additionally, we have a lot more visibility of these improvements these days, with better documentation, a more active community of GJS developers, and initiatives like This Week in GNOME, so a yearly GUADEC talk is no longer the only channel where people find out about these things. Nonetheless, I've gotten feedback that people still find these talks useful, so that's a good reason to keep doing them in addition to all those other things I mentioned.
-->

---

# Introduction

- Presentation is full of links if you want to click through later
- If you want to follow along: [**ptomato.name/talks/guadec2023**](https://ptomato.name/talks/guadec2023)

<!--
  This slide deck is also meant to be a resource that you can consult later. The slides are already available on my web space so if you want to click on the links NOW, you can already go there and follow along with the presentation.
-->

---

<!-- _class: invert lead -->

# What's new in JavaScript land for GNOME ㊹ and ㊺?

<!--
    What's new for GNOME 44 and the upcoming GNOME 45?
    By the way, I'll use these little circled 44 and 45 icons to show things that you can expect to see in GNOME 44 and 45. And when you see a hat emoji, that's me tipping my hat to the person who contributed it.
-->

---

# Mainloop/Promises interaction ㊹

- Did you start using promises in your code and notice it **<span style="color: #729FCF">FREEZE</span>**?
- Deadlock is unavoidable consequence of combining module imports with GLib main loop

<!--
    One thing that you might have noticed is that you started using 
-->

---

# Mainloop/Promises interaction ㊹

- `app.run(args)` and `mainloop.run()` are **blocking** calls
- importing a module is a **promise** (to the JS engine, at least)
- other promises resolve **after** the current one finishes
* This doesn't mix!

---

**launch.js**
```js
await import('./main.js');
```

**main.js**
```js
const app = new Gio.Application({flags: Gio.ApplicationFlags.HANDLES_OPEN});
app.connect('activate', () => {});
app.connect('open', whoWantsToAwaitForever);
app.run(args);  // runs GLib main loop NOW

async function whoWantsToAwaitForever(app, [file]) {
    app.hold();
    const [contents] = await file.load_contents_async(null);  // resolves AFTER import promise (i.e. NEVER)
    console.log(contents);
    app.release();
}
```

---

# Mainloop/Promises interaction ㊹

- When you use Promise async operations instead of callbacks,
  - Use `await Gio.Application.runAsync()` instead of `Gio.Application.run()`
  - Use `await GLib.MainLoop.runAsync()` instead of `GLib.MainLoop.run()`
- :tophat: Evan Welsh
- Would you like to help with a linter rule for this?

<!--
    It took a while to figure out what was going on here; I, at least, was more likely to assume that an app freezing in an I/O operation was something wrong with the I/O and not a quirk of how promises resolve in JavaScript! Evan got to the bottom of this.
-->

---

**launch.js**
```js
await import('./main.js');
```

**main.js**
```js
const app = new Gio.Application({flags: Gio.ApplicationFlags.HANDLES_OPEN});
app.connect('activate', () => {});
app.connect('open', iCantAwaitAnyLonger);
await app.runAsync(args);  // runs GLib main loop AFTER import promise

async function iCantAwaitAnyLonger(app, [file]) {
    app.hold();
    const [contents] = await file.load_contents_async(null);  // resolves next in queue when microtasks run
    console.log(contents);
    app.release();
}
```

---

# Performance improvements ㊹ & ㊺

- Improved memory leak checking for new merge requests in CI
- Fixes of existing memory leaks not previously caught
- Dramatic performance improvements in legacy signals module (used in Shell)
- :tophat: Marco Trevisan

<!--
    To my embarrassment, we were not making full use of AddressSanitizer on GitLab merge requests, so we failed to detect some memory leaks in the past 2 years. Marco found this out, and fixed all of the newly detected memory leaks.
-->

---

# JS engine upgrades ㊺

GJS's underlying JavaScript engine is the one from Firefox, but embedded.

- Tracks Firefox's long-term support version (currently Firefox ESR 102)
- Brings in newer editions of the JS language, and security updates
- GNOME ㊺: will upgrade to Firefox ESR 115
- :tophat: Xi Ruoyao

<!--
    I almost always have a section on what JavaScript language features are new, when we upgrade the version of the underlying JavaScript engine to one from a newer Firefox. This year we're planning to upgrade to Firefox 115 in GNOME 45.

    This used to take up the bulk of my presentations, but more recently, this has been somewhat less interesting, since Firefox no longer has a backlog of already-specified language features to implement. So now the new language features we get are still pretty new and not that widely used yet.

    Also, Firefox used to have lots of nonstandard extensions in its JavaScript, but those are I think all gone now. I used to have to warn people about all sorts of code that wouldn't work anymore after the upgrade. Luckily, this time around there are no backwards incompatibilities that you have to remove from your code, this time, that we know of, yet!
-->

---

# JS engine upgrades ㊺

- New `Array` methods
- Most of them also on typed arrays
    - (typed array: `Uint8Array`, `Int32Array`, etc.)

---

# findLast(Index) ㊺

- `arr.findLast()`: like `find()`, but starts at the end
- `arr.findLastIndex()`: like `findIndex()`, but starts at the end
- Just in case you need these; nice to not have to implement them yourself

```js
const a = [1, 2, 3, 4, 5];
function less3(item) { return item <3; }
a.findLast(less3)        // -> 2
a.findLastIndex(less3)   // -> 1
// For comparison:
a.find(less3)            // -> 1
a.findIndex(less3)       // -> 0
```

---

# Non-mutating array methods ㊺

- `arr.toReversed()`
- `arr.toSorted()`
- `arr.toSpliced()`
- `arr.with()`

---

# Non-mutating array methods ㊺

- `toReversed()`, `toSorted()`, and `toSpliced()` are copying versions of `reverse()`, `sort()`, `splice()`
- `arr.with(3, 5)` is a copying version of `arr[3] = 5`
- Programming in a non-mutating style has become more popular lately due to frameworks like React
- Affects GJS too (JS object-valued GObject properties)

```js
myObject.connect('notify::arr', onNotify);
myObject.arr.reverse();  // CAUTION
myObject.arr = myObject.arr.toReversed();  // OK
```

<!--
    For example, It's dangerous to do `reverse()` on an array that a GObject has as a property. Because the array changes, but you don't get a GObject property notification, which might break an invariant of your program.
-->

---

# Array.fromAsync ㊺

`Array.fromAsync()` is to `for await` as `Array.from()` is to `for`.

```js
// OLD
const chunks = [];
for await (const chunk of inputStream.createAsyncIterator(4)) {
    chunks.push(chunk);
}

// NEW
const chunks = await Array.fromAsync(inputStream.createAsyncIterator(4));
```

- Notice this new `createAsyncIterator()` method? :tophat: Sonny Piers ㊹

---

# Native async operations ㊺

- :tophat: Previously, focus of Outreachy 2018 (Avi Zajac) and 2021 (Veena Nagar)
- :tophat: Now being worked on by Evan Welsh

---

# Native async operations ㊺

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
    As an example I'll use a simple task: read the contents of one file and write it to another file, both asynchronously.

    This code you see here is the old way of doing things. Isn't it horrendous. Not simple at all! It's not called the pyramid of doom for nothing. Imagine if you needed to chain even more async operations!

    Each operation has an async start function that takes a callback parameter, and a finish function that must be called inside the callback.

    Strictly speaking the try-catches aren't necessary here because when an exception is thrown inside a callback it'll get logged anyway when control returns to the main loop, since there's nowhere else to throw it. But we might fix that at some point if it's possible and doesn't break too much existing code. So it's good practice to have them anyway, but it makes everything even less readable.
-->

---

# Native async operations ㊺

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
    This is what the current situation looks like. We opt in to making these two methods return Promises using an unofficial function, underscore-promisify. Once you do that, you can use await with them, and they will return Promises so you don't have to pass callback parameters. This was originally released as a "technology preview" but it stuck around.
-->

---

# Native async operations ㊺

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
    The last bit we need to do is to just make this automatic, without the manual opt-in. So we can just delete these underscore-promisify lines.
-->

---

# Native async operations ㊺

GIR annotations:
```xml
<method name="load_contents_async" c:identifier="g_file_load_contents_async"
        glib:finish-func="g_file_load_contents_finish"
        glib:sync-func="g_file_load_contents">
```

<!--
    What the underscore-promisify does is tie the start and finish functions together, and it relies on you the programmer to provide that information. In order for it to be done automatically, we need to get that information from gobject-introspection annotations. As of last year we have a merge request that adds those annotations, and can put that information into the GIR files, and now it also has a C API.
-->

---

# Import maps (maybe) ㊺



---

# About that [Big Hammer](https://gitlab.gnome.org/GNOME/gjs/-/issues/217)... :hammer:

**Please** help get this over the finish line.

We need help with:
- Help to **measure the impact** on GNOME Shell so we can tune the garbage collector
- A **communication plan**

(there are also some coding tasks, but they'll get done: [glib#623](https://gitlab.gnome.org/GNOME/glib/-/issues/623), [gjs#52](https://gitlab.gnome.org/GNOME/gjs/-/issues/52))

<!--
    Speaking of impactful. I had to mention the Big Hammer sometime.
    Well, I've milked four GUADEC talks out of this Big Hammer issue, so it's really time to get the fix landed.
    If you want all the details, watch my talk at GUADEC 2019, but I'll give a quick summary.
    We had an issue with the garbage collector in GJS holding on to objects longer than it should have, because of the interaction between JavaScript objects and GObjects referencing each other.
    We have a workaround, affectionately called the Big Hammer, that runs the garbage collector every 10 seconds to clean up these objects. However, this is obviously bad for performance. We have had a fix mostly finished for several years! But the problem is that if we merge it, then counterintuitively, GNOME Shell's memory usage will go up again because we are no longer OVER-collecting garbage. Users do watch this stuff, and so if we do that, it will be a public relations disaster with all sorts of articles published complaining about memory leaks that are not actually memory leaks.

    So, there are only squishy, demotivating problems left to solve where it's not clear what success looks like, and we really need some help with these. The first thing we need is a way to measure and quantify the effect of the fix on GNOME Shell's performace, that takes both CPU time and memory usage into account, and is not just someone running Top - so that we can correctly and effectively tune the garbage collector's parameters for a long-lived process like GNOME Shell. Last year I added some metrics for this to Sysprof, but I need help figuring out how to interpret the numbers.

    We also need help with a communication plan, so we can figure out how to set expectations for users who have been awaiting this fix for a long time.

    Can you help quantify this, or help communicate it? PLEASE help.
-->

---

<!-- _class: invert lead -->

# Community

---

<!-- _class: gaia -->

# Thanks

GJS contributors from 44 and 45

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

<!--
    On that note, I'd like to end by acknowledging everyone who helped in any way with GJS in GNOME 44 and 45!

    Here's the license for this slide deck; you may reuse bits as-is, with attribution, and not for commercial use.

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
