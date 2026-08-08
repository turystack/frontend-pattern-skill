---
name: tury-stack-frontend-pattern
description: "Frontend application constitution for Turystack web and mobile apps. Defines ownership, generated API boundaries, components, client state, routes, forms, data surfaces, async UX, accessibility, content, responsive behavior, errors, security, testing and telemetry. Libraries own setup and component APIs; this skill owns decisions and invariants."
---

# tury-stack-frontend-pattern

Use this skill when creating, changing or reviewing a web/mobile frontend.

## Order

1. Detect web or mobile in `00-overview.md`.
2. Read `01-project-structure.md`.
3. Read only the sections the task touches.
4. Check the library documentation for props, components, hooks and setup.

## Routing

| Touching | Read |
|---|---|
| Structure, owner, imports and conditional folders | `01-project-structure.md` |
| SDK, HTTP, query, mutation, cache and real-time | `02-sdk.md` |
| Feature boundary, business component, local primitive or client state | `03-components-client-state.md` |
| Route, screen, params, navigation, providers or shell | `04-routes-app-shell.md` |
| Form | `05-forms.md` |
| Table, toolbar, list or detail surface | `06-data-surfaces.md` |
| Loading, overlay, empty, error and feedback | `07-ui-states-and-feedback.md` |
| Keyboard, ARIA and focus | `08-accessibility.md` |
| Copy, labels, placeholders and language | `09-content-i18n.md` |
| Breakpoints, density and touch targets | `10-responsive-density.md` |
| Exception, feedback and error boundary | `11-error-handling.md` |
| Auth, permission, secrets, XSS and PII | `12-security-permissions.md` |
| Unit, render, interaction, a11y and e2e | `13-testing.md` |
| Error reporting, web vitals and events | `14-telemetry.md` |

## Ownership rule

The skill answers **which decision to apply and where the code belongs**. The
libraries and the `frontend-primitives-pattern` skill answer **how the
primitive/hook is implemented and which props exist**. Do not replicate their
READMEs here.
