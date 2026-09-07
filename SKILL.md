---
name: turystack-frontend-pattern
description: "How a Turystack web or mobile frontend is written — project structure and ownership, the generated SDK and server state, business components and client state, routes, app shell and resource deep links, forms, tables and detail surfaces, the loading/empty/partial/error/denied states, blocked actions left inert with their reason, blast radius before a destructive confirm, accessibility, copy and i18n, responsive density, error handling, permissions, performance, uploads, tests and telemetry. Use it whenever you create, change or review frontend code in a Turystack app: a screen or route, a form, a table, a filter, a modal, a permission-gated action, an empty or error state, a deep link — including when the request sounds routine, like 'add a column', 'show this field' or 'hide this button', because those are the ones that quietly break a law. Read turystack-architecture-pattern first. For how a single UI primitive is built, use turystack-frontend-primitives-pattern."
---

# turystack-frontend-pattern

Use this skill when creating, changing or reviewing a web/mobile frontend.

## Prerequisite

`turystack-architecture-pattern` is the constitution and comes first. The law
lives there; this skill is how the frontend expresses it. A constitutional law
is **never restated here** — it is cited by its id, which always starts with
`ARC-` (`ARC-CTR-7`, `ARC-DEL-8`, `ARC-ERR-8`, `ARC-ERR-9`…). When the two
disagree, the constitution wins.

Local ids (`STR`, `API`, `COM`, `RTE`, `FRM`, `TBL`, `UST`, `ACC`, `TXT`, `RSP`,
`ERR`, `PRM`, `TST`, `TEL`, `PRF`, `UPL`) are scoped to this skill. The backend skill has its
own `ERR`, `TST` and `TEL`, and the two never cross.

Most of what feels like a "frontend rule" is constitutional: where state lives,
who invalidates a cache, how many outcomes a read has, what a denied capability
looks like, what a destructive write owes the user. Reading the constitution
first is what keeps those decisions from being reinvented per screen.

## How to use

1. Detect web or mobile in `00-overview.md`. The domain, contract, state and UX
   decisions are shared; only the navigation runtime and the concrete primitives
   change.
2. Read `01-project-structure.md`. It also carries the *Which library owns which
   concern* table: check it **before** hand-writing a hook, a formatter or a
   primitive, because most of them already have an owner (`ARC-LAY-8`).
3. Read the sections your task touches, from the routing table below — the whole
   section, not a remembered summary of it.
4. Check the library documentation for props, components, hooks and setup. Never
   invent a prop: read the primitive's real types.

## How a section is written

Every section splits in two, and the split is the point:

- **🌐 Generic pattern** — the portable law. Each rule has a stable id and, when
  the reason is not obvious, a paragraph explaining *why*. It closes with the
  **Invariants** table, which is what a review binds to, and a **Governed by the
  constitution** table mapping each `ARC-…` law to its frontend expression.
- **🛠️ Project-specific** — the same rules as TypeScript · React · TanStack ·
  Expo · @turystack code: mechanisms per rule, ✅ scenarios, and a **❌ Never
  do** block where every item carries the id it violates.

`XXX-n` is **constitutional** (it would survive a stack swap); `XXX-Ln` is a
**stack lint** (it exists because of this toolchain and would invert elsewhere —
still enforced here). Comments inside the examples are didactic: they explain
the rule, and never belong in real code.

The **Invariants** table is read column by column: the **id** a review binds
to, the **law** in one line, its **class** (`constitutional` or `stack lint`),
the **gate** that checks it, and the **detector** — how that gate catches the
violation, which is stack-specific and therefore lives in the 🛠️ half.

Every section opens with a **Rules defined here** line naming the ids it owns,
so you can confirm you opened the right file before reading it. A section
that says `none` states no law of its own — everything in it is cited.

## Routing

| Touching | Read |
|---|---|
| Structure, owner, imports, which library owns a concern | `01-project-structure.md` |
| SDK, HTTP, query, mutation, cache, real-time channel | `02-sdk.md` |
| Feature boundary, business component, local primitive or client state | `03-components-client-state.md` |
| Unique field, availability check, "is this name already taken" | `03-components-client-state.md` (`COM-10`) |
| Route, screen, params, navigation, deep link (`resourceId`), providers or shell | `04-routes-app-shell.md` |
| Form | `05-forms.md` |
| Table, toolbar, list or detail surface | `06-data-surfaces.md` |
| Loading, overlay, empty, error, feedback, blocked action, denied surface, blast radius | `07-ui-states-and-feedback.md` |
| Any query whose result reaches the screen — the outcome and who paints it | `07-ui-states-and-feedback.md` (`UST-10`, `UST-11`) |
| Keyboard, ARIA and focus | `08-accessibility.md` |
| Copy, labels, placeholders and language | `09-content-i18n.md` |
| Breakpoints, density and touch targets | `10-responsive-density.md` |
| Exception, feedback and error boundary | `11-error-handling.md` |
| Auth, permission, secrets, XSS and PII | `12-security-permissions.md` |
| Unit, render, interaction, a11y and e2e | `13-testing.md` |
| Error reporting, web vitals and events | `14-telemetry.md` |
| Code splitting, list windowing, media budget, cache staleness | `15-performance.md` |
| File upload, transfer states, stored-file lifetime | `16-file-uploads.md` |

## Before you finish

1. **Five outcomes.** Does every asynchronous surface you touched decide
   pending, denied, error, empty and success — derived once with
   `useDataOutcome` and handed to the surface, never restated as a chain of
   `if`s and never fed a `?? []`? (`ARC-ERR-8`, `UST-10`, `UST-11`)
2. **Denial.** Is any action or surface hidden because of a permission, a plan,
   an entity state or an unmet dependency, instead of inert with its reason?
   (`ARC-ERR-9`, `PRM-2`)
3. **Address.** Does state that must survive a reload or a shared link live in
   the URL — including the identity of a resource opened in an overlay?
   (`ARC-DEL-8`, `ARC-DEL-9`)
4. **Derived data.** Is server data still read from the cache rather than copied
   into local state, and does every mutation invalidate what it made stale?
   (`ARC-CTR-7`, `ARC-CON-9`)
5. **Contract.** Any type, permission string or endpoint shape written by hand
   that the generated SDK already declares? (`ARC-CTR-1`)
6. **Destructive writes.** Does the confirmation show what else the action
   reaches, before it runs? (`ARC-CON-11`)
7. **Primitives.** Did you reach for parallel HTML or a `className` escape hatch
   instead of extending the primitive? (`COM-5`)
8. **Accessibility.** Role and accessible name present, focus handled, `axe`
   clean in the component test? (`ACC-*`, `TST-6`)
9. **Uniqueness.** Is a "is this already taken" answered by the contract's
   availability read inside its own input component — advisory, failing open —
   rather than by a listing counted in the consumer? (`COM-10`, `ARC-CON-4`)

Each id above carries a gate binding in its Invariants table, and
`turystack-proof` prints this same list with real pass/fail — `manual` bindings
stop for a person to sign. Binding kinds: `turystack-architecture-pattern` ›
`00-overview.md` › *Gate*.

## Ownership rule

This skill answers **which decision to apply and where the code belongs**. The
libraries and `turystack-frontend-primitives-pattern` answer **how the
primitive/hook is implemented and which props exist**. Do not replicate their
READMEs here.
