# Telemetry

**Concept.** Client telemetry covers failures, real-world experience and product
events without duplicating handlers or leaking data.

## Invariants

| ID | Law |
|---|---|
| TEL-1 | The provider is initialized once at bootstrap. |
| TEL-2 | The root error boundary reports before rendering the fallback. |
| TEL-3 | Global/unhandled errors are captured once; do not duplicate if the SDK already does it. |
| TEL-4 | An API error carries `Exception.code` and `statusCode` for correlation. |
| TEL-7 | RUM/web-vitals is enabled when the product requires it and in the applicable runtime. |
| TEL-8 | The vendor sits behind an explicit app-local module; components do not import its SDK directly. |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law |
|---|---|
| `ARC-OBS-1` | Correlation per operation, born at the edge. |
| `ARC-10` | PII out of telemetry. |
| `ARC-11` | Low cardinality in dimensions and attributes. |
| `ARC-OBS-7` | An alert represents actionable impact. |


## Placement

`telemetry/` is optional and is born when a provider is selected. It exposes the
app's API (`initTelemetry`, `reportError`, `trackEvent`) and is the only place
that imports the vendor. If a Turystack lib covers that integration, use it
directly and do not create a parallel wrapper.

## Error reporting

- The root boundary reports render failures.
- Unhandled rejections and global errors are captured by the bootstrap/vendor.
- An API error preserves `code` and `statusCode`.
- An expected error already presented inline does not necessarily become an
  alert; report according to the product's policy.

## Product signals

Name the event after the domain (`invoice_paid`, `subscription_cancelled`) and
record only useful, limited dimensions (`plan`, `paymentMethod`, `status`). IDs,
free text and PII do not become analytics attributes.

## Web and mobile

- Web can measure LCP, INP and CLS when RUM is enabled.
- Mobile uses signals compatible with the runtime/provider; do not copy
  web-vitals.
- Both initialize once at the root and respect consent/data policy.

## Never do

- Using `console.error` as production reporting.
- Importing Sentry/PostHog/vendor inside a component.
- Recording an event on every render or click with no product meaning.
- Sending an API/form payload to “make debugging easier”.
- Reporting an API error as a new string and losing its `code`.
