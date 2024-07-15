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
---

<!-- _class: invert lead -->

# 🗣️ **ShadowRealm**

**Philip Chimento**
Igalia, in partnership with Salesforce  
TC39 June 2024

---

## Brief update

- **Last time:**
  - Described a criterion for including web APIs ("confidentiality")
  - HTML integration PR (html#9893) is open and waiting for review
- **What happened since then:** We collected feedback
  - We've heard from HTML reviewers: the "confidentiality" language is confusing
  - We've heard concerns from Mozilla about difficulties with WPT coverage
  - More feedback is welcomed; we want to have html#9893 in a mergeable state ASAP
- **What's next:**
  - Will investigate ways to simplify the criterion of what web APIs are included
  - Will continue to add WPT coverage including addressing Mozilla's concerns

---

<!-- _class: lead -->

# Questions?
