# Project Structure & Ownership

**Concept.** The tree reflects real owners. A folder is born when it holds code;
there is no generic zone to receive what has not been modeled yet.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as tree and the imports in
> TypeScript · React · TanStack · Expo · @turystack. The split, `XXX-n` versus
> `XXX-Ln`, and why an `ARC-…` law is cited and never restated: `turystack-
> frontend-pattern` › *How a section is written*.

---

**Rules defined here:** `STR-1` · `STR-2` · `STR-3` · `STR-4` · `STR-L1` — the
law is the *Invariants* table below; every ❌ item cites the id it violates.

## 🌐 Generic pattern (portable — stack-independent)

**STR-1 — three infrastructure owners are mandatory in every app.**

Every frontend has exactly one place for the product shell, one for transport
and cache configuration, and one for the generated contract. They exist from day
one because every app eventually needs them, and a second one born later is a
fork of behavior the whole app depends on. **[STR-1]**

**STR-2 — the import distance declares the boundary.**

Same folder → relative import. Another folder of the same feature → the internal
alias. Outside the feature → its public barrel and nothing else. The rule exists
so a feature's internals can be reorganized without a single consumer noticing,
and so the barrel does not become a circular import hub. **[STR-2]**

**STR-3 — every unit publishes a barrel; the feature publishes a surface.**

A component folder exposes its own barrel, and the feature's entry point
re-exports only what routes and other features may consume. A helper that is not
re-exported is private by contract, not by convention. **[STR-3]**

**STR-4 — a scaffolded folder is a reservation, not a feature.**

A canonical root folder may exist empty and tracked before it has code, because
its position is part of the app's shape. What it must never contain is an
invented feature, layout, permission or telemetry provider written only to make
the folder look inhabited. Real code arrives → the placeholder goes. **[STR-4]**

### Invariants (the law the gates enforce)


| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| STR-1 | Shell, transport/cache and generated contract each have exactly one mandatory owner | constitutional | `manual` | Web tree / ❌ |
| STR-2 | Same folder → relative; same feature → internal alias; outside → the feature's barrel only | constitutional | `biome:noRestrictedImports` | Web tree / ❌ |
| STR-3 | A component folder exposes a barrel; the feature's entry re-exports only its public surface | constitutional | `gate:barrel-shape` | Ownership / ❌ |
| STR-4 | A scaffolded folder may be empty; it may never hold an invented feature | constitutional | `manual` | Initial scaffold / ❌ |
| STR-L1 | `@turystack/frontend-config` is the source of truth for lint, format and TypeScript; a project never redefines those rules locally | stack lint | `gate:config-extends` | Files and exports / ❌ |

Ids are stable across versions. A gap in the numbering is a law that moved to
the constitution or to the section that owns it.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-LAY-5` | The barrel exposes the public surface. | `features/{feature}/index.ts` is the only import path from outside |
| `ARC-LAY-6` | Organization by feature; no global folder of technical type. | `features/{capability}/`, never a global `src/components/` |
| `ARC-LAY-7` | A helper lives with its owner. | `features/{feature}/support/`; root `support/` only when ownerless |
| `ARC-TOP-4` | A folder exists when it has real code. | `.gitkeep` marks a reservation, never a fictional implementation |
| `ARC-CTR-4` | A generated artifact is immutable. | `~sdk/` and `routeTree.gen.ts` are never edited |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · Expo · @turystack)

### Where an application lives

Every frontend is an app in `apps/`, and every app answers to exactly one API
audience — `apps/admin` consumes `/api/v1/admin`, and generates its SDK from
that audience's document and no other. The pairing is the rule: an audience
without an application is a surface nobody consumes, and an application without
an audience is one reaching into a contract that was not written for it.

One application is different. `apps/auth` is the sign-in application, it is the
only one with authentication screens, and every other app redirects to it. What
a product app carries instead is one line:

```tsx
<AuthProvider client="admin">
```

`@repo/oauth-clients` owns the redirect, the PKCE exchange and the session
decision. A product app therefore has no `auth/` of its own: no storage access,
no token, no callback handling. That is `ARC-SEC-11` — a credential has a single
owner — applied across the repository rather than inside one app.

### Initial scaffold

A new application materializes `api/`, `features/`, `hooks/`, `layouts/`,
`routes/`, `support/`, `telemetry/`, `ui/` and `~sdk/`. Every folder holds a
`.gitkeep`; `api/` also receives the mandatory SDK infrastructure. The only
product screen implemented is `routes/index.tsx`, with a centered Welcome
Turystack. `__root.tsx`, `main.tsx` and `router.tsx` are runtime infrastructure,
not example screens.

### Web

```text
src/
├── routes/                   # TanStack Router files
│   ├── __root.tsx
│   ├── _app.tsx             # authenticated boundary, when it exists
│   └── _auth.tsx            # public boundary, when it exists
├── features/                # business capabilities
│   └── orders/
│       ├── components/
│       │   ├── order-table/
│       │   └── order-form/
│       ├── support/         # optional: orders-private helpers
│       └── index.ts         # the feature's only public API
├── api/                     # mandatory: app-local SDK integration
│   ├── http-client.ts
│   └── query-client.ts
│                             # no auth/: @repo/oauth-clients owns the session
├── ui/                      # app-local primitives that emerge in the product
├── hooks/                   # truly cross-cutting hooks, optional
├── layouts/                 # mandatory: building the product shells
│   ├── auth-layout/
│   └── default-layout/
├── support/                 # pure cross-cutting utilities, optional
├── telemetry/               # provider integration, only when selected
├── ~sdk/                    # mandatory: generated from OpenAPI, read-only
├── main.tsx
├── router.tsx
└── styles.css
```

`layouts/` composes the layout primitives from `@turystack/react-web` and builds
the product's real shells. The route selects the layout and keeps params, guards
and redirects. `hooks/` is optional: feature hooks stay with the feature; a
truly cross-cutting hook is born in
`hooks/` only if
`@turystack/react-hooks` does not provide it.

`support/` may hold pure, cross-cutting app functions, such as a local formatter
that belongs to no feature and does not yet justify a library. It contains no
HTTP, auth, QueryClient, primitives, business rule or library wrappers; those
owners are already explicit.

Each feature is a public boundary. Its business components live in
`features/{feature}/components/`; helpers, mappers and pure functions that only
make sense inside it live in `features/{feature}/support/`. The feature's
`index.ts` re-exports the components and contracts that routes or other features
may consume. A feature's `support/` is private by default; a type or hook is
re-exported only when it is an intentional part of the feature's public contract.

Inside the feature itself, a component may import another through the internal
alias, for example `@/features/orders/components/order-status`. Outside it, the
only valid import is `@/features/orders`. This avoids circularity through the
root barrel without exposing internals to consumers.

`api/` and `~sdk/` appear together in every frontend.
`api/http-client.ts` implements the transport the generator expects and
`api/query-client.ts` configures the server state cache. The scaffold asks
whether a specific audience exists, to select the correct OpenAPI surface.

### Mobile

```text
src/
├── app/                     # Expo Router routes/screens
│   ├── _layout.tsx
│   ├── (app)/
│   └── (auth)/
├── features/
│   └── orders/
│       ├── components/
│       ├── support/        # conditional
│       └── index.ts
├── api/                    # mandatory
│                            # no auth/: @repo/oauth-clients owns the session
├── ui/                     # app-local primitives, when they emerge
├── hooks/                  # conditional
├── layouts/                # mandatory: shells with react-mobile
├── support/                # conditional
├── telemetry/              # conditional
└── ~sdk/                   # mandatory: generated, read-only
```

`src/app/` contains only route/screen and `_layout.tsx`. Each `_layout.tsx`
selects/composes a shell from `src/layouts/`; features, clients and reusable
rules stay out of `app/`.

### Ownership

| Code | Owner |
|---|---|
| Component that interprets an entity/flow | `features/{feature}/components/{component}/` |
| Feature-private helper, mapper or hook | `features/{feature}/support/{concern}/` |
| The feature's public API | `features/{feature}/index.ts` |
| HTTP client and QueryClient | `api/` |
| Session and the OAuth exchange | `@repo/oauth-clients` — never the app |
| The permissions the signed-in user holds | the profile query, into `ProtectedProvider` |
| Invoice-specific formatter | `features/invoices/support/` |
| Generic formatter already shared | appropriate Turystack library |
| Order-table-specific hook | `features/orders/components/order-table/` |
| Product shell using the library's Layout | `layouts/` mandatory |
| Cross-cutting pure function with no feature owner | `support/{concern}/` optional |
| Product-specific primitive | `ui/{primitive}/` |
| Telemetry provider | `telemetry/` |

### Files and exports

- File/folder in kebab-case; component export in PascalCase.
- Non-trivial props in `{component}.types.ts`.
- A component's and a feature's `index.ts` contains only re-exports.
- A route and another feature import `@/features/{feature}`, never
  `@/features/{feature}/components/...` or `/support/...`.
- Generated code gets no manual barrel and no edits.
- Use `@/*` → `src/*` consistently in TypeScript, Vite and tests.

### Which library owns which concern

The skill decides **where code belongs**; the library decides **which component,
hook or prop exists**. Before writing something by hand, check whether it already
has an owner (`ARC-LAY-8`):

| Concern | Owner |
|---|---|
| Web primitives, layout primitives, theme provider | `@turystack/react-web` |
| Mobile primitives and shells | `@turystack/react-mobile` |
| Cross-cutting hooks (`useDisclosure`, `useUnsaved`, `useIsMobile`…) | `@turystack/react-hooks` |
| The five outcomes of a remote read (`useDataOutcome`) | `@turystack/react-hooks` |
| Painting those five outcomes (`Table`/`List`/`Select` `outcome`, `Loaded`) | `@turystack/react-web` |
| Icon set | `@turystack/react-icons` |
| Field schemas for forms (shared with the API) | `@turystack/fields` |
| Lint, format, TypeScript | `@turystack/frontend-config` |
| Types, schemas and query/mutation hooks for the API | the generated `~sdk/` (never hand-written) |
| How a primitive is written or extended | the `turystack-frontend-primitives-pattern` skill |

A capability missing from a primitive is solved by extending the library — a
parallel HTML implementation or a `className` escape hatch is the violation, not
the workaround (see `07-consumption.md` in the primitives skill).

### ❌ Never do

- `[ARC-LAY-7]` Using `support/`, `utils/` or `helpers/` to avoid choosing an owner.
- `[ARC-LAY-6]` Creating a global `src/components/` or putting a business component in `ui/`.
- `[STR-2]` Importing another feature's internals, or accidentally exposing a `support/` implementation.
- `[STR-3]` Re-exporting a feature's private helper from its entry point just to reach it from outside.
- `[ARC-LAY-8]` Duplicating `useDisclosure`, `useUnsaved`, `useIsMobile` or any hook the library already provides.
- `[STR-4]` Implementing fictional features, layouts, auth, helpers, primitives or telemetry just to fill the scaffold's canonical folders.
- `[STR-1]` Putting params, redirect or permission guard inside the layout component, or configuring a second transport/cache outside `api/`.
- `[STR-2]` Importing `@/routes` inside a component.
- `[ARC-LAY-8]` Copying a utility across web/admin/mobile instead of extracting it into a library.
- `[ARC-CTR-4]` Editing `~sdk/` or `routeTree.gen.ts` by hand.
- `[STR-L1]` Redefining lint/format/TypeScript rules locally instead of extending `@turystack/frontend-config`.
