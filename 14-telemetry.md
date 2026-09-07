# Telemetry

**Concept.** Client telemetry covers failures, real-world experience and product
events without duplicating handlers or leaking data.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> React Native. The split, `XXX-n` versus `XXX-Ln`, and why an `ARC-…` law is
> cited and never restated: `turystack-frontend-pattern` › *How a section is
> written*. `TEL-n` here is the frontend's namespace — the backend skill has
> its own `TEL-n`, and the two never cross.

---

**Rules defined here:** `TEL-1` · `TEL-2` · `TEL-3` · `TEL-4` · `TEL-7` ·
`TEL-8` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

**Retired ids:** `TEL-5` · `TEL-6` — retired, not renumbered. A review or
commit citing one points at a rule that no longer exists; the number is never
reused.

## 🌐 Generic pattern (portable — stack-independent)

### Invariants (the law the gates enforce)

| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| TEL-1 | The provider is initialized once, at bootstrap | constitutional | `manual` | Placement / ❌ |
| TEL-2 | The root error boundary reports before rendering the fallback | constitutional | `manual` | Error reporting / ❌ |
| TEL-3 | Global and unhandled errors are captured once; no duplicate capture when the vendor already does it | constitutional | `manual` | Error reporting / ❌ |
| TEL-4 | An API error keeps its catalogue code and status for correlation | constitutional | `grit:log-level` | Error reporting / ❌ |
| TEL-7 | Real-user metrics are enabled when the product requires them, in the runtime that supports them | constitutional | `manual` | Web and mobile |
| TEL-8 | The vendor sits behind an app-local module; a component never imports the vendor SDK | constitutional | `biome:noRestrictedImports` | Placement / ❌ |

Ids are stable across versions; a gap is a law that moved to the constitution.

**Why TEL-4 is a law and not a nicety.** The catalogue code is the only thing
that ties a client-side report to the backend log that produced it
(`ARC-ERR-2`). Reporting the failure as a fresh message string breaks that
join, and the same incident then reads as two unrelated problems.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-OBS-1` | Every operation carries a correlation identifier. | the SDK client sends it; the report carries it back |
| `ARC-OBS-5` | Dimensions and attributes are low cardinality. | `plan`, `status`, `route` — never an id or free text |
| `ARC-OBS-7` | An alert represents actionable impact. | an expected inline error is not automatically an alert |
| `ARC-OBS-8` | Instrumentation never changes behavior. | a failed report never breaks a render |
| `ARC-ERR-2` | The code is the contract; the message is human. | reports carry `Exception.code`, not the rendered text |
| `ARC-SEC-7` | Secrets and PII stay out of telemetry. | no payload, email, document or token in an event |
| `ARC-LAY-8` | Infra covered by a lib is used directly. | when a Turystack lib covers the provider, no parallel wrapper |

---

## 🛠️ Project-specific (TypeScript · React · React Native)

### Placement

`telemetry/` is optional and is born when a provider is selected. It exposes the
app's API (`initTelemetry`, `reportError`, `trackEvent`) and is the only place
that imports the vendor. If a Turystack lib covers that integration, use it
directly and do not create a parallel wrapper.

### Error reporting

- The root boundary reports render failures.
- Unhandled rejections and global errors are captured by the bootstrap/vendor.
- An API error preserves `code` and `statusCode`.
- An expected error already presented inline does not necessarily become an
  alert; report according to the product's policy.

### Product signals

Name the event after the domain (`invoice_paid`, `subscription_cancelled`) and
record only useful, limited dimensions (`plan`, `paymentMethod`, `status`). IDs,
free text and PII do not become analytics attributes.

### Web and mobile

- Web can measure LCP, INP and CLS when RUM is enabled.
- Mobile uses signals compatible with the runtime/provider; do not copy
  web-vitals.
- Both initialize once at the root and respect consent/data policy.

### ❌ Never do

- `[TEL-2]` Using `console.error` as production reporting.
- `[TEL-8]` Importing Sentry/PostHog/vendor inside a component.
- `[ARC-OBS-5]` Recording an event on every render or click with no product meaning.
- `[ARC-SEC-7]` Sending an API/form payload to "make debugging easier".
- `[TEL-4]` Reporting an API error as a new string and losing its `code`.
- `[TEL-1]` Initializing the provider inside a component or a route.
- `[TEL-3]` Capturing the same unhandled rejection in both the bootstrap and the vendor.
- `[ARC-OBS-8]` Letting a failed report break the render it was observing.
