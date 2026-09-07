# Security & Permissions

**Concept.** Everything that reaches the client is public. Visual permission
shapes the UX; real authorization stays in the backend. And because the client
was never the protection, the choice of *how* it reacts is free to be the useful
one: it **states** the denial instead of erasing it.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> @turystack. The split, `XXX-n` versus `XXX-Ln`, and why an `ARC-…` law is
> cited and never restated: `turystack-frontend-pattern` › *How a section is
> written*.

---

**Rules defined here:** `PRM-1` · `PRM-2` · `PRM-3` · `PRM-4` — the law is the
*Invariants* table below; every ❌ item cites the id it violates.

## 🌐 Generic pattern (portable — stack-independent)

**PRM-1 — one primitive decides visibility, everywhere.**

A permission check written inline is a check nobody can change centrally: the
day the denial's shape changes — inert instead of hidden, a reason instead of
silence — every inline check has to be found. One primitive means one place.
**[PRM-1]**

**PRM-2 — denial is stated, not erased.**

Without the permission the action stays on screen, inert, carrying its reason;
a denied read surface renders the reason in place of the content. Erasing the
affordance teaches the user the capability does not exist, which is false, and
leaves them with nothing to ask for. **[PRM-2]**

**PRM-3 — a protected page decides at the route boundary.**

No session → redirect to authentication. A session without the permission → the
denied screen at that address. Deciding inside the page means protected content
renders first; redirecting a permitted-but-unauthorized user destroys the
address they were sent. **[PRM-3]**

**PRM-4 — the reason names the permission, never the data.**

The text says which permission is missing, taken from the product catalogue. It
never quotes the record, the amount or the name behind the wall — a reason that
leaks the content defeats the denial it explains. **[PRM-4]**

### Invariants (the law the gates enforce)

| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| PRM-1 | A conditional action/section uses the `Protected` primitive; no inline check | constitutional | `grit:no-inline-permission-check` | Permission surfaces / ❌ |
| PRM-2 | Denied action stays visible and inert with its reason; a denied surface renders the reason card. Erasing is banned | constitutional | `test:denial-visible` | Permission surfaces / ❌ |
| PRM-3 | A protected page decides at the route boundary: no session → login; no permission → denied screen | constitutional | `gate:auth-gate-placement` | Permission surfaces / ❌ |
| PRM-4 | The reason names the missing permission from the catalogue, never the data behind it | constitutional | `manual` | Permission surfaces / ❌ |

Ids are stable across versions; a gap is a law that moved to the constitution.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-SEC-1` | Scope of the authenticated context. | session from `@acme/oauth-clients`, never a UI field |
| `ARC-SEC-2` | The backend revalidates; blocked UI is never enforcement. | `Protected` is UX; the `403` remains the protection |
| `ARC-ERR-9` | Unavailability is stated, never hidden — permission is one cause among plan, state, dependency and limit. | `Protected` makes it inert + reason; a denied surface renders `Unavailable`; non-permission blocks in 07-ui-states-and-feedback.md |
| `ARC-SEC-10` | Untrusted data neutralized on output. | React escapes text by default; rich HTML only via a dedicated sanitizer |
| `ARC-SEC-11` | A credential has a single owner. | `@acme/oauth-clients` is the owner; no app reads storage or a token |
| `ARC-SEC-12` | Permission comes from a single backend catalogue. | permission ids from the session/SDK contract, never a string per feature |
| `ARC-SEC-7` | Secrets and PII out of bundle, log and telemetry. | `VITE_`/`EXPO_PUBLIC_` is public by construction |

---

## 🛠️ Project-specific (TypeScript · React · @turystack)

### Public configuration

Prefixes such as `VITE_` and `EXPO_PUBLIC_` mean the value will be public.
Validate the configuration at boot, but never put a secret in it. An operation
that requires a secret belongs to the backend and is consumed through the SDK.

### Auth ownership

One package owns the session for the whole repository: `@acme/oauth-clients`.
It performs the redirect, the PKCE exchange and the refresh, and it exposes
`AuthProvider` and `useSession`. An application wraps its tree and writes
nothing else — no storage access, no token, no callback route of its own.

The tokens are httpOnly cookies, so there is no token in the page to own. What
the page keeps is one non-sensitive value: when the session expires. Reading it
**synchronously at boot** — not in an effect — is what decides the first render,
and it is why a protected screen never flashes a login form and a signed-in
person never sees a redirect they did not ask for.

Signing in happens in one application, `apps/auth`. A product application that
grows its own login form has forked the flow, and the fork will be the one that
misses the next security fix.

### Permission surfaces

`Protected` has two shapes, one per kind of thing being protected — an **action**
you cannot fire, and a **surface** you cannot read.

```tsx
// action → stays on screen, inert, with the reason within reach
<Protected permissionIds={[Permission.ORDER_UPDATE]}>
  <Button onClick={handleEdit}>Edit order</Button>
</Protected>

// surface → the reason takes the content's place, in the empty state's shape
<Protected
  fallback={
    <Unavailable
      description="Ask an administrator for the Read reports permission."
      title="You do not have access to these reports"
    />
  }
  permissionIds={[Permission.REPORTS_READ]}
>
  <ReportTable reports={reports} />
</Protected>
```

Without `fallback`, `Protected` renders its child **disabled and wrapped in a
`Tooltip`** carrying the catalogue reason — it does not return `null`. The
wrapper is what makes the tooltip reachable: a disabled control fires no pointer
event (`UST-8`). With `fallback`, the child is replaced — that shape is for read
surfaces, and its fallback is `Unavailable` — the app-local primitive in
`src/ui/unavailable/` (see 07-ui-states-and-feedback.md) — never `null`
and never a plain empty state.

- Gate each action at its point of use.
- Navigation item, row action, tab and section use the same primitive: an
  unreachable menu entry renders inert with its reason instead of vanishing —
  which is how the user learns the area exists and what unlocks it.
- A whole route/screen is checked at the router boundary (`PRM-3`): missing
  session redirects to login (that is authentication, not denial); a session
  lacking the permission renders the denied screen at the route, so the address
  keeps working and the reason is on screen.
- Session loading is an explicit state; do not briefly render protected content
  before the permissions are known — and do not flash the inert version either:
  while permissions are unknown, the control renders in its loading state.

### Untrusted content and data

React/React Native escape text by default. Legitimate rich HTML uses dedicated
sanitization before any API equivalent to `dangerouslySetInnerHTML`.
Telemetry uses code, status, route and low-cardinality dimensions — never form
payload, email, document, phone number or token.

### ❌ Never do

```typescript
// ❌ [ARC-SEC-7] a secret in a public-by-construction variable
const secret = import.meta.env.VITE_STRIPE_SECRET_KEY

// ❌ [ARC-SEC-11] a second owner for the credential
localStorage.setItem('accessTokenCopy', token)

// ❌ [PRM-1, ARC-SEC-12] inline check with a permission string written by hand
permissions.includes('order.update')

// ❌ [ARC-SEC-7] credential and PII in the console
console.log({ token, user })
```

- `[ARC-SEC-2]` Calling a privileged provider directly from the client, or treating `<Protected>` as a replacement for the backend's `403`.
- `[PRM-2]` Erasing an action, a menu entry or a whole surface because the permission is missing.
- `[PRM-4]` Leaving a control inert with no reason, or with a generic one (`Not available`).
- `[PRM-3]` Redirecting away from a page the session may not read, instead of showing the denied screen.
- `[PRM-4]` Putting the protected data into the reason text.
- `[ARC-SEC-10]` Injecting user HTML without sanitization.
- `[PRM-3]` Rendering protected content while the session is still loading.
