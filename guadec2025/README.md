---
theme: gaia
paginate: true
style: |
  @import url('https://fonts.googleapis.com/css2?family=Cantarell:ital,wght@0,400;0,700;1,400;1,700&display=swap');
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css');
  @import url('https://cdn.jsdelivr.net/npm/hack-font@3/build/web/hack-subset.css');
  code { font-family: Hack; }
  section { font-family: Cantarell, sans-serif; letter-spacing: 0; }
  section:not(.lead):not(.gaia):not(.invert) {
    background-color: white;
    background-image: linear-gradient(90deg, rgba(216, 216, 216, .03) 50%,
                                      transparent 50%),
      linear-gradient(90deg, rgba(216, 216, 216, .06) 50%, transparent 50%),
      linear-gradient(90deg, transparent 50%, rgba(216, 216, 216, .08) 50%),
      linear-gradient(90deg, transparent 50%, rgba(216, 216, 216, .1) 50%);
    background-size: 26px, 58px, 74px, 106px;
    color: black;
  }
  section.lead.invert { text-shadow: 0 0 10px black, 0 0 20px black; }
  section.smaller-type li, ul ul li { font-size: 85% }
  marp-pre code, marp-pre { background-color: #042029; }
  .twocolumn { display: flex; }
  .col { flex: 1 }
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
  .time {
    background-color: black;
    color: white;
    font-size: 150%;
    padding: 0.2em;
    text-decoration: underline 2px red;
  }
marp: true
---

<!--
_class: gaia lead
-->

# Title

**Philip Chimento**
<i class="fa-solid fa-message"></i> @ptomato:gnome.org
<i class="fab fa-gitlab"></i> ptomato
<i class="fab fa-bluesky"></i> @ptomato.name
<i class="fab fa-mastodon"></i> mstdn.ca/@ptomato

GUADEC, July X, 2025

---

TODO

- Use new GNOME fonts
- 48 and 49 icons
- `using` statements
- Temporal?
- MessageFormat for translations
- Show nonograms app

---

# Introduction: What this talk is about

- Using **TypeScript** to write GNOME apps
- **What's new and what's next** in GJS?
- Some cool **project ideas** for you

(...project ideas)
- write tests for import maps
- figure out what Gio objects should get `[Symbol.dispose]` methods
- port the date components and calendar in gnome-shell to Temporal
- figure out how to call g-i functions in jit?

<!--
  Today we'll be talking about what's new and what's next with JavaScript in GNOME. Evan will be 
-->

---

# Introduction

- Presentation is full of links if you want to click through later
- If you want to follow along: [**ptomato.name/talks/guadec2025**](https://ptomato.name/talks/guadec2025)
- ㊽, ㊾, 🎩

<!--
  This slide deck is also meant to be a resource that you can consult later. The slides are already available on Philip's web space so if you want to click on the links NOW, you can already go there and follow along with the presentation.

    By the way, I'll use these little circled 48 and 49 icons to show things that you can expect to see in GNOME 48 and 49. And when you see a hat emoji, that's me tipping my hat to the person who contributed it.
-->

---

<!-- _class: lead invert -->

# Apps in TypeScript

---

# How to use TypeScript today

A massive shoutout to Chris Davis for working continuously on a [TypeScript template for GJS](https://gitlab.gnome.org/World/javascript/gnome-typescript-template)

Most apps should start there!

---

# How to use TypeScript today (experimental)

If you're developing Shell extensions or are adventurous, the beta bindings are [currently published as NPM packages](https://github.com/gjsify/ts-for-gir?tab=readme-ov-file#npm-packages)

⚠️ Distro packaging with NPM packages is not simple, so proceed with caution for apps. ⚠️

We're working to support an ideal format for the TypeScript template and distro packaging, reach out if you want to help!

---

# More Community initiatives

_We have a GNOME-home for JavaScript projects now!_

## gjs.guide

gjs.guide has a new (more official) home at https://gitlab.gnome.org/World/javascript/gjs-guide

## the TypeScript template

The TypeScript template is at https://gitlab.gnome.org/World/javascript/gnome-typescript-template

---

# gjs.guide

- Updated and upgraded with better search and easier navigation
- Docs written this year...
  - Extensions, the Shell, and GNOME 46 (🎩 Andy, Javad)
  - D-Bus (🎩 Andy)
  - St widget examples (🎩 Nitin Dhembare)

🎩 _to Sebastian Wiesner, Daniel Steele, Adrien Delessert,  Evangelos Paterakis, "Bytez", Jeffery To, Marko Kocic, Pedro Sader Azevedo and many more for adding examples, fixing typos, and making the community better_ 💙 

---

# Workbench

![width:1100px drop-shadow](workbench.png)

<!--
    There is a library of demos and examples, similar to GTK demo but more aligned with the HIG and less "benchmarks" / test cases
    with examples ready to use, copy paste and play with
    I will talk more about it tomorrow in the  GNOME JavaScript Tooling talk
    also, there will be a workbench workshop on Saturday
-->

---

# Workbench improvements

<div class="twocolumn">
<div class="col">

- TypeScript support!
- Write JS/TS code in Workbench with autocomplete support using the language server!
- 🎩 Angelo Verlain

</div>
<div class="col">

![drop-shadow](typescript.png)

</div>
</div>

---

<!-- _class: invert lead -->

# What's new in GJS

## for GNOME ㊽ and ㊾?

<!--
    What's new for GNOME 48 and the upcoming GNOME 49?



  This part of the talk is primarily aimed at people who write code for the GNOME platform in the JavaScript programming language, whether that is GNOME Shell, apps, shell extensions, or even command line scripts. GNOME has its own JavaScript engine for all these purposes, GJS, which is an extended version of the JavaScript engine from the Firefox browser.

  In past years I talked a lot in the "what's new and what's next" sections about the various ways to modernize your code, but nowadays we are running a pretty modern JavaScript in GNOME already, with no modernization "backlog". The new language features are still pretty exciting, but there's nothing earthshaking like ES6 classes. Additionally, we have a lot more visibility of these improvements these days, with better documentation, a more active community of GJS developers, and initiatives like This Week in GNOME, so a yearly GUADEC talk is no longer the only channel where people find out about these things. Nonetheless, I've gotten feedback that people still find these talks useful, so that's a good reason to keep doing them in addition to all those other things I mentioned.
-->

---

---

# JS engine upgrades ㊽ ㊾

GJS's underlying JavaScript engine is the one from Firefox, but embedded.

- Tracks Firefox's long-term support version (currently Firefox ESR 128)
- Brings in newer editions of the JS language, and security updates
- GNOME ㊾ will upgrade to Firefox ESR 140

<!--
    I almost always have a section on what JavaScript language features are new, when we upgrade the version of the underlying JavaScript engine to one from a newer Firefox. This year we're planning to upgrade to Firefox 140 in GNOME 49.
-->

Float16Array
duplicate named capturing groups
iterator methods: drop, every, filter, find, flatMap, forEach, map, reduce, some, take
regex pattern modifiers
base64: Uint8Array.fromBase64, Uint8Array.fromHex, setFromBase64, setFromHex, toBase64, toHex
RegExp.escape
Promise.try
JSON.parse with source: context arg, JSON.isRawJSON, JSON.rawJSON
Intl.DurationFormat
Math.sumPrecise
Atomics.pause
Error.captureStackTrace
Error.isError
JSON imports
Temporal


---

# New Set methods

union
intersection
...

---

# Resizable ArrayBuffer

---

# "using"

streams?
cr.$dispose()?
anything you want to be dropped when you're done with the variable, instead of waiting for it to be garbage collected

<!--
Using statements are kind of like context managers and with-blocks in Python. They'll be quite useful for explicitly disposing resources, like disconnecting signals and such, which is currently a bit of a hassle in GJS. Relying on it implicitly often runs into problems with callbacks being run during garbage collection. "Using" should help 
-->

---

<!-- _class: invert lead -->

# What's coming down the line in JS

<!--
This part overlaps with what I do in my day job at Igalia, as a member of the TC39 committee that standardizes the JavaScript language. I'm excited to present some of these!
-->

---

## Already implemented in non-LTS or nightly Firefox

- `Temporal` (I gave a [talk](https://ptomato.name/talks/globalscope2021/) on it if you want to know more)
- We'll get these in next year's SpiderMonkey update

<!--
Excited to see Temporal finally coming to SpiderMonkey as I've been personally working to bring this proposal to JavaScript for the past couple of years! Especially together with Intl which I'll talk about in a bit, I'm excited to use it in GNOME Shell's JS code at some point.
-->

---

## TC55

- [Common Minimum API Proposal](https://common-min-api.proposal.wintercg.org/) (update link)
- Any interest in joining this group on GNOME's behalf?

---

## Other exciting proposals coming 2026 or 2027

- Decorators

<!--
There are many more proposals in the pipeline, but these are the ones I'm particularly looking forward to or that I think are relevant to GNOME.

Decorators we've been eagerly awaiting for the better part of a decade! - I first wrote a blog post on designing a decorator API for GObject classes in July 2017.

If you've written code in Rust or Python, you probably have gotten used to iterators. Iterators exist already in JS but without all the neato methods 
-->

---

<!-- _class: gaia -->

# Thanks

GJS contributors from 48 and 49

# License

Presentation licensed under Creative Commons BY-NC-ND 4.0

<!--
    A big thank you as well to everyone who helped in any way with GJS in GNOME 48 and 49!

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
