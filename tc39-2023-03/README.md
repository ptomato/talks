---
theme: gaia
paginate: true
style: |
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.14.0/css/all.min.css');
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
  ul ul li { font-size: 75%; }
---

<!--
_class: invert lead
-->

# Test262 — Funding Situation

**Philip Chimento**
Igalia, in partnership with Google  
TC39 March 2023

---

## Why is test262 important?

- Important effort which helps all of us
  - Ensure interoperability between implementations
  - Prevent bugs in implementations
  - Find bugs in proposals before implementation starts

---

## Test262 effort comes from

- Maintenance:
    - Contracted maintainers (first Bocoup, now Igalia): 0.4 FTE
    - Maintainers with paid time from employer
    - Volunteer maintainers

- Test writing:
    - Maintainers
    - Implementors
    - Proposal authors
    - Community contributors

---

## Test262 by numbers in 2022

- Commits: 453
- PRs merged: 299
- New tests: 3561
- Average time from PR creation to merge: 8.5 days

---

## Other achievements in 2022

- Creation of maintainers group, with governance policies
- RFC process
- Staging directory

---

## Funding of maintainers

- The contracted maintainers were funded by Google
- Per 1 April, Google can no longer do this

---

## What does this mean for TC39?

- Coverage of proposals with funding will continue
    - Recent examples: Temporal, Change Array by copy
- Coverage of proposals without funding will move slower
    - Recent examples: Array.fromAsync, Duplicate named capture groups
    - Some of this burden likely absorbed by proposal champions, i.e. **YOU**
- PR reviews & other maintenance will move _much_ slower
    - Reliant on maintainers with limited and/or unpaid time

---

### Possible solutions in absence of funding

- Continue with reduced involvement from maintainers
    - More work for proposal authors, proposals move slower
    - Process change to require test262 in earlier stages? (not proposing this)
- Make test262 more like WPT
    - Bar is lower for contributions, coverage is best-effort
- More paid contributors
    - Take this message back to your company if you can!
- ...?

<!--
    - Process change - discussion would not fit in the time slot we have for this, and neither Igalia nor Google are proposing it now, but we can consider it if someone else feels it's the right move to make
    - Shu to talk about pros and cons of WPT-style here
-->

---

<!-- _class: invert lead -->

## Discussion

<!--
    goal is to hear opinions and brainstorm additional ideas, not necessarily reach a consensus
-->
