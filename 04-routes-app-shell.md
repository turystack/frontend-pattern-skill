# Routes & App Shell

**Concept.** The route is the sole owner of the router: it declares `Route`, validates search params and translates navigation into callbacks. The page is thin — it extracts router state, wires callbacks and renders business components; data never travels through a loader, always through the SDK hooks inside the components.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `RTE-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `RTE-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**RTE-1 — the route is the sole owner of the router.** Only the route file declares the route object and reads router state (search params, path params, navigation). Business components receive plain values and callbacks — they do not even know the router exists, and that is why the same toolbar/list mounts under different routes. **[RTE-1]**

**RTE-2 — declarative tree; generated artifact untouchable.** Routes are born from the router's canonical file structure (global root, pathless layouts, dynamic segments); the artifact derived from the tree is tool-generated and never hand-edited. **[RTE-2]**

**ARC-DEL-8 — what lives in the URL is not a matter of taste.** Before deciding where a piece of state lives, answer: *if the user copies the URL and sends it to a colleague, does the colleague need to see the same thing?* If yes, the state belongs to the address — filter, search, sorting, page/cursor, tab, open record, date range. If no, it belongs to the process and must die with it — hover, focus, open menu, unsent draft, scroll position. Keeping address state in `useState` breaks the back button, the shared link and refresh all at once, and none of the three comes back by adding more code to the component. **[ARC-DEL-8]**

**RTE-3 — the search schema comes from the SDK.** Every search param goes through schema validation in the route. When the page mirrors a backend list endpoint (list/filter), the schema **comes from the generated SDK** — the same shape the list hook consumes. An inline schema in the route is reserved for purely UI params (e.g. a `tab`) with no backend counterpart. **[RTE-3]**

**RTE-4 — no loader for data.** The route does not preload business data; components fetch on mount via SDK hooks and render their own states (skeleton until it resolves — see 02-sdk.md and 07-ui-states-and-feedback.md). **[RTE-4]**

**ARC-DEL-1 — thin page: the route is a delivery boundary.** The page component does three things: extracts router state, wires callbacks (`handle*` → navigation), renders business components. Zero inline domain markup — if the page goes past 2–3 structural elements, extract a component (see 03-components-client-state.md). A route orchestrates, it does not present. **[ARC-DEL-1]**

**RTE-6 — the search goes down whole.** When a component consumes the same shape returned by the route's search, pass the complete object (`params={search}`, `value={search}`) — never rebuild an identical object field by field. **[RTE-6]**

**RTE-7 — authentication gate in the pathless layout's `beforeLoad`.** Authentication denies at the route's entrance (redirect to login), in the pathless layout that groups the authenticated pages — never inside a child component. Denial by **permission** follows the same mechanism — the law lives in 12-security-permissions.md. **[RTE-7]**

**RTE-8 — an authenticated page renders the `Page` wrapper.** Every page in the authenticated group mounts the standard chrome: header with breadcrumbs (Dashboard is the only exception), title/description in a vertical group and actions on the right; optional toolbar for filters; content area. Product consistency is not optional per page. **[RTE-8]**

**RTE-L1 — named `Route` export; default export banned.** The tree generator consumes the named export — a default export breaks the convention and exists in no file of the app. **[RTE-L1]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| RTE-1 | Only the route file declares `Route` and reads search/params/navigate; components receive values + callbacks | constitutional | Scenarios 1–4 / ❌ |
| RTE-2 | Route tree declarative by file; `routeTree.gen.ts` generated, never edited | constitutional | Scenarios 2–3 / ❌ |
| RTE-3 | Search validated by schema; a page that mirrors a list endpoint uses the `~sdk` schema; inline only for a UI-local param | constitutional | Scenarios 1, 4 / ❌ |
| RTE-4 | No `loader`/`useLoaderData` for data; components fetch via SDK hooks | constitutional | Scenario 2 / ❌ |
| RTE-6 | Search goes down whole (`params={search}`); never rebuilt field by field | constitutional | Scenario 1 / ❌ |
| RTE-7 | Authentication gate in a pathless layout's `beforeLoad`; never in a child component | constitutional | Scenario 3 / ❌ |
| RTE-8 | An `_app` page renders `Page` (`Page.Header` with breadcrumbs — Dashboard excepted —, `Page.Toolbar` optional, `Page.Content`) | constitutional | Scenarios 1–2, 4 |
| RTE-L1 | Named `Route` export; default export banned | stack lint | ❌ |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law |
|---|---|
| `ARC-DEL-1` | A delivery boundary is thin: it translates, validates shape, delegates. |
| `ARC-DEL-8` | State that survives a reload, a link or history lives in the address. |
| `ARC-CTR-1` | The contract is the single source — the search schema derives from it (`RTE-3`). |
| `ARC-SEC-2` | The backend revalidates; the route gate is experience (`RTE-7`). |

The route is a delivery boundary like a controller or a handler: it receives an
envelope (path, search, fragment), validates shape, extracts values and
delegates. `RTE-1` through `RTE-8` are **how this stack writes that** — which
file declares the route, where the schema comes from, where the gate sits.

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into real code**: the standard is zero comments (see Overview).

**Mechanisms per rule (TanStack Router):**

- **RTE-1** — `export const Route = createFileRoute('...')({...})` exists only in `src/routes/`. `Route.useSearch()`, `Route.useParams()`, `Route.useNavigate()` and `useRouter()` are called only inside the route's `component:`; no component/hook/layout imports `Route` from a route file (see 03-components-client-state.md).
- **RTE-2** — the TanStack Router plugin in Vite generates `src/routeTree.gen.ts` from `src/routes/` — never edit it, never commit it dirty. Files/folders in kebab-case; a dynamic segment is `$param`; a pathless layout carries the `_` prefix (`_auth.tsx`, `_app.tsx`); `__root.tsx` is the global root — it mounts the `Provider` from `@turystack/react-web` (single mount) and the root `Outlet`.
- **Initial scaffold** — `routes/index.tsx` is the only screen and renders a
  centered Welcome Turystack. `__root.tsx` mounts only providers and `Outlet`;
  `layouts/` stays with a `.gitkeep` until a real product shell exists.
- **RTE-3** — `validateSearch: (raw) => listUsersQueryParamsSchema.parse(raw)` with the Zod schema from `@/~sdk/...` — the same shape the list hook consumes, dynamic pagination included (see 02-sdk.md and 02-sdk.md). Inline Zod in the route only for UI-local params (`tab`, open panel).
- **RTE-4** — no `loader:`/`useLoaderData`. The business component calls the TanStack Query hook from `~sdk` (`useListUsers`, `useGetUser`) and handles its own states (see 07-ui-states-and-feedback.md).
- **ARC-DEL-1** — the `component:` extracts (`Route.useSearch/useParams`), wires (`handleSelect` → `navigate`), renders (`<UserList />`). Domain markup lives in `features/{feature}/components/`; the route imports only `@/features/{feature}`.
- **RTE-6** — a toolbar and a list that consume the search shape receive the complete object: `value={search}` / `params={search}` (the toolbar's full contract — see 06-data-surfaces.md).
- **RTE-7** — the `beforeLoad` of `_app.tsx` checks the session via `@/auth` (see 01-project-structure.md) and throws `redirect({ to: '/login' })`. Denial by permission uses `hasPermissions` in the specific route's `beforeLoad` — see 12-security-permissions.md.
- **RTE-8** — `Page` is an app-local primitive in `src/ui/page/` (see 03-components-client-state.md): `Page.Header` with `breadcrumbs`, `Page.Title` (title + description), actions on the right; `Page.Toolbar` for filters; `Page.Content` for the body.
- **RTE-L1** — the tree generator resolves the named `Route` export; a default export does not exist in the app.

**Tree structure:**

```
src/
  routeTree.gen.ts            # generated by the plugin — never edit
  routes/
    __root.tsx                # global root: Provider (single mount), root Outlet
    _auth.tsx                 # pathless layout — mounts the AuthLayout
    _auth/
      login.tsx               # → /login
      forgot-password.tsx     # → /forgot-password
    _app.tsx                  # pathless layout — DefaultLayout + auth gate
    _app/
      index.tsx               # → /  (Dashboard — the only page without breadcrumbs)
      users.tsx               # → /users
      users/
        $userId.tsx           # → /users/$userId
```

### ✅ How to do it

**Scenario 1 — list page with SDK search:** `[RTE-1, RTE-3, ARC-DEL-1, RTE-6, RTE-8]`
```tsx
// src/routes/_app/users.tsx — the route extracts, wires and renders; zero inline domain
import { createFileRoute } from '@tanstack/react-router'

import { listUsersQueryParamsSchema } from '@/~sdk/users'
import type { ListUsersQueryParams, User } from '@/~sdk/users'
import { UserList, UsersToolbar } from '@/features/users'
import { Page } from '@/ui/page'

export const Route = createFileRoute('/_app/users')({
  component: UsersPage,
  validateSearch: (raw) => listUsersQueryParamsSchema.parse(raw),
})

function UsersPage() {
  const search = Route.useSearch()
  const navigate = Route.useNavigate()

  function handleSearchChange(nextSearch: ListUsersQueryParams) {
    navigate({ search: nextSearch })
  }

  function handleSelect(user: User) {
    navigate({ params: { userId: user.userId }, to: '/users/$userId' })
  }

  return (
    <Page>
      <Page.Header breadcrumbs={[{ label: 'Users' }]}>
        <Page.Title description="Manage access and profiles." title="Users" />
      </Page.Header>
      <Page.Toolbar>
        {/* RTE-6: the search goes down whole — the toolbar does not know Route exists */}
        <UsersToolbar onSearchChange={handleSearchChange} value={search} />
      </Page.Toolbar>
      <Page.Content>
        <UserList onSelect={handleSelect} params={search} />
      </Page.Content>
    </Page>
  )
}
```

The components see plain values and callbacks (`onSearchChange`, `onSelect`) — the same `UsersToolbar` mounts under another route with a different navigation target.

**Scenario 2 — detail page with `$param`:** `[RTE-1, RTE-2, RTE-4, ARC-DEL-1, RTE-8]`
```tsx
// src/routes/_app/users/$userId.tsx — the route hands over the param; the component fetches via SDK
import { createFileRoute } from '@tanstack/react-router'

import { UserDetail } from '@/features/users'
import { Page } from '@/ui/page'

export const Route = createFileRoute('/_app/users/$userId')({
  component: UserDetailPage,
})

function UserDetailPage() {
  const { userId } = Route.useParams()

  return (
    <Page>
      <Page.Header breadcrumbs={[{ label: 'Users', to: '/users' }, { label: 'Details' }]}>
        <Page.Title title="User details" />
      </Page.Header>
      <Page.Content>
        {/* RTE-4: no loader — UserDetail calls useGetUser({ userId }) from ~sdk
            and handles its own loading/empty/error (see 07-ui-states-and-feedback.md) */}
        <UserDetail userId={userId} />
      </Page.Content>
    </Page>
  )
}
```

**Scenario 3 — pathless layout with authentication gate:** `[RTE-2, RTE-7]`
```tsx
// src/routes/_app.tsx — pathless: gate in beforeLoad, layout in component
import { createFileRoute, Outlet, redirect } from '@tanstack/react-router'

import { DefaultLayout } from '@/layouts/default-layout'
import { getAuthToken } from '@/auth'

export const Route = createFileRoute('/_app')({
  beforeLoad: () => {
    if (!getAuthToken()) {
      throw redirect({ to: '/login' })
    }
  },
  component: AppShell,
})

function AppShell() {
  return (
    <DefaultLayout>
      <Outlet />
    </DefaultLayout>
  )
}
```

**Scenario 4 — UI-local search param (the only exception to the SDK schema):** `[RTE-3, RTE-8]`
```tsx
// src/routes/_app/settings.tsx — `tab` does not exist in the backend: inline Zod is allowed
import { createFileRoute } from '@tanstack/react-router'
import { z } from 'zod'

import { SettingsTabs } from '@/features/settings'
import { Page } from '@/ui/page'

const settingsSearchSchema = z.object({
  tab: z.enum(['profile', 'notifications']).catch('profile'),
})

export const Route = createFileRoute('/_app/settings')({
  component: SettingsPage,
  validateSearch: (raw) => settingsSearchSchema.parse(raw),
})

function SettingsPage() {
  const { tab } = Route.useSearch()
  const navigate = Route.useNavigate()

  function handleTabChange(nextTab: 'profile' | 'notifications') {
    navigate({ search: { tab: nextTab } })
  }

  return (
    <Page>
      <Page.Header breadcrumbs={[{ label: 'Settings' }]}>
        <Page.Title title="Settings" />
      </Page.Header>
      <Page.Content>
        <SettingsTabs onTabChange={handleTabChange} tab={tab} />
      </Page.Content>
    </Page>
  )
}
```

## App shell and providers

- The global provider is registered once at the root.
- The authenticated/public boundary is expressed by the router tree.
- Guard and redirect belong to the route/layout route, not to the visual shell.
- `layouts/` is mandatory and builds the product shells by composing the Layout
  primitives from `react-web` or `react-mobile`.
- Web selects the shell in pathless/root routes; mobile selects it in
  `_layout.tsx`.
- The shell receives slots/children and composes navigation, header and content;
  it does not fetch feature data nor decide resource permission.
- The root error boundary covers the tree; smaller boundaries exist when a
  subtree has recovery of its own.

```tsx
// src/layouts/default-layout/default-layout.tsx
import { Layout } from '@turystack/react-web'

import type { DefaultLayoutProps } from './default-layout.types'

export function DefaultLayout({ children }: DefaultLayoutProps) {
  return (
    <Layout withSidebar>
      <Layout.Header bordered>{/* product navigation */}</Layout.Header>
      <Layout.Main>
        <Layout.Content>{children}</Layout.Content>
      </Layout.Main>
    </Layout>
  )
}
```

On mobile, the same `DefaultLayout` composes `Layout` from
`@turystack/react-mobile`; `src/app/(app)/_layout.tsx` only selects it and
renders `Slot`/`Stack` inside it.

### ❌ Never do

```tsx
// ❌ [RTE-1] a component reaching for the router — banned outside the route file
// src/features/users/components/users-toolbar/users-toolbar.tsx
import { Route } from '@/routes/_app/users'

export function UsersToolbar() {
  const search = Route.useSearch()
  const navigate = Route.useNavigate()
}
```

```tsx
// ❌ [RTE-4] loader/useLoaderData for business data (data comes from the ~sdk hooks)
export const Route = createFileRoute('/_app/users')({
  loader: () => api.listUsers(),
  component: UsersPage,
})

// ❌ [RTE-3] hand-duplicating the shape of a list endpoint — the schema already exists in ~sdk
validateSearch: (raw) => z.object({ filter: z.string().optional(), page: z.number() }).parse(raw)

// ❌ [RTE-6] rebuilding the search field by field — pass the whole object
<UserList params={{ filter: search.filter, limit: search.limit, page: search.page }} />
```

```tsx
// ❌ [RTE-7] auth gate inside the component instead of the pathless layout's beforeLoad
function UsersPage() {
  if (!getAuthToken()) {
    return <Navigate to="/login" />
  }
}

// ❌ [RTE-2] editing src/routeTree.gen.ts by hand — it is generated by the Vite plugin

// ❌ [RTE-L1] default export — the generator consumes the named `Route` export
export default function UsersPage() {}
```
