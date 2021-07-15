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
  pre code { background-color: #042029; }
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
_class: invert lead
-->

# **Guidance for nested namespace objects**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 July 2021

<!--
-->

---

# Top-level namespace objects

- Should be capitalized (e.g. `Math`, `Temporal`)
- Should have a @@toStringTag ([June 2020 plenary](https://github.com/tc39/notes/blob/5a3580a5a057f3160338025acc2b674e03cc00a3/meetings/2020-07/july-20.md#adding-reflectsymboltostringtag-2057))

---

# Nested namespace objects

- `Temporal.now`? `Temporal.Now`?
- `Temporal.now[Symbol.toStringTag] == 'Temporal.now'`?

---

# Proposal

- Provide explicit guidance in how-we-work that both top-level and nested namespaces should
  - be capitalized
  - have a @@toStringTag equal to the fully qualified name

---

# Asking the plenary for (1/2)

- Consensus on the point**s** on the previous slide?
  - ✅ @@toStringTag equal to the fully qualified name
  - ❓ capitalization
    + Definition of namespace? "I know it when I see it"?
    + _"The convention for how they are capitalized or spelled usually is derived from the kind of thing they are, not where they are put."_ — Shu

---

# Asking the plenary for (2/2)

- Consensus on making this change in any current proposals?
- A volunteer to write this up in how-we-work?
