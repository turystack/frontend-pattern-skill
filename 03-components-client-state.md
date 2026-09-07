# Features, Components & Client State

**Concept.** A business component belongs to a feature, composes primitives and
interprets entities/flows without knowing the router: route state arrives as
props, navigation leaves as a callback. When there is an API, it consumes the
generated SDK. Recurring API-derived rendering becomes a `{Entity}{Concern}`
widget with its own hook and mapping; a primitive wrapper derives props via
`Omit` and never redeclares a parallel contract.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> TanStack · @turystack. The split, `XXX-n` versus `XXX-Ln`, and why an
> `ARC-…` law is cited and never restated: `turystack-frontend-pattern` › *How
> a section is written*.

## In this file

- [🌐 Generic pattern (portable — stack-independent)](#generic-pattern-portable-stack-independent)
  - [Invariants (the law the gates enforce)](#invariants-the-law-the-gates-enforce)
- [🛠️ Project-specific (TypeScript · React · TanStack · @turystack)](#project-specific-typescript-react-tanstack-turystack)
  - [✅ How to do it](#how-to-do-it)
- [Client state ownership](#client-state-ownership)
- [Primitive ownership](#primitive-ownership)
  - [❌ Never do](#never-do)

**Rules defined here:** `COM-1` · `COM-2` · `COM-3` · `COM-4` · `COM-5` · `COM-6` · `COM-7` · `COM-8` · `COM-9` · `COM-10` · `COM-L1` · `COM-L2` — the law itself is the *Invariants* table below; every ❌ item cites the id it violates.

---

## 🌐 Generic pattern (portable — stack-independent)

**COM-1 — components belong to a feature.** Units that carry domain meaning
live in `features/{feature}/components/`: lists, cards, forms, filters and
actions. Each component folder has its own local barrel;
`features/{feature}/index.ts` is the only public API for routes and other
features. Another feature's internals are never imported. Inside the feature
itself, sibling components may be consumed through the internal alias to avoid
circularity through the root barrel. Prop-drilling beyond two levels is a signal
to promote the state to the route or to expose a callback along the chain. **[COM-1]**

**COM-2 — API shapes come from the SDK.** When the feature consumes an API,
every prop that carries an entity or a query uses the SDK type — preferring the
entity's main type (`User`, `Organization`) over a list-response item alias.
Never redeclare the shape by hand; slicing (`Pick`/`Omit`/`Partial`, indexed
access) happens in the component's types file, on top of the imported type. The
payload schema also comes from the SDK; every remote access goes through the
generated hooks (see `02-sdk.md` and `05-forms.md`). **[COM-2]**

**COM-3 — the component is router-free.** A component never knows the router: route state (search, params) arrives as props; navigation leaves as a callback (`onSelect`, `onOpenDetails`). The route is the only layer that wires those props to the router — including for navigating (see `04-routes-app-shell.md`). **[COM-3]**

**COM-4 — prop conventions.** Data props are named after the entity (`user`, `users` — plural for arrays); callbacks are prefixed `on*` (`onSelect`, `onSubmit`, `onCancel`); state booleans are positive and unprefixed (`open`, `loading`, `disabled` — never `isOpen`/`isLoading`); control props and callbacks are optional when the component still renders safely (`open?`, `onOpenChange?`, `onSuccess?`) and are called with `?.` — entity props required to render stay mandatory. An absent optional callback does **not** hide the action nor short-circuit the render — and neither do permission or state: an unavailable action renders inert with its reason (`ARC-ERR-9`, see `07-ui-states-and-feedback.md` and `12-security-permissions.md`). Removal from the surface is only ever an explicit product decision, never the fallout of a missing callback or a failed check. Slots take `ReactNode` (`header?: ReactNode`) or a render prop (`renderRow?: (row: T) => ReactNode`). **[COM-4]**

**COM-5 — a wrapper derives from the primitive.** A business component that wraps a primitive derives its public props from the **primitive's exported type**, omitting only what it manages internally (`Omit`) — it never redeclares a parallel contract (`value`/`onChange`/`placeholder`/`disabled` by hand). This holds especially for entity selects: they inherit the contract of the library's `Select`, encapsulate options/loading/search, and support the `single` and `multiple` modes when the entity is naturally selected both ways. **[COM-5]**

**COM-6 — a `{Entity}{Concern}` widget is mandatory once boilerplate emerges.** Recurring API-derived rendering — enum→display mapping (status → tone + label), API-backed options (a select populated by a listing query), fetch + fallback (avatar with initials, logo with placeholder), domain-governed formatting — **must** become a `{Entity}{Concern}` business widget (`UserStatus`, `OrganizationSelect`, `UserAvatar`) that owns its own hook **and** its own mapping, with an almost empty prop surface. Trigger: extract when **two** of three hold — (a) enum→display mapping; (b) fetch or fallback; (c) it appears in 2+ places (or clearly will). Distinct from the "dumb" component (`UserCard` receiving `user: User`): the encapsulated widget owns *fetching* and *interpretation*, not just layout. **[COM-6]**

**COM-7 — premature extraction is a bug too.** Zero or **one** trigger condition → inline is the right call. These stay inline: (1) trivial render — no mapping, fetch or fallback; (2) composition coupled to the parent's state; (3) page-specific composition that will not repeat; (4) UI-only visual rule that is not a domain concept; (5) one-off list with the entity already in scope. Purely presentational formatting (currency, document, copyable value) uses a **neutral** primitive from the library or from `src/ui/`, never a per-entity wrapper. **[COM-7]**

**COM-8 — destructive confirmation is composed, and shows what it reaches.** Confirming a destructive action composes the library's shared confirmation primitive — never a modal reassembled by hand; a typed-confirmation rule only if the product explicitly requires it. When the action reaches other entities, the confirmation is the composed modal showing the **blast radius** read from the backend before the confirm is offered (`ARC-CON-11`, contract in `07-ui-states-and-feedback.md`) — the component never counts the impact itself. The failure of the confirmed action follows the feedback law (see `07-ui-states-and-feedback.md` and `11-error-handling.md`). **[COM-8]**

**COM-9 — states are first-class.** Every component that renders asynchronous data handles loading, empty and error explicitly — happy-path-only does not ship. The full matrix of the five realities (loading/empty/error/partial/success) and mutation feedback are law in `07-ui-states-and-feedback.md`. **[COM-9]**

**COM-10 — a field with a uniqueness rule becomes its own input component.** A
field the backend constrains to be unique gets a `{Entity}{Field}Input`
(`ProductNameInput`, `OrganizationSlugInput`, `UserEmailInput`) that owns the
availability read, its states and the reason it renders. This extraction is
**mandatory at the first occurrence**: it overrides the 2-of-3 trigger of
`COM-6` and the "one-off stays inline" of `COM-7`, because what is being
contained is not boilerplate but a **rule** — a domain constraint with two
implementations is two rules, and the copy is the one that diverges the day the
rule gains a condition.

The verdict comes from the contract's availability read (`ARC-CTR-1`; endpoint
standard is `turystack-backend-pattern` › `CTL-8`). A listing filtered in the
consumer is never the check: it is the same defect `ARC-CON-11` names for a
blast radius — the consumer re-deriving what the authority owns, silently, in
the direction of under-reporting. A missing endpoint is a contract blocker
(`ARC-CTR-5`), not a license to reconstruct the rule client-side.

The read is **advisory and non-binding**: the constraint at the authority is
what decides (`ARC-CON-4`), so the submit still handles the conflict code
(`FRM-11`) and two people typing the same value at once stays an expected race,
not a defect. Six consequences follow, in order:

1. the value **always** leaves through `onChange`
   (`turystack-frontend-primitives-pattern` › `PROP-7`). What an
   unavailable value loses is its **validity**, never its emission — a
   component that swallows the emission leaves the form holding the last
   available value while the screen shows another, and the submit writes the
   stale one in silence, which is a worse failure than the one the check was
   built to prevent;
2. the verdict leaves on its own channel, carrying the catalogue's `code` and
   `message` (`ARC-ERR-2`, `ARC-ERR-6`) — never a sentence written inside the
   component, which would drift from the one the write throws;
3. the component renders its own reason inline (`ARC-ERR-9`), so it states the
   block on its own and works outside a form;
4. `checking` gates the submit exactly as a pending mutation does (`FRM-7`);
   without it the user beats the debounce and submits before the verdict;
5. a **failed** read is not a taken value. Unknown fails open and lets the
   submit through, because refusing is the authority's job — the read has five
   outcomes and `error` is not `success` carrying a `false` (`ARC-ERR-8`);
6. in edit, the read excludes the record being edited, or every form collides
   with itself.

The counter-case is real and follows `COM-7`: when losing the submit costs the
user nothing — a single-field form, a value retyped in two seconds — the
conflict error on submit already says it, and the probe is premature extraction
wearing a rule's clothes. The check earns its place when the submit carries work
already done. **[COM-10]**

**Stack lints (React/JSX/TS — they invert in another stack):**

**COM-L1 — handlers named `handle*`.** Event/callback props in JSX take only named functions: `function handleSelect() { ... }` in the component body, or a direct reference to an already-named function (e.g. `disclosure.open`). An inline lambda (`onClick={() => ...}`, `onChange={(next) => { ... }}`) is banned. **Render** callbacks (`.map`, a column's `render`, render props) are not event handlers and stay inline when trivial. **[COM-L1]**

**COM-L2 — no redundant nullish fallback.** Do not coerce an optional value out of habit: if the target prop accepts `undefined`, the value passes straight through (`open={open}`, never `open={open ?? false}`). Use `??` only when the fallback intentionally changes behavior **and** the target type requires a concrete value (e.g. `options` requires an array; `null` → `undefined` required by the type). **[COM-L2]**

### Invariants (the law the gates enforce)


| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| COM-1 | A business component lives in `features/{feature}/components/`; an external consumer imports only `features/{feature}/index.ts` | constitutional | `biome:noRestrictedImports` | Scenarios 1–2 / ❌ |
| COM-2 | When there is an API, entity/query shapes come from the SDK; never redeclared; sliced on top of the imported type | constitutional | `gate:sdk-shadow` | Scenarios 1, 3 / ❌ |
| COM-3 | A component never imports `Route` and never navigates; it receives props and emits callbacks | constitutional | `biome:noRestrictedImports` | Scenario 2 / ❌ |
| COM-4 | Data props named after the entity; `on*`; positive unprefixed booleans; optionals called with `?.`; neither a missing callback nor a failed check removes the action | constitutional | `manual` | Scenarios 2, 4 / ❌ |
| COM-5 | A wrapper derives its props from the primitive's exported type with `Omit`; never a parallel contract | constitutional | `gate:no-parallel-contract` | Scenario 3 / ❌ |
| COM-6 | Recurring API-derived rendering (2-of-3 trigger) becomes a `{Entity}{Concern}` widget with its own hook + mapping | constitutional | `manual` | Scenario 3 / ❌ |
| COM-7 | Zero-or-one trigger condition → inline; five counter-cases; neutral formatting stays generic | constitutional | `manual` | Scenario 5 / ❌ |
| COM-8 | Destructive confirmation composes the shared `Confirm`; never a reassembled modal | constitutional | `gate:no-reassembled-confirm` | Scenario 4 / ❌ |
| COM-9 | Explicit loading/empty/error in every component with asynchronous data | constitutional | `test:five-outcomes` | Scenario 1 (law in `07-ui-states-and-feedback.md`) |
| COM-10 | A field with a uniqueness rule becomes a `{Entity}{Field}Input` owning the contract's availability read; advisory, fails open, emits value and verdict on separate channels | constitutional | `manual` | Scenario 6 / ❌ |
| COM-L1 | An event prop in JSX takes a named function (`handle*`/reference); inline lambda banned | stack lint | `grit:no-inline-handler` | Scenarios 2, 4 / ❌ |
| COM-L2 | No `?? false`/`?? ''` when the target accepts `undefined` | stack lint | `grit:no-redundant-fallback` | Scenarios 3–4 / ❌ |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see `00-overview.md`).

**Mechanisms per rule (React 19 · TanStack · `@turystack/react-web`):**

- **COM-1** — the component lives in `src/features/{feature}/components/{component}/`
  with `{component}.tsx`, `{component}.types.ts` and an exports-only `index.ts`.
  Inside `users`, a sibling uses `@/features/users/components/user-card`; a route
  or another feature uses only `@/features/users`, whose public API is in
  `index.ts`. Helpers live in `features/{feature}/support/`, private by
  default; only an intentional contract is re-exported. Missing primitive or prop →
  extend the library or create it in `src/ui/`,
  never work around it with `className`. UI boolean → `useDisclosure`
  (`@turystack/react-hooks`, re-exported by `@turystack/react-web`); dismissible
  container with a form → `useUnsaved` (`@turystack/react-web`). Action gated by
  permission → `<Protected permissionIds={[...]}>`.
- **COM-2** — `import type { User } from '@/~sdk/users'`: always the main type (`User`, `Organization`), never `ListUsers200['data'][number]`. Same for listing params (`ListUsersQueryParams`). Slicing in the component's `.types.ts` (`User['status']`, `Pick<User, ...>`). Zod from `~sdk` with `standardSchemaResolver` when there is a form (see `05-forms.md`). Every read/write through the generated hooks (`useListUsers`, `useCreateUser`) — no `fetch`/`axios`; a symbol missing after `api:generate` is a backend blocker (see `02-sdk.md`).
- **COM-3** — `createFileRoute`, `Route.useSearch`, `Route.useParams` and `useNavigate` from `@tanstack/react-router` exist **only** in `src/routes/` (see `04-routes-app-shell.md`). The route wires `onSelect` to a `navigate({ to: ... })`; the component never knows a router exists.
- **COM-4** — alphabetical prop destructuring; `onDelete?.(user)` in the handler; the button renders even with `onDelete` `undefined`. `<Protected>` does not remove it either — it renders it inert with the reason (see `12-security-permissions.md`); removal takes an explicit visibility prop and a product reason. Slots typed `ReactNode` in the `.types.ts`.
- **COM-5** — the public type is born from `Omit<SelectProps<...>, 'loading' | 'optionGroup' | 'optionLabel' | 'optionValue' | 'options' | 'searchable'>`: the internal plumbing goes out, and `value`, `defaultValue`, `onChange`, `mode`, `placeholder`, `disabled`, `size` are **inherited**, never retyped (a single select clears via `value = null`). The same holds for any wrapper of a `@turystack/react-web` primitive (contracts in the `frontend-primitives-pattern` skill).
- **COM-6** — `{Entity}{Concern}` naming: `UserStatus`, `UserAvatar`, `OrganizationSelect`, `RoleBadge`, `PaymentMethodIcon`. The widget calls its own SDK hook (`useListOrganizations`, `useGetUser`) and concentrates the mapping (`Record<Status, Variant>`); every consumer becomes a one-liner: `<UserStatus status={user.status} />`.
- **COM-7** — the five counter-cases are in Scenario 5. Repeated presentational formatting (currency, document, copyable value) belongs in a neutral primitive from the library/`src/ui/` — do not create `InvoiceAmount` for what a generic currency primitive already solves.
- **COM-8** — `Confirm` from `@turystack/react-web` controlled by `useDisclosure` from `@turystack/react-hooks` for the radius-free case; composed `Modal` + the SDK's impact hook when the write cascades (Scenario 7 of `07-ui-states-and-feedback.md`). The confirmed mutation's error shows `error.message` as it came (see `11-error-handling.md`); pending/toast feedback in `07-ui-states-and-feedback.md`.
- **COM-9** — early returns in the order `isPending` → `error` → empty, with `Skeleton`/`Loader`, `Alert` and the empty message before the happy path. Full matrix, pagination by `meta.mode` and mutation feedback in `07-ui-states-and-feedback.md` and `02-sdk.md`.
- **COM-10** — `src/features/products/components/product-name-input/`. Public type from `Omit<InputProps, 'debounce' | 'loading' | 'rightSection'>` (`COM-5`: only the plumbing it manages goes out — `value`, `defaultValue`, `onChange`, `placeholder`, `disabled`, `size` stay inherited), plus `excludeProductId?` and `onAvailabilityChange?: (availability: {Entity}{Field}Availability) => void`. The probe is `useGetProductAvailability` from `~sdk` with `query: { enabled }` — never `useListProducts` (see `CTL-8` in the backend skill). The debounce is on the **probe**, never on the emission: `useDebounceValue` from `@turystack/react-hooks` feeds the query's `value`, while `onChange` forwards the keystroke straight through (`Input`'s own `debounce` prop would delay the emission too — that is the swallowed-value failure of `COM-10`). Status is `idle | checking | available | taken`, and it is derived from the query, not stored in state (`ARC-CTR-7`); it reaches the parent through an effect keyed on the derived status, with `useEventCallback` stabilising the parent's handler — the verdict arrives from the network, so there is no interaction to hang it on, and an unstable handler identity would re-fire it on every parent render. `loading={status === 'checking'}` uses the `Input`'s own loader. Inside a form, the consumer maps `taken` to `setError(field, { message: reason.message })` and `available` to `clearErrors(field)`, and adds `checking` to the in-flight gate of `FRM-7` (see `05-forms.md`).
- **COM-L1** — extract a `function handle*` in the component body; a named reference coming from a hook (`confirm.open`) is valid. A render `.map` and a `Table` column's `render` are not event props — they stay inline.
- **COM-L2** — with `open?: boolean` on the primitive, `open={open}` passes straight through. `?? []` for a mandatory `options: Option[]` and `?? undefined` to convert `null` → `undefined` are intentional — allowed.

### ✅ How to do it

**Scenario 1 — list component: SDK + composition + explicit states:** `[COM-1, COM-2, COM-9]`

```tsx
// src/features/users/components/user-list/user-list.types.ts — SDK shapes, never redeclared
import type { ListUsersQueryParams, User } from '@/~sdk/users'

export type UserListProps = {
  params: ListUsersQueryParams
  onSelect?: (user: User) => void
}
```

```tsx
// src/features/users/components/user-list/user-list.tsx
import { Alert, Flex, Skeleton } from '@turystack/react-web'

import { useListUsers } from '@/~sdk/users'

import { UserCard } from '@/features/users/components/user-card'

import type { UserListProps } from './user-list.types'

export function UserList({ onSelect, params }: UserListProps) {
  const { data, error, isPending } = useListUsers(params)

  // the three realities come before the happy path (full matrix in 07-ui-states-and-feedback.md)
  if (isPending) {
    return <Skeleton className="h-60" />
  }

  // error.message shown exactly as it came from the API (see 11-error-handling.md)
  if (error) {
    return (
      <Alert variant="destructive">
        <Alert.Description>{error.message}</Alert.Description>
      </Alert>
    )
  }

  if (data.data.length === 0) {
    return (
      <Alert>
        <Alert.Description>No users found.</Alert.Description>
      </Alert>
    )
  }

  return (
    <Flex direction="col" gap="sm">
      {/* .map is a render callback, not an event handler — COM-L1 does not apply */}
      {data.data.map((user) => (
        <UserCard key={user.userId} onSelect={onSelect} user={user} />
      ))}
    </Flex>
  )
}
```

```tsx
// src/features/users/components/user-list/index.ts — exports-only barrel (see 01-project-structure.md)
export * from './user-list'
export * from './user-list.types'
```

```tsx
// src/features/users/index.ts — the feature's only public API
export * from './components/user-actions'
export * from './components/user-detail'
export * from './components/user-form'
export * from './components/user-list'
export * from './components/users-toolbar'
```

**Scenario 2 — router-free card with a named handler:** `[COM-1, COM-3, COM-4, COM-L1]`

```tsx
// src/features/users/components/user-card/user-card.types.ts
import type { User } from '@/~sdk/users'

export type UserCardProps = {
  user: User
  onSelect?: (user: User) => void
}
```

```tsx
// src/features/users/components/user-card/user-card.tsx
import { Button, Card } from '@turystack/react-web'

// another business component, imported through the barrel (COM-1)
import { UserStatus } from '@/features/users/components/user-status'

import type { UserCardProps } from './user-card.types'

export function UserCard({ onSelect, user }: UserCardProps) {
  // handler named handle*; optional callback called with ?. (COM-4)
  function handleSelect() {
    onSelect?.(user)
  }

  return (
    <Card>
      <Card.Header>
        <Card.Title>{user.name}</Card.Title>
        <UserStatus status={user.status} />
      </Card.Header>
      <Card.Footer>
        {/* the button exists even without onSelect — visibility is <Protected>'s job */}
        <Button onClick={handleSelect} variant="ghost">
          Open
        </Button>
      </Card.Footer>
    </Card>
  )
}
```

> The route wires `onSelect` to a `navigate(...)` — the component never imports `Route` nor `useNavigate` (COM-3, see `04-routes-app-shell.md`).

**Scenario 3 — `{Entity}{Concern}` widgets: enum→display, fetch+fallback, derived select:** `[COM-2, COM-5, COM-6, COM-L2]`

```tsx
// src/features/users/components/user-status/user-status.types.ts — slicing ON TOP OF the SDK type (COM-2)
import type { User } from '@/~sdk/users'

export type UserStatusProps = {
  status: User['status']
}
```

```tsx
// src/features/users/components/user-status/user-status.tsx — trigger (a)+(c): mapping in ONE place
import { Badge, type BadgeVariant } from '@turystack/react-web'

import type { User } from '@/~sdk/users'

import type { UserStatusProps } from './user-status.types'

const VARIANT_BY_STATUS = {
  ACTIVE: 'default',
  INVITED: 'outline',
  SUSPENDED: 'destructive',
} as const satisfies Record<User['status'], BadgeVariant>

const LABEL_BY_STATUS: Record<User['status'], string> = {
  ACTIVE: 'Active',
  INVITED: 'Invited',
  SUSPENDED: 'Suspended',
}

export function UserStatus({ status }: UserStatusProps) {
  return (
    <Badge variant={VARIANT_BY_STATUS[status]}>{LABEL_BY_STATUS[status]}</Badge>
  )
}
```

```tsx
// src/features/users/components/user-avatar/user-avatar.tsx — trigger (b)+(c): fetch + fallback in ONE place
import { Avatar } from '@turystack/react-web'

import { useGetUser } from '@/~sdk/users'

import type { UserAvatarProps } from './user-avatar.types'

export function UserAvatar({ size = 'md', userId }: UserAvatarProps) {
  const { data: user } = useGetUser(userId)

  const initials = user?.name
    .split(' ')
    .map((part) => part[0])
    .slice(0, 2)
    .join('')
    .toUpperCase()

  // the fallback (initials) goes as children; src accepts null directly in the Avatar contract
  return (
    <Avatar size={size} src={user?.avatarUrl}>
      {initials}
    </Avatar>
  )
}
```

```tsx
// src/features/organizations/components/organization-select/organization-select.types.ts — derives from the primitive (COM-5)
import type { SelectProps } from '@turystack/react-web'

import type { Organization } from '@/~sdk/organizations'

export type OrganizationSelectMode = 'single' | 'multiple'

// Omit removes ONLY the internal plumbing; value/onChange/mode/placeholder/disabled/size
// are INHERITED from Select — never retyped
export type OrganizationSelectProps<
  Mode extends OrganizationSelectMode = 'single',
> = Omit<
  SelectProps<Organization, string, string, Mode>,
  'loading' | 'optionGroup' | 'optionLabel' | 'optionValue' | 'options' | 'searchable'
>
```

```tsx
// src/features/organizations/components/organization-select/organization-select.tsx
import { Select } from '@turystack/react-web'

import { useListOrganizations } from '@/~sdk/organizations'

import type {
  OrganizationSelectMode,
  OrganizationSelectProps,
} from './organization-select.types'

export function OrganizationSelect<
  Mode extends OrganizationSelectMode = 'single',
>(props: OrganizationSelectProps<Mode>) {
  const { data, isPending } = useListOrganizations({ limit: 100 })

  return (
    <Select
      placeholder="Select an organization"
      {...props}
      loading={isPending}
      optionLabel="name"
      optionValue="organizationId"
      // ?? [] is intentional: options requires a concrete array — allowed by COM-L2
      options={data?.data ?? []}
    />
  )
}
```

> Every consumer becomes a one-liner: `<UserStatus status={user.status} />`, `<UserAvatar size="sm" userId={userId} />`, `<OrganizationSelect mode="multiple" onChange={handleChangeOrganizations} value={organizationIds} />`. Mapping, fetch and fallback live in a single place.

**Scenario 4 — destructive action composes `Confirm`:** `[COM-4, COM-8, COM-L1, COM-L2]`

```tsx
// src/features/users/components/user-actions/user-actions.tsx
import { useDisclosure } from '@turystack/react-hooks'
import { Button, Confirm, Flex } from '@turystack/react-web'

import type { UserActionsProps } from './user-actions.types'

export function UserActions({ onDelete, onEdit, user }: UserActionsProps) {
  const confirm = useDisclosure()

  function handleEdit() {
    onEdit?.(user)
  }

  function handleConfirmDelete() {
    onDelete?.(user)
    confirm.close()
  }

  return (
    <Flex gap="sm">
      <Button onClick={handleEdit} variant="ghost">
        Edit
      </Button>
      {/* confirm.open is a named reference coming from the hook — valid under COM-L1 */}
      <Button onClick={confirm.open} variant="destructive">
        Delete
      </Button>
      {/* shared Confirm, never a hand-reassembled Modal (COM-8);
          open passed straight through, no ?? false (COM-L2) */}
      <Confirm
        description={`Are you sure you want to delete ${user.name}?`}
        onCancel={confirm.close}
        onConfirm={handleConfirmDelete}
        open={confirm.opened}
        title="Delete user"
      />
    </Flex>
  )
}
```

> The delete mutation and its feedback (pending, toast, invalidation) follow `02-sdk.md` and `07-ui-states-and-feedback.md`; the failure shows `error.message` exactly as it came (`11-error-handling.md`).

**Scenario 5 — when NOT to extract (the five counter-cases):** `[COM-7]`

```tsx
// 1) trivial render — no mapping/fetch/fallback: the cell stays inline;
//    <UserEmail user={user} /> would only hide a string render
{ key: 'email', label: 'Email', width: 220, selector: (user) => user.email }

// 2) composition coupled to the parent's state — the encapsulated widget (UserAvatar ✅)
//    lives INSIDE, but checkbox+avatar+name with selection belongs to the table;
//    <UserNameCell /> would only rename the boilerplate
{
  key: 'name',
  label: 'Name',
  width: 280,
  selector: (user) => (
    <Flex align="center" gap="sm">
      <Checkbox checked={isSelected(user.userId)} onChange={handleToggleSelect} />
      <UserAvatar size="sm" userId={user.userId} />
      {user.name}
    </Flex>
  ),
}
```

```tsx
// 3) page-specific composition that does not repeat — the dashboard greeting;
//    <DashboardGreeting /> is over-engineering
<h1>Welcome back, {profile.name}</h1>
<span>{organizations.length} active organizations</span>
```

```tsx
// 4) UI-only visual rule — "recent" is a table rule, not a User domain concept;
//    it lives in the table config, not in a <UserFreshnessIndicator />
highlighted: (user) => daysSince(user.createdAt) < 7,
```

```tsx
// 5) one-off list with the entity already in scope — the audit only shows in the detail;
//    <UserAudit user={user} /> adds noise
<Flex direction="col" gap="xs">
  <span>Created by {user.createdBy.name}</span>
  <span>Updated on {formatDate(user.updatedAt)}</span>
</Flex>
```

**Scenario 6 — uniqueness rule contained in one input component:** `[COM-2, COM-5, COM-10, COM-L1]`

```tsx
// src/features/products/components/product-name-input/product-name-input.types.ts
import type { InputProps } from '@turystack/react-web'

import type { Product } from '@/~sdk/products'

export type ProductNameAvailabilityStatus =
  | 'idle'
  | 'checking'
  | 'available'
  | 'taken'

export type ProductNameAvailability = {
  status: ProductNameAvailabilityStatus
  // code + message come from the catalogue, never written here (ARC-ERR-2)
  reason?: { code: string; message: string }
}

// COM-5: only the plumbing the component manages goes out;
// value/defaultValue/onChange/placeholder/disabled/size stay inherited
export type ProductNameInputProps = Omit<
  InputProps,
  'debounce' | 'loading' | 'rightSection'
> & {
  excludeProductId?: Product['productId']
  onAvailabilityChange?: (availability: ProductNameAvailability) => void
}
```

```tsx
// src/features/products/components/product-name-input/product-name-input.tsx
import { useDebounceValue, useEventCallback } from '@turystack/react-hooks'
import { Input, Typography } from '@turystack/react-web'
import { useEffect } from 'react'

import { useGetProductAvailability } from '@/~sdk/products'

import type {
  ProductNameAvailability,
  ProductNameInputProps,
} from './product-name-input.types'

export function ProductNameInput({
  excludeProductId,
  onAvailabilityChange,
  onChange,
  value,
  ...props
}: ProductNameInputProps) {
  const [probe, setProbe] = useDebounceValue<string | null>(value ?? null, 400)

  const { data, error, isFetching } = useGetProductAvailability(
    { excludeProductId, field: 'name', value: probe ?? '' },
    // COM-10: no probe with nothing typed, and none while the user is still typing
    { query: { enabled: Boolean(probe) } },
  )

  function resolveStatus(): ProductNameAvailabilityStatus {
    if (!probe) {
      return 'idle'
    }
    if (isFetching) {
      return 'checking'
    }
    // COM-10: a FAILED probe is not a taken value — unknown fails open,
    // the backend constraint is what refuses (ARC-CON-4, ARC-ERR-8)
    if (error || !data) {
      return 'idle'
    }
    return data.available ? 'available' : 'taken'
  }

  const status = resolveStatus()
  const reason = status === 'taken' ? data?.reason : undefined

  // the verdict arrives from the network, not from an interaction, so the effect is
  // the honest channel; useEventCallback keeps the parent's handler identity stable,
  // which is what stops the notification from re-firing on every parent render
  const emitAvailability = useEventCallback(onAvailabilityChange)

  useEffect(() => {
    emitAvailability?.({ reason, status })
  }, [emitAvailability, reason, status])

  function handleChange(next: string | null) {
    // COM-10: the value ALWAYS leaves; what an unavailable value loses is
    // validity, never the emission (PROP-7)
    onChange?.(next)
    setProbe(next)
  }

  return (
    <>
      <Input
        {...props}
        loading={status === 'checking'}
        onChange={handleChange}
        value={value}
      />
      {/* the component states the block on its own, so it works outside a form (ARC-ERR-9) */}
      {reason && <Typography variant="caption">{reason.message}</Typography>}
    </>
  )
}
```

```tsx
// src/features/products/components/product-form/product-form.tsx — the form maps the verdict
// to its own error channel and adds `checking` to the in-flight gate (FRM-7)
const [availability, setAvailability] = useState<ProductNameAvailability>({
  status: 'idle',
})

function handleAvailabilityChange(next: ProductNameAvailability) {
  setAvailability(next)

  if (next.status === 'taken' && next.reason) {
    // message as it came from the catalogue — never rewritten (FRM-11)
    form.setError('name', { message: next.reason.message })
    return
  }

  // cleared only on a confirmed `available`: clearing on idle/checking would wipe
  // the schema's own error for the field
  if (next.status === 'available') {
    form.clearErrors('name')
  }
}

const submitting =
  form.formState.isSubmitting ||
  createProduct.isPending ||
  availability.status === 'checking'
```

> The submit still handles the conflict code the write throws (`FRM-11`): the
> probe narrows the window, it never closes it (`ARC-CON-4`).

## Client state ownership

Choose the owner before creating state:

| State | Owner |
|---|---|
| Remote data and request status | generated hook/TanStack Query |
| Params, shareable filters and pagination | route search |
| Form values, dirty and errors | React Hook Form |
| Session and permissions | `auth/` module |
| Open/closed and ephemeral local selection | component or `useDisclosure` |
| Exit guard with unsaved changes | `useUnsaved` |

Use `@turystack/react-hooks` directly. An app-local hook is born only when the
library does not cover the need: a feature-specific one stays in the feature's
folder; a cross-cutting one may live in `hooks/`.

Context exists only for client state shared by a subtree and with a clear owner.
Do not use context/store to mirror a query response.

## Primitive ownership

- Generic component reusable across products → Turystack kit.
- Primitive specific to this product → `ui/`, when it emerges in the project.
- Component that interprets an entity/flow →
  `features/{feature}/components/`.
- Feature-private helper/mapping → `features/{feature}/support/`.
- Primitive props and implementation follow `frontend-primitives-pattern`.

### ❌ Never do

```tsx
// ❌ [COM-3] component imports Route / reads the router directly
import { Route } from '@/routes/users'

export function UserList() {
  const search = Route.useSearch()
}

// ❌ [COM-3] component navigates — navigation is a callback the route wires to the router
import { useNavigate } from '@tanstack/react-router'

export function UserCard({ user }: UserCardProps) {
  const navigate = useNavigate()
}
```

```tsx
// ❌ [COM-L1] inline lambda in an event prop — extract function handleSelect()
<Button onClick={() => onSelect?.(user)}>Open</Button>

// ❌ [COM-L2] redundant fallback — open?: boolean accepts undefined
<Confirm open={open ?? false} />
```

```tsx
// ❌ [COM-5] parallel contract redeclaring the primitive's props — derive from SelectProps + Omit
export type OrganizationSelectProps = {
  value?: string
  onChange?: (value: string) => void
  placeholder?: string
  disabled?: boolean
}

// ❌ [COM-2] hand-redeclared shape / list-item alias instead of the main type
export type UserCardProps = {
  user: { userId: string; name: string; status: string }
}
type UserItem = ListUsers200['data'][number]
```

```tsx
// ❌ [COM-6] inline enum→display mapping repeated in 2+ places — trigger tripped: becomes <UserStatus />
<Badge variant={user.status === 'ACTIVE' ? 'default' : 'destructive'}>
  {user.status === 'ACTIVE' ? 'Active' : 'Suspended'}
</Badge>

// ❌ [COM-7] premature extraction — zero trigger conditions, the wrapper only hides a <span>
export function UserEmail({ user }: UserEmailProps) {
  return <span>{user.email}</span>
}
```

```tsx
// ❌ [COM-4] optional callback removing the action — it renders, and it fires with ?.
{onEdit && <Button onClick={handleEdit}>Edit</Button>}

// ❌ [COM-4] permission/state removing the action — the law is inert + reason (ARC-ERR-9)
{canDelete && <Button onClick={handleDelete}>Delete</Button>}

// ❌ [COM-8] modal reassembled by hand for a destructive confirmation — compose <Confirm />
<Modal open={confirm.opened}>
  <span>Are you sure?</span>
  <Button onClick={handleDelete}>Delete</Button>
</Modal>

// ❌ [COM-8] cascading deletion confirmed on the entity's name alone — the radius
// has to be read from the backend and shown BEFORE the confirm (ARC-CON-11)
<Confirm description="This action cannot be undone." onConfirm={handleDelete} title="Delete team?" />

// ❌ [COM-1] route/another feature importing users internals
import { UserCard } from '@/features/users/components/user-card/user-card'

// correct outside users:
import { UserCard } from '@/features/users'

// ❌ copying server state into client state
const [orders, setOrders] = useState<Order[]>([])
useEffect(() => setOrders(query.data ?? []), [query.data])

// ❌ recreating a hook already available in the library
export function useToggle() {}
```

```tsx
// ❌ [COM-10] uniqueness rebuilt from a listing — a second implementation of the
// constraint, which diverges the first time the rule gains a condition
const { data } = useListProducts({ name })
const taken = (data?.data.length ?? 0) > 0

// ❌ [COM-10] the emission swallowed when the value is taken — the form keeps the last
// available value while the screen shows another, and the submit writes the stale one
function handleChange(next: string | null) {
  if (taken) {
    return
  }
  onChange?.(next)
}

// ❌ [COM-10] a failed probe read as "taken" — a dead network blocks a submit
// the backend would have accepted
const taken = Boolean(error) || data?.available === false

// ❌ [COM-10] reason written in the component — it drifts from what the write throws
setError('name', { message: 'This name is already in use.' })

// ❌ [COM-10] no `checking` in the in-flight gate — the user beats the debounce
const submitting = form.formState.isSubmitting || createProduct.isPending

// ❌ [COM-10] edit form probing without excluding itself — it collides with its own record
useGetProductAvailability({ field: 'name', value: probe })

// ❌ [COM-10] the rule inlined in the form because "it only happens here" — COM-7 does not
// apply to a domain constraint; the extraction is mandatory at the first occurrence
```
