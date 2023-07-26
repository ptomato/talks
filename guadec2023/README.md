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
    This year we have a lot to tell! We have two sessions of GNOME JavaScript. Today you will see me, talking about new features, and Sonny, talking about the JavaScript community. Tomorrow you get Sonny again, talking about tooling, and Evan Welsh, talking about TypeScript.
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

<!--
    _footer: Image by <a href="https://pixabay.com/users/miezekieze-607096">miezekieze</a> from <a href="https://pixabay.com">Pixabay</a>
-->

### Mainloop-Promises interaction ㊹

- Did you start using promises in your code and notice it **<span style="color: #729FCF">FREEZE</span>**?
- Unavoidable consequence of combining dynamic imports with GLib main loop
- :tophat: Evan Welsh

![bg right](coldcat.jpg)

<!--
    One thing that you might have noticed is that you started using dynamic imports and promises in your code, and it suddenly would freeze instead of completing an asynchronous I/O operation.
    This was already a trap, but it used to be difficult to fall into it. If you used legacy imports, or sync imports, it wouldn't be a problem. Now that we have toplevel await it's much more attractive to use it to import a module dynamically, and 

    It took a while to figure out what was going on here. At least, I ran into the problem when I was porting an example app to use promise I/O. I assumed that my app freezing in an I/O operation was something wrong with the I/O and not a quirk of how promises resolve in JavaScript! And so I shoved it onto the "debug this later" pile. But Evan identified the problem, a workaround that could be used immediately, and a solution that landed in GNOME 44.
-->

---

**launch.js**
```js
await import('./main.js');  // This launches a new Promise
```

**main.js**
```js
const app = new Gio.Application({flags: Gio.ApplicationFlags.HANDLES_OPEN});
app.connect('activate', () => {});
app.connect('open', whoWantsToAwaitForever);
app.run(args);                                                // Blocking call, runs GLib main loop NOW

async function whoWantsToAwaitForever(app, [file]) {
    app.hold();
    const [contents] = await file.load_contents_async(null);  // resolves AFTER import promise (i.e. NEVER)
    console.log(contents);
    app.release();
}
```

<!--
    What happens here is that dynamic import is a promise. launch.js imports main.js, and main.js launches an async file operation which is also a promise. When the import finishes, the promise is queued to resolve. But the dynamic import promise never finishes resolving because THIS LINE (app.run) blocks, so the code never finishes executing, so the file operation promise never finishes resolving. Promises can't resolve nested in JavaScript.
-->

---

**launch.js**
```js
await import('./main.js');  // This launches a new Promise
```

**main.js**
```js
const app = new Gio.Application({flags: Gio.ApplicationFlags.HANDLES_OPEN});
app.connect('activate', () => {});
app.connect('open', iCantAwaitAnyLonger);
await app.runAsync(args);                                     // runs GLib main loop AFTER import promise

async function iCantAwaitAnyLonger(app, [file]) {
    app.hold();
    const [contents] = await file.load_contents_async(null);  // resolves next in queue when microtasks run
    console.log(contents);
    app.release();
}
```

<!--
    This is what the new code looks like.
    The API that Evan added is to schedule the blocking call to g_application_run or g_main_loop_run as its own promise, which only resolves when the blocking call is done. That way, the module load can resolve and then other promises can resolve as well.
-->

---

# Mainloop-Promises interaction ㊹

- Use `await app.runAsync()` instead of `app.run()` for `Gio.Application`
- Use `await loop.runAsync()` instead of `loop.run()` for `GLib.MainLoop`
- Would you like to help with a linter rule for this?

<!--
    Basically, it's always OK to await runAsync instead of calling run. So the recommendation is to just always do that.

    This is an evil trap for unsuspecting programmers. Having evil traps in our platform is not good, so it would be great if we could have a linter rule that would just warn you not to call run().
-->

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
    I almost always have a section on what JavaScript language features are new, when we upgrade the version of the underlying JavaScript engine to one from a newer Firefox. This year we're planning to upgrade to Firefox 115 in GNOME 45. Thanks to Xi Ruoyao who started the process.
-->

---

<!-- _footer: Image by <a href="https://pixabay.com/users/dimitrisvetsikas1969-1857980">Dimitris Vetsikas</a> from <a href="https://pixabay.com/">Pixabay</a> -->

# JS engine upgrades ㊺

- New `Array` methods
- Most of them also on typed arrays (`Uint8Array`, `Int32Array`, etc.)

![bg right](arraycat.jpg)

<!--
    The new features in the new version of SpiderMonkey are basically all array methods. They of course work on regular old JavaScript arrays. Most of them also work on typed arrays, in case you are using those. But not Array.fromAsync.
-->

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

<!--
    The first set of new array methods is findLast and findLastIndex. They are just like find and findIndex, but they start at the end. Occasionally you need these, so they are nice to have just so you don't make an off-by-one error when you implement it yourself.
    -->

---

# Non-mutating array methods ㊺

- `arr.toReversed()`
- `arr.toSorted()`
- `arr.toSpliced()`
- `arr.with()`

<!--
    The next set is these four, which I call non-mutating array methods. (The proposal to add them to JavaScript was originally called "change Array by copy". Some of my colleagues at Igalia actually worked on this proposal and adding the methods to SpiderMonkey!)
-->

---

# Non-mutating array methods ㊺

- `toReversed()`, `toSorted()`, and `toSpliced()` are copying versions of `reverse()`, `sort()`, `splice()`
- `arr.with(3, 5)` is a copying version of `arr[3] = 5`
- Programming in a non-mutating style has become more popular lately
- Affects GJS too (JS object-valued GObject properties)

```js
myObject.connect('notify::arr', onNotify);
myObject.arr.reverse();  // CAUTION
myObject.arr = myObject.arr.toReversed();  // OK
```

<!--
    toReversed, toSorted, and toSpliced are copying versions of reverse, sort, and splice. What this means, reverse changes the order of elements in the original array. toReversed returns a new array which is a different object from the original one.

    The motivation for adding this to JavaScript was that this non-mutating style of programming has become more popular recently. It's become idiomatic in many popular web frameworks such as React. But it can affect GNOME too!

    For example, It's dangerous to do `reverse()` on an array that is the value of a GObject property. Because the array changes, but you don't get a GObject property notification, which might break an invariant of your program.
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

<!--
    Array.fromAsync can be used to build an array from the results of an async iterator. This is a pretty common pattern (and it will be even more common when we get more iterator methods in a future JavaScript) and using Array.fromAsync is slightly more efficient because pushing each value into the array happens in the JS engine instead of in your code.
-->

---

# Native async operations ㊺

- :tophat: Previously, focus of Outreachy 2018 (Avi Zajac) and 2021 (Veena Nagar)
- :tophat: Now being worked on by Evan Welsh

<!--
    Here's something that I really hope we can land for GNOME 45. Native async operations. This has been a LONG-awaited feature and two outreachy interns have worked on pieces of it already. Evan is working on tying it all together.
-->

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
    This is what the current situation looks like. We opt in to making these two methods return Promises using an unofficial function, underscore-promisify. Once you do that, you can use await with them, and they will return Promises so you don't have to pass callback parameters. This was originally released as a "technology preview" but it stuck around. This was from Avi's internship in 2018.
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
    What the underscore-promisify does is tie the start and finish functions together, and it relies on you the programmer to provide that information. In order for it to be done automatically, we need to get that information from gobject-introspection annotations. As a result of Veena's internship in 2021 we have a merge request that adds those annotations, and can put that information into the GIR files. Now thanks to Evan it also has a C API.

    The last bit to do would be to use that API in GJS, which I'm happy to say Evan also has a draft for.
-->

---

# Import maps (maybe) ㊺

- [HTML Import Maps Spec](https://html.spec.whatwg.org/multipage/webappapis.html#import-maps)
- Normally, importing a "bare word" is reserved:

```js
import System from 'system';  // Built-in GJS module, reserved name
import foo from 'foo';  // ImportError, no way to supply this module
```

<!--
Another feature that it's possible we might get for GNOME 45 is import maps, but this still needs some discussion of how exactly it would work and how it might be used.

Import maps is originally a web feature. There's a link on this slide to the HTML spec.

How it works approximately. You might know that normally you can't import what is called a "bare word" - all imports must be a URL or a relative path that starts with a dot-slash. Bare word modules are reserved for the environment to provide. For example, in GJS we provide a built-in bare word module called "system", along with a few others.

If you try to import a bare word that we don't provide, you get an import error.
-->

---

# Import maps

**imports.json**
```json
{
    "imports": {
        "foo": "resource:///org/example/MyApp/foo.js"
    }
}
```

**main.js**
```js
#!/usr/bin/gjs -m --import-map imports.json
import foo from 'foo';  // imports foo from foo.js GResource
```

<!--
    With import maps, you get to map bare words to URLs or relative paths in a JSON file, which you provide to GJS on startup. GJS then makes those bare word modules available to your program, and behind the scenes redirects to the mapping you gave.
-->

---

# Possible uses in GNOME (1)

- Defining platform library versions all in one place
- Separate import map for multi version support, e.g. GTK 3 and GTK 4?

```json
{
    "imports": {
        "Gdk": "gi://Gdk?version=4.0",
        "Gio": "gi://Gio",
        "GObject": "gi://GObject",
        "Gtk": "gi://Gtk?version=4.0"
    }
}
```

<!--
    So why would you want to do this? Why not just provide the URLs in the first place, if you have to write them in the import map anyway?

    One thing you could do is define all the API versions of your GNOME platform libraries in one place. We do recommend for apps that you define these in your entry point, but that's kind of arbitrary, and maybe defining them in an import map makes more sense.

    You could even use this to implement multi version support, e.g. launch your app with either GTK 3 or 4 if the bits of API you use aren't too different. Or import a library if it's present and a shim if it's not. Otherwise you might have to use dynamic imports for that, which is a bit of a pain.
    -->

---

# Possible uses in GNOME (2)

- Use a build tool to download online dependencies
- But you might use GResource for this, anyway?

**imports.json.in**
```json
{ "imports": { "lodash": "https://unpkg.com/lodash@4.17.21" } }                           
```

**imports.json.out**
```json
{ "imports": { "lodash": "./_build/cache/lodash-4.17.21-7679856329457360398/lodash.js" } }
```

<!--
    Sonny might talk more about this tomorrow, but you could also have your build tools generate your import map so that online URLs are downloaded and cached. But on the other hand, you might use GResource for this in your build system anyway?
-->

---

# Import maps (maybe) ㊺

- Would you use this?
- We'd like to hear your use cases!

<!--
    So we still have some questions to answer, but if you would use this we'd love to hear from you!
-->

---

# About that [Big Hammer](https://gitlab.gnome.org/GNOME/gjs/-/issues/217)... :hammer:

**Please** help get this over the finish line.

We need help with:
- Help to **measure the impact** on GNOME Shell so we can tune the garbage collector
- A **communication plan**

(there are also some coding tasks, but they'll get done: [glib#623](https://gitlab.gnome.org/GNOME/glib/-/issues/623), [gjs#52](https://gitlab.gnome.org/GNOME/gjs/-/issues/52))

<!--
    Finally I'll give a short status update on the Big Hammer.
    If you want all the details, watch my talk at GUADEC 2019, but I'll give a quick summary.
    We had an issue with the garbage collector in GJS holding on to objects longer than it should have, because of the interaction between JavaScript objects and GObjects referencing each other.
    We have a workaround, affectionately called the Big Hammer, that runs the garbage collector every 10 seconds to clean up these objects. However, this is obviously bad for performance. We have had a fix mostly finished for several years! But the problem is that if we merge it, then counterintuitively, GNOME Shell's memory usage will go up again because we are no longer OVER-collecting garbage. Users do watch this stuff, and so if we do that, it will be a public relations disaster with all sorts of articles published complaining about memory leaks that are not actually memory leaks.

    So, there are only squishy, demotivating problems left to solve where it's not clear what success looks like, and we really need some help with these. The first thing we need is a way to measure and quantify the effect of the fix on GNOME Shell's performace, that takes both CPU time and memory usage into account, and is not just someone running Top - so that we can correctly and effectively tune the garbage collector's parameters for a long-lived process like GNOME Shell. Can you help answer the question of whether a particular branch makes things better or worse for users of GNOME Shell? We really need that!

    We also need help with a communication plan, so we can figure out how to set expectations for users who have been awaiting this fix for a long time.

    Can you help quantify this, or help communicate it? PLEASE help.
-->

---

<!-- _class: invert lead -->

# Community

---

<!-- _class: gaia -->

# Thanks

Nasah Kuma
GJS contributors from 44 and 45

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

<!--
    We'd especially like to thank Nasah Kuma for joining in with ideas for this talk, but unfortunately she couldn't be here to present.

    A big thank you as well to everyone who helped in any way with GJS in GNOME 44 and 45!

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
