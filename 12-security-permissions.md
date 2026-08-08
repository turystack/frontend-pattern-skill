# Security & Permissions

**Concept.** Everything that reaches the client is public. Visual permission
shapes the UX; real authorization stays in the backend.

## Invariants

| ID | Law |
|---|---|
| PRM-1 | A conditional action/section uses the app-local `Protected` primitive; no inline check. |
| PRM-2 | Without permission, the action disappears; `disabled` is an operational state, not authorization. |
| PRM-3 | A protected page denies/redirects at the route boundary before rendering. |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-SEC-1` | Scope of the authenticated context. | session from the `auth/` module, never a UI field |
| `ARC-SEC-2` | The backend revalidates; hiding UI is never enforcement. | `Protected` is UX; the `403` remains the protection |
| `ARC-SEC-10` | Untrusted data neutralized on output. | React escapes text by default; rich HTML only via a dedicated sanitizer |
| `ARC-SEC-11` | A credential has a single owner. | `auth/` is the owner; a component never reads storage nor token |
| `ARC-SEC-12` | Permission comes from a single backend catalogue. | permission ids from the session/SDK contract, never a string per feature |
| `ARC-10` | Secrets and PII out of bundle, log and telemetry. | `VITE_`/`EXPO_PUBLIC_` is public by construction |


## Public configuration

Prefixes such as `VITE_` and `EXPO_PUBLIC_` mean the value will be public.
Validate the configuration at boot, but never put a secret in it. An operation
that requires a secret belongs to the backend and is consumed through the SDK.

## Auth ownership

`auth/` concentrates reading/writing the session, the source of permissions and
the integration with the HTTP client. Other modules consume public
functions/hooks; they do not touch storage or the token directly.

## Permission surfaces

```tsx
<Protected permissionIds={['order.update']}>
  <Button onClick={handleEdit}>Edit</Button>
</Protected>
```

- Gate each action at its point of use.
- Navigation item, row action, tab and section use the same primitive.
- A whole route/screen is checked at the router boundary.
- Session loading is an explicit state; do not briefly render protected content
  before the permissions are known.

## Untrusted content and data

React/React Native escape text by default. Legitimate rich HTML uses dedicated
sanitization before any API equivalent to `dangerouslySetInnerHTML`.
Telemetry uses code, status, route and low-cardinality dimensions — never form
payload, email, document, phone number or token.

## Never do

```typescript
const secret = import.meta.env.VITE_STRIPE_SECRET_KEY
localStorage.setItem('accessTokenCopy', token)
permissions.includes('order.update')
console.log({ token, user })
```

- Calling a privileged provider directly from the client.
- Treating `<Protected>` as a replacement for the backend's `403`.
- Disabling an action to communicate a missing permission.
- Injecting user HTML without sanitization.
