---
marp: true
theme: gaia
paginate: true
style: |
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
---

<!--
_class: invert lead
footer: <span style="color:red;font-weight:bold">DRAFT</span>
-->

# 🌒 **ShadowRealm** 🌘
## for Stage 3

**Philip Chimento**
Igalia, in partnership with Salesforce  
TC39 December 2024  
**DRAFT** - contains info expected to be outdated by the meeting date

---

# Recap: What is ShadowRealm?

A mechanism to execute JavaScript code within the context of a new global object and set of built-ins.

Goal is not **security**, but **integrity**: complete control over the execution environment.

```js
globalThis.someValue = 1;
const realm = new ShadowRealm();
realm.evaluate(`globalThis.someValue = 2;`);
console.assert(globalThis.someValue === 1);
```

---

<!-- _backgroundColor: white -->

![bg contain](shadowrealm-according-to-chatgpt.png)

---

# Recap: Proposal history

- Everything revolves around the question [**Which web APIs should be present inside ShadowRealm**](https://github.com/tc39/proposal-shadowrealm/issues/393)?
- Several different ways of answering this question have been proposed and rejected:
  - "None"
  - A vetted list
  - A criterion based on confidentiality

---

# Recap: Proposal history

- **Sept. 2023**: Stage 2, readvancement pending explicit support from two implementations that the testing and list of APIs to be exposed to ShadowRealms are sufficient
- **Feb. 2024**: Stage 2.7, with the understanding that stage 3 requires signoff from HTML folks on the HTML integration, as well as resolution of Mozilla's concerns about test coverage
- Not an opportunity to relitigate previous design decisions

---

# Today: Exposed web APIs

**Which web APIs should be present inside ShadowRealm?**

- New W3C TAG [design principle](https://github.com/w3ctag/design-principles/pull/510):
  - Only purely computational features are exposed everywhere
    - not performing I/O
    - not affecting the state of the user agent or the user's device
  - Exception: Anything relying on an event loop is not exposed
  - Expose conservatively: features primarily useful for unexposed feature are not exposed

---

# Today: Exposed web APIs

- Developed based on a number of conversations with implementors and web platform experts
- Clear criterion to decide whether something is in or out
- Distinction between "which standards body standardized a feature" is irrelevant

---

# Today: HTML integration

- Pull request: [whatwg/html#9893](https://github.com/whatwg/html/pull/9893)
- **TBD what to say here**

---

# Today: Test coverage

- ✅ [Test APIs in ShadowRealms created in multiple scopes](https://github.com/web-platform-tests/wpt/issues/48573)
  - ✅ Window
  - ✅ Worker
  - ✅ SharedWorker
  - ✅ ServiceWorker
  - ✅ AudioWorklet
  - ✅ other ShadowRealm

---

# Today: Test coverage

- ✅ [AbortController](https://github.com/web-platform-tests/wpt/pull/41965)
- ✅ [AbortSignal](https://github.com/web-platform-tests/wpt/pull/41965) (except `AbortSignal.timeout`, not exposed)
- ✅ [addEventListener](https://github.com/web-platform-tests/wpt/pull/41966) - pending review
- ✅ [atob](https://github.com/web-platform-tests/wpt/pull/49188)
- ✅ [btoa](https://github.com/web-platform-tests/wpt/pull/49188)
- ✅ [ByteLengthQueuingStrategy](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [CompressionStream](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ console ([1](https://github.com/web-platform-tests/wpt/pull/34361), [2](https://github.com/web-platform-tests/wpt/pull/49283) - pending review)

---

# Today: Test coverage

- ✅ [CountQueuingStrategy](https://github.com/web-platform-tests/wpt/pull/42005)\*
- crypto.getRandomValues
- crypto.randomUUID
- ✅ [CustomEvent](https://github.com/web-platform-tests/wpt/pull/41966) - pending review
- ✅ [DataCloneError](https://github.com/web-platform-tests/wpt/pull/49282) - pending review
- ✅ [DOMException](https://github.com/web-platform-tests/wpt/pull/49159)
- ✅ [dispatchEvent](https://github.com/web-platform-tests/wpt/pull/41966) - pending review
- ErrorEvent

---

# Today: Test coverage

- ✅ [Event](https://github.com/web-platform-tests/wpt/pull/41966) - pending review
- ✅ [EventTarget](https://github.com/web-platform-tests/wpt/pull/41966) (including `globalThis` being one) - pending review
- onerror
- onrejectionhandled
- onunhandledrejection
- PromiseRejectionEvent
- queueMicrotask
- ✅ [ReadableByteStreamController]()\*

---

# Today: Test coverage

- ✅ [ReadableStream](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [ReadableStreamBYOBReader](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [ReadableStreamBYOBRequest](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [ReadableStreamDefaultController](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [ReadableStreamDefaultReader](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [removeEventListener](https://github.com/web-platform-tests/wpt/pull/41966) - pending review
- ✅ [self](https://github.com/web-platform-tests/wpt/pull/49252) - pending review
- ✅ [structuredClone](https://github.com/web-platform-tests/wpt/pull/49282) - pending review

---

# Today: Test coverage

- ✅ TextDecoder ([1](https://github.com/web-platform-tests/wpt/pull/41968), [2](https://github.com/web-platform-tests/wpt/pull/49286) - pending review)
- ✅ TextDecoderStream ([1](https://github.com/web-platform-tests/wpt/pull/41968), [2](https://github.com/web-platform-tests/wpt/pull/49286) - pending review)
- ✅ TextEncoder ([1](https://github.com/web-platform-tests/wpt/pull/41968), [2](https://github.com/web-platform-tests/wpt/pull/49286) - pending review)
- ✅ TextEncoderStream ([1](https://github.com/web-platform-tests/wpt/pull/41968), [2](https://github.com/web-platform-tests/wpt/pull/49286) - pending review)
- ✅ [TransformStream](https://github.com/web-platform-tests/wpt/pull/42005)\*
- TransformStreamDefaultController\*
- ✅ [URL](https://github.com/web-platform-tests/wpt/pull/41985) - pending Interop question
- URLPattern

---

# Today: Test coverage

- ✅ [URLSearchParams](https://github.com/web-platform-tests/wpt/pull/41985) - pending Interop question
- ✅ WebAssembly ([1](https://github.com/web-platform-tests/wpt/pull/36412), [2](https://github.com/web-platform-tests/wpt/pull/41987)) (except `compileStreaming` and `instantiateStreaming`, not exposed)
- ✅ [WritableStream](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [WritableStreamDefaultController](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ [WritableStreamDefaultWriter](https://github.com/web-platform-tests/wpt/pull/42005)\*
- ✅ Remove any tests for things that were previously thought to be exposed

---

# Requirements for stage 3

- ✅ The feature has sufficient testing and appropriate pre-implementation experience 

**Specific to ShadowRealm**
- Explicit support from two implementations that the testing and list of APIs to be exposed to ShadowRealms are sufficient
- Signoff from HTML folks on the HTML integration
- Resolution of Mozilla's concerns about test coverage

---

<!-- _class: lead -->

# Questions?

---

<!-- _class: lead -->

# Consensus to move the proposal to stage 3?

---

# Proposed summary for notes

> **TBD** what to say here
