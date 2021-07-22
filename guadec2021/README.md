---
theme: gaia
paginate: true
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.14.0/css/all.min.css');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  code { font-family: Hack; }
  section { font-family: Cantarell, sans-serif; letter-spacing: 0; }
  section.lead.invert { text-shadow: 0 0 10px black, 0 0 20px black; }
  pre code { background-color: #042029; }
  ul ul li { font-size: 75%; }
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
_class: gaia lead
_footer: Photo by <a href="https://www.pexels.com/@lucaspezeta">Lucas Pezeta</a> from <a href="https://www.pexels.com/photo/tabby-cat-sticking-its-tongue-out-2930204/">Pexels</a>
-->

# What's new with JavaScript in GNOME

![bg right](wink.jpg)

## The 2021 edition

**Philip Chimento**
<i class="fab fa-rocketchat"></i> pchimento • <i class="fab fa-gitlab"></i>&thinsp;<i class="fab fa-github"></i> ptomato
<i class="fab fa-twitter"></i> @therealptomato
GUADEC Online, July 22, 2021

<!--
  Hi! My name is Philip Chimento. I've been a regular GNOME contributor for about 8 years. I used to like tabs, now I like spaces. You can probably also see that I like cat pictures. At my day job, I work on JavaScript engines at Igalia.

  Today I'll be talking about what's new with JavaScript in GNOME. This is the 2021 episode of a talk that has become somewhat of a tradition at GUADEC. I guess people must like JavaScript, or something!
-->

---

# Introduction: What we will talk about

- Same thing we talk about every year!
- **What's new** in GJS?
- **What's next** in GJS?
- We need some **help!** Here's what you can do

<!--
  As every year, this talk is primarily aimed at people who write code for the GNOME platform in the JavaScript programming language, whether that is GNOME Shell, apps, shell extensions, or even command line scripts.
  GNOME has its own JavaScript engine, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  The first half will be about what's new with GJS in GNOME 40 and 41 and how it can benefit you.
  The second half will be about what improvements you can expect to see coming up in GNOME 42.
  The third half will be about what we need help with and how you can get involved!
-->

---

# Introduction

- Presentation is full of [links](https://www.youtube.com/watch?v=dQw4w9WgXcQ) if you want to click through later
- If you want to follow along: **ptomato.name/talks/guadec2021**

<!--
  This presentation is also meant to be a resource that you can consult later. So I will be running pretty quickly through things like the new APIs, because they're not really that interesting to talk about; but you can go back and read them later and click through the links.
-->

---

<!-- _class: invert lead -->

# What's new in JavaScript land for GNOME 40 and 41?

<!--
    Part One. What's new for GNOME 40 and the soon-to-be-released GNOME 41?
-->

---

# Crash fixes

- ㊵ ㊶ 10 crashing bugs fixed in the past year
  - But also, 10 crashing bugs reported in the past year
  - [Total of 9](https://gitlab.gnome.org/GNOME/gjs/-/issues?label_name%5B%5D=1.+Crash) currently open
  - 2 older than 1 year
- ㊵ Many refactors adding type safety (🎩 Marco Trevisan)

<!--
    Crash fixes!
    By the way, I'll use these little circled 40 and 41 icons to show things that you can expect to see in GNOME 40 and 41. And I'll doff my hat icon to the person who contributed it.

    First of all, we have fixed just as many crashing bugs as were reported in the past year. Although that means we have the same number as we started out with, most of them are new.

    Marco did several refactors for GNOME 40 that increased type safety in the codebase with C++ features. I've done a few as well. These are not specifically crash fixes, but they decrease the likelihood that there are yet undiscovered crashes lurking in there.
-->

---

# Performance improvements ㊶

- Memory usage improvements (🎩 Marco Trevisan)
    + [!625](https://gitlab.gnome.org/GNOME/gjs/-/merge_requests/625) for objects
    + [!519](https://gitlab.gnome.org/GNOME/gjs/-/merge_requests/519) for functions

<!--
    Marco has also worked on a pair of pull requests, which should be ready to merge in time for GNOME 41, which not only increase type safety, but also reduce the amount of memory taken by introspected objects and functions. It's a savings of a few bytes per object, but when you have ten-thousands of objects alive in GNOME Shell, it actually makes a difference.
-->

---

# ES Modules ㊵

```js
// Old, but still works, not deprecated:
const System = imports.system;  // built-in module
imports.gi.versions.Gtk = '3.0';
const {Gtk} = imports.gi;  // G-I binding
imports.searchPath.unshift('resource:///com/example/MyApp');
const {MyClass, myFunction} = imports.src.myModule;
  // imports src/myModule.js from search path

// New 👍
import System from 'system';  // built-in module
import Gtk from 'gi://Gtk?version=3.0';  // G-I binding
import {MyClass, myFunction} from './src/myModule.js';
  // imports path relative to currently executing file
```

<!--
    In GNOME 40 we got ES Modules. Thanks to Evan for all the work on this.

    (ES stands for ECMAScript. They're called ECMAScript modules meaning they are the standardized module system for JavaScript, as opposed to the nonstandard module systems that many environments had previously.)

    Here is a before-and-after example of using the old nonstandard module system in GJS, and using ES modules. In both systems we have built-in modules, GObject-introspection modules, and modules consisting of other JavaScript files, but in ES modules they are a bit more distinct from each other.

    Another advantage of ES modules is that when you import from a relative path, you don't need to use some sort of hack to add the current file path to the module search path. Relative paths are always resolved relative to the path of the currently executing file. Even if that file is in a GResource.
-->

---

# ES Modules ㊵

- Run your main file with `-m`
  - or `gjs_context_eval_module()` if using the GObject-based API
- ES modules can import legacy modules, but not vice versa
  - To port your code, start at the top level and work downwards
- 🎩 Evan Welsh

<!--
    If you want to port your GJS program, run it with the dash-M command line flag, or if you have a C executable for your app then start your JavaScript code with gjs_context_eval_module instead of gjs_context_eval. The reason for this is that ES modules are technically a different type of JavaScript source, in which import and export statements are legal.

    For that reason, ES modules can use the old-style `imports` object to import legacy modules, but legacy modules cannot import ES modules with import statements, because those import statements are not legal in non-module sources. (You could still use dynamic imports, but that's inconvenient.)

    So, if you want to port your code from legacy modules to ES modules, then start at the top level and work downwards.

    The legacy module system is not going away, for sure. I would encourage new code to use ES modules, and we are not going to make any enhancements to the legacy module system in the future, but there is too much existing code that relies on it for it to go away. I expect there is going to be a long transition period before most GJS code is fully ported to ES modules, and I'll talk a bit about this in the next slide.

    In particular, porting is going to be a minor pain for Shell extensions, because they will not be able to port until GNOME Shell starts supporting ES module extensions. It should be possible to allow ES-module and non-ES-module extensions to exist side by side in GNOME Shell, but we'll need to take care in order to allow extensions to define whether they should be imported as an ES module or a legacy module.
-->

---

# ES Modules ㊵

- ES modules are always in [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)
- Importing is compile-time!
  - To import conditionally, use dynamic async `import()`

```js
// NOT POSSIBLE 🚫
if (usingSomeLibrary)
    import SomeLibrary from 'gi://SomeLibrary';

// Possible ✅
if (usingSomeLibrary)
    const SomeLibrary = await import('gi://SomeLibrary');
```

<!--
    Here are the ways that ES modules might not be straightforward to port from non-module source code.

    ES modules are always automatically in strict mode, so there may be some incompatibilities there. However, a lot of the strict mode errors are already discouraged by linters. The bodies of ES classes are also automatically in strict mode, and I'm not aware of much breakage due to that. But the one instance that I personally did see was obscure, so this is something to keep in the back of your mind if you start using ES modules and you see an obscure error.

    The other thing that you may have to take care to do differently is conditional imports. Unlike the legacy module system, resolving imports happens at compile time, not run time, so import statements are only legal at the top level. If you need to import something conditionally, use the built-in dynamic import function. This function is asynchronous, so it returns a Promise that needs to be await-ed. That means you'll have to refactor your code somewhat.
-->

---

# Debugger improvements ㊵

- Backtrace command that prints all the locals, like GDB
- Program listing
- Show line along with frame
- 🎩 Nasah Kuma

<!--
    Next major improvement in GNOME 40 on my list to talk about, is the work of Nasah, GJS's Outreachy intern from the December 2020 round. Look for her lightning talk in the intern lightning talk session on Friday at 1500 hours UTC!

    We've gotten several improvements to make the debugger more useful for debugging. The original debugger was quite basic. It had a backtrace command of course, but now it has a backtrace full option just like GDB, which prints all the information about the local variables in the stack frame.

    It also now has access to the source code, so there is a list command that also works like GDB, and when you show a frame it also shows the line of source code that goes along with it.

    I'll show you an example of what this looks like.
-->

---

# Debugger improvements ㊵

```js
function myFunction(a, b, c) {
    const d = Math.random();
    debugger;
}
myFunction(3, 2, 1);
```

<!--
    Here's a sample program to debug...
-->

---

# Debugger improvements ㊵

```
db> bt full
#0    myFunction(3, 2, 1) at bug.js:3:4
d = 0.7590159046381023
#1    toplevel at bug.js:5:10

db> list
   1    function myFunction(a, b, c) {
   2        const d = Math.random();
  *3        debugger;
   4    }
   5    myFunction(3, 2, 1);

db> frame 1
#1    toplevel at bug.js:5:10
   5    myFunction(3, 2, 1);
```

<!--
    ...and here's a demonstration of the new features. For the backtrace, you can see the value of the local variable `d`.
    The program listing can even highlight the current line, depending on the value of an option, although that's not visible here because I couldn't get it to highlight in a plain text markdown block.
-->

---

# JS Object GObject parameters ㊵

```js
Properties: {
    myArray: GObject.ParamSpec.jsobject('my-array', 'My array',
        'You can now use JS objects as the value of GObject properties',
        GObject.ParamFlags.READWRITE),
},
```
🎩 Marco Trevisan

<!--
    Next up I'll talk about JavaScript Object GObject parameters. That's too many Objects in one sentence!
    What it means is, on your GObject classes, you can now define properties that take a JavaScript object as their value, and you can have signals that take a JavaScript object as one of their parameters or their return value.

    This is probably most useful for having an Array as the value of a GObject property, since it's a bit tricky to define arrays within the GObject type system. But there are plenty of other JavaScript built-in objects that might be useful for this: Date, Map, Set, etc.

    Thanks to Marco for doing this!
-->

---

# Text encoding/decoding ㊶

```js
// Old, but still works, not deprecated:
const ByteArray = imports.byteArray;
const pizza = ByteArray.toString(Uint8Array.of(0xf0, 0x9f, 0x8d, 0x95) /*, 'utf-8' */);
const pizzaBytes = ByteArray.fromString('🍕' /*, 'utf-8' */);

// New 👍
const decoder = new TextDecoder(/* 'utf-8' */);
const pizza = decoder.decode(Uint8Array.of(0xf0, 0x9f, 0x8d, 0x95));
const encoder = new TextEncoder();
const pizzaBytes = encoder.encode('🍕');
```

🎩 Evan Welsh

<!--
    Something that I hope will be coming up in GNOME 41 is the TextEncoder and TextDecoder API, which is another initiative from Evan. This is technically a web API, not JavaScript, but NodeJS has it as well. It's good for us to have this, because it's a more standardized way to convert between strings and bytes. This is the reason that the new ES module system didn't include a ByteArray module, because Evan's idea was to not perpetuate the old way into the new module system.

    If you still want to use the old ByteArray module, you can, but you won't be able to import it as an ES module.
-->

---

# Memory profiling ㊶

![w:1000](sysprof.png)

<!--
    This is one of the things that I did during this cycle, as a step towards removing the Big Hammer. More about that later in the presentation.
    
    This is a screenshot from Sysprof that I took with a demonstration app that intentionally creates a lot of garbage very quickly, and includes a button to trigger a garbage collection.

    You can see various counters here, these are all new. Like the number of signal handlers and the number of GObjects. There are about a dozen of these counters now, that you can show or hide in Sysprof. The bottom two show some statistics from the JavaScript engine itself: how many bytes are used in garbage-collectable memory, and how many bytes allocated by malloc are owned by those garbage-collectable things.

    (The malloc bytes are a bit underreported right now, because the JavaScript engine doesn't know about all of our memory allocations in GObject. I'll talk more about this later.)

    Then down below you can see that the timing statistics on garbage collections have been enhanced. And they include information on why the garbage collection occurred.

    So this first one, you can see the reason is "API", meaning I clicked the button that calls the System.gc() API. Then you can see at the ten-second mark a "Big Hammer hit", and so on.
-->

---

# Console API ㊶

```js
// Old, but still works, not deprecated:
log('A message from your program');

// New, nice if you're used to browsers and Node:
console.log('A message from your program');
```

🎩 Evan Welsh

<!--
    Another project from Evan is to provide an implementation of the console specification. Like TextEncoder, this is another web API which is also implemented by NodeJS and it can be useful for sharing code. Most people know console.log but there are a bunch of other functions in the console namespace as well that can be useful.

    With a bit of luck, we can land this for GNOME 41.
-->

---

## Documentation and Examples

- 🎩 Extensions Rebooted

<!--
    Finally I want to give a special shout out to the Extensions Rebooted initiative which apart from all the other work they've been doing that's important for the GNOME Desktop, have also been gradually improving the documentation of GJS.
-->

---

<!-- _class: invert lead -->

# What is upcoming in JavaScript land for GNOME?

<!--
    That was a selection of some of the exciting new things that we either have already or will get soon.

    Now for Part Two. What are the exciting things that we should be able to see in the upcoming year in GNOME 42, or possibly 43?
-->

---

# Native async operations ㊶ ㊷

[Annotations](https://gitlab.gnome.org/GNOME/gobject-introspection/-/merge_requests/278):
```xml
<method name="load_contents_async" c:identifier="g_file_load_contents_async"
        glib:finish-func="g_file_load_contents_finish"
        glib:sync-func="g_file_load_contents">
```

```js
// Opt-in per method no longer needed:
// Gio._promisify(Gio._LocalFilePrototype, 'load_contents_async', 'load_contents_finish');
const [contents] = await file.load_contents_async(/* cancel = */ null);
```

🎩 Veena Nagar

<!--
    First off I'd like to talk about the work of Veena, GJS's Outreachy intern from the May 2021 round. She will also be giving a lightning talk about this, so if you want to know more, here's another reminder to go see the intern lightning talk session on Friday at 1500 hours UTC!

    Veena is working on adding annotations to gobject-introspection, such as finish-func and sync-func, that allow us to tie together these pairs of functions automatically.

    Previously we had to tie together these functions manually with Gio.promisify, in order to use async style programming. In the future, we'll be able to leave that out, like it shows here in the commented out code. Promisify was the work of Avi, our Outreachy intern from 2018, and I'm excited that it's being continued.

    The annotations are getting near to being ready in gobject-introspection. The corresponding changes in GJS are a stretch goal for the internship, so those may happen for GNOME 41 or later.
-->

---

# Next JS engine upgrade (Firefox 91) ㊷

### [`#private` class fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)

```js
class MyClass {
    #value;

    someMethod() {
        doSomethingWith(this.#value);
    }
}
```

**Note:** Doesn't [yet](https://gitlab.gnome.org/GNOME/gjs/-/issues/331) integrate with GObject classes.

<!--
    Evan has been working on upgrading the JavaScript engine to the next long-term-support release from Firefox. Luckily, there are no backwards incompatibilities that you have to remove from your code, this time, that we know of, yet!

    I'll talk a bit about the new language features this will bring us. One thing that we get is private class fields. If you have a JavaScript class, you can now have truly encapsulated fields that cannot be accessed outside the class.

    Just like the public class fields that we already got a year ago, this doesn't yet integrate well with GObject classes. We hope to solve this in a future release.
-->

---

# Next JS engine upgrade (Firefox 91) ㊷

#### `at()`: Python-style indexing for [arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at), [strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/at), and [typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/at)

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

# Next JS engine upgrade (Firefox 91) ㊷

### [`Promise.any()`: First successful sub-promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)

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

# Next JS engine upgrade (Firefox 91) ㊷

### `??=`, `&&=`, `||=` [operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment)

Short-circuiting assignment operators
- `a ??= b` - assign `a = b` if `a` is `null` or `undefined`
- `a ||= b` - assign `a = b` if `a` is falsey
- `a &&= b` - assign `a = b` if `a` is truthy

<!--
    We also get a bunch of new operators, that can be used to assign default values to a variable. They short-circuit, so the right hand side is not evaluated if the left-hand side already determines that the assignment doesn't take place.
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

GJS contributors from 40 and 41

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

<!--
    On that note, I'd like to end by acknowledging everyone who helped in any way with GJS in GNOME 40 and 41!

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
