# Project Structure & Ownership

**Concept.** The tree reflects real owners. A folder is born when it holds code;
there is no generic zone to receive what has not been modeled yet.

## Invariants

| ID | Law |
|---|---|
| STR-4 | `layouts/`, `api/` and `~sdk/` are mandatory. In the scaffold, `layouts/` starts merely tracked; once the product shell is defined, it is built with the library. `api/` configures transport/cache and `~sdk/` is generated from OpenAPI. |
| STR-5 | Within the same folder use a relative import; across folders of the same feature use the internal alias. Outside the feature, import only its `index.ts`. |
| STR-6 | A component folder exposes a local barrel; the feature's `index.ts` re-exports only its public surface. |
| STR-7 | The scaffold creates every canonical root folder with `.gitkeep`, without implementing fictional concerns. Once real code appears in a folder, remove its `.gitkeep`. Feature subfolders are born only with the real feature. |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law |
|---|---|
| `ARC-2` | Public surface via barrel. |
| `ARC-3` | Organization by feature. |
| `ARC-4` | A helper lives with its owner. |


## Initial scaffold

A new project materializes `api/`, `auth/`, `features/`, `hooks/`, `layouts/`,
`routes/`, `support/`, `telemetry/`, `ui/` and `~sdk/`. Every folder holds a
`.gitkeep`; `api/` also receives the mandatory SDK infrastructure. The only
product screen implemented is `routes/index.tsx`, with a centered Welcome
Turystack. `__root.tsx`, `main.tsx` and `router.tsx` are runtime infrastructure,
not example screens.

## Web

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
├── auth/                    # session/storage, only in an authenticated app
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

## Mobile

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
├── auth/                   # conditional
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

## Ownership

| Code | Owner |
|---|---|
| Component that interprets an entity/flow | `features/{feature}/components/{component}/` |
| Feature-private helper, mapper or hook | `features/{feature}/support/{concern}/` |
| The feature's public API | `features/{feature}/index.ts` |
| HTTP client and QueryClient | `api/` |
| Session, token and permission source | `auth/` |
| Invoice-specific formatter | `features/invoices/support/` |
| Generic formatter already shared | appropriate Turystack library |
| Order-table-specific hook | `features/orders/components/order-table/` |
| Product shell using the library's Layout | `layouts/` mandatory |
| Cross-cutting pure function with no feature owner | `support/{concern}/` optional |
| Product-specific primitive | `ui/{primitive}/` |
| Telemetry provider | `telemetry/` |

## Files and exports

- File/folder in kebab-case; component export in PascalCase.
- Non-trivial props in `{component}.types.ts`.
- A component's and a feature's `index.ts` contains only re-exports.
- A route and another feature import `@/features/{feature}`, never
  `@/features/{feature}/components/...` or `/support/...`.
- Generated code gets no manual barrel and no edits.
- Use `@/*` → `src/*` consistently in TypeScript, Vite and tests.

## Never do

- Using `support/`, `utils/` or `helpers/` to avoid choosing an owner.
- Creating a global `src/components/` or putting a business component in `ui/`.
- Importing another feature's internals or accidentally exposing a `support/`
  implementation.
- Duplicating `useDisclosure`, `useUnsaved`, `useIsMobile` or any hook the
  library already provides.
- Implementing fictional features, layouts, auth, helpers, primitives or
  telemetry just to fill the scaffold's canonical folders.
- Putting params, redirect or permission guard inside the layout component.
- Importing `@/routes` inside a component.
- Copying a utility across web/admin/mobile instead of extracting it into a
  library.
