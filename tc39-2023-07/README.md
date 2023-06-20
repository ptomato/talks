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
---

<!-- _class: invert lead -->

# ⌚ **Temporal**

**Philip Chimento**
Igalia, in partnership with Bloomberg  
TC39 July 2023

---

## Progress update

- Presenting integer arithmetic change discussed in previous plenaries
- Also presenting two normative changes arising from implementor discussions
- **All known discussions settled**; these are the last expected changes
- Implementation continues (JSC, LibJS, SpiderMonkey, V8)

---

## IETF standardization progress ([#1450](https://github.com/tc39/proposal-temporal/issues/1450))

- IETF document currently in last call
- Some suggestions from area directors, and one complaint
- Complaint seems to be able to be handled with discussion

---

<!-- _class: invert lead -->

# Normative changes

---

### Integer math in durations (PR [#2612](https://github.com/tc39/proposal-temporal/pull/2612)) (1/3)

- Storage doesn't change; one f64-representable integer for each unit
    - "90 minutes" preserved unless explicitly balanced to "1 hour 30"
- Calculations with time units (days through nanoseconds) use "normalized form", _s_ × 10⁹ + _ns_
    - |_s_| ≤ max safe integer
    - |_ns_| ≤ 999,999,999
    
---

### Integer math in durations (2/3)

- Date units years, months, weeks, all limited to maxuint32
    - |_N_| < 2³² for _N_ = _y_, _mon_, _d_
    - Why not maxint32? Because sign is common to all units
- With limits, calculations in loops no longer necessary
    - Concern emerged in review of SpiderMonkey implementation

---

### Integer math in durations (3/3)

- Spec text encapsulates all 96-bit operations in AOs
- Can be changed editorially to explicitly be 64+32 or seconds+subseconds, if implementations prefer
- Few observable effects, but some durations no longer allowed

```js
// e.g. no longer allowed:
Temporal.Duration.from({ seconds: Number.MAX_VALUE })
Temporal.Duration.from({
  seconds: Number.MAX_SAFE_INTEGER,
  milliseconds: 1000
})
```

---

### Limit precision of offset time zones (PR [#2607](https://github.com/tc39/proposal-temporal/pull/2607))

- Offset time zones: e.g., `"+01:00"`
- Previously allowed up to ns precision: `"+01:00:00.000000001"`
- IXDTF (the IETF string format) does not actually support &lt;minutes precision in these; we overlooked this
- Better to limit here and expand later, than change IETF at this point
- Offsets of named time zones can still be &lt; minutes (e.g. `Africa/Monrovia`)

```js
Temporal.TimeZone.from("+01:00:01")  // no longer allowed
```

---

### Adjustment of string coercion (PR [#2574](https://github.com/tc39/proposal-temporal/pull/2574)) (1/2)

- Some Numbers are valid ISO strings (e.g. `20230711`)
- Some are not (e.g. `05000101` equals 1310785)
- Concern found through Anba's test cases for Firefox
- Proposed change: don't use ToString, require primitives to be String
- Throw TypeError ("wrong type"), not RangeError ("converted to a disallowed string")

---

### Adjustment of string coercion (2/2)

Examples:

```js
Temporal.Calendar.from(10).id
    // Current spec text: "iso8601" (parsed as TemporalTimeString)
    // Proposed: from() throws TypeError
Temporal.TimeZone.from(-10).id
    // Current spec text: "-10:00"
    // Proposed: from() throws TypeError
Temporal.PlainDate.from(20230711)
    // Current spec text: PlainDate of July 11, 2023
    // Proposed: throws TypeError
Temporal.PlainDate.from(true)
    // Current spec text: throws RangeError ("true" is not a valid ISO string)
    // Proposed: throws TypeError
```

---

<!-- _class: invert lead -->

# Questions?

---

<!-- _class: lead -->

# Requesting consensus

On the normative changes just presented

---

# Proposed conclusion for the notes

> Consensus on making normative changes to:
> - Remove arbitrary-precision integer math and calculations in loops (PR [#2612](https://github.com/tc39/proposal-temporal/pull/2612))
> - Limit offset time zones to minutes precision (PR [#2607](https://github.com/tc39/proposal-temporal/pull/2607))
> - Require ISO strings and offset strings to be Strings (PR [#2574](https://github.com/tc39/proposal-temporal/pull/2574))
