# Tables & Detail Surfaces

**Concept.** A data table and its filter toolbar are the URL-driven list surface: the columns declare width by the expected content (never the browser's auto-sizing), and the toolbar is a pure function of the route's search object — it receives the whole search, it emits the next whole search. Row actions and detail sheets follow predictable contracts, not per-page improvisation.

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
- [Detail surfaces](#detail-surfaces)
  - [❌ Never do](#never-do)

**Rules defined here:** `TBL-1` · `TBL-2` · `TBL-3` · `TBL-4` · `TBL-5` · `TBL-6` · `TBL-7` · `TBL-8` — the law itself is the *Invariants* table below; every ❌ item cites the id it violates.

---

## 🌐 Generic pattern (portable — stack-independent)

**TBL-1 — every column declares a width.** The table layout is never left to the browser's auto-sizing: every column declares a width sized by the expected content. Long or variable text (UUID, e-mail, name, description) uses a fixed width **+ truncation**; an actions column uses an explicit compact width. **[TBL-1]**

**TBL-2 — the toolbar is a function of the route's search.** The page toolbar receives `value` = **exactly** the route's search object and exposes **a single** change callback (`onSearchChange`). Every filter change emits the **next complete search object** — never a partial one, never per-field callbacks (`onStatusChange`, `onTypeChange`). It is this contract that allows cross-cutting decisions (e.g. resetting the page when a filter changes) to live in a single place. **[TBL-2]**

**TBL-3 — the URL shape is stable; UI conversion is internal.** A multi-select filter serialized in the URL (e.g. CSV `'active,blocked'`) converts to the control's shape (array) **inside** the toolbar/select and emits back **in the route schema's shape**. A UI-only shape never leaks into the search contract. **[TBL-3]**

**TBL-4 — a filter placeholder declares intent.** A filter placeholder is action-oriented: `Filter by organization`, `Filter by status` — never option copy like `All organizations`. The empty/all behavior belongs to the filter's clear, not to the placeholder. Copy law detail in 09-content-i18n.md. **[TBL-4]**

**TBL-5 — a predictable row actions menu.** The row actions menu carries the label `Options`; normal actions (`View details`, `Edit`) come before the destructive ones; destructive ones are separated by a divider. A missing optional callback does **not** hide the item — it is called with `?.`; an item the user cannot use renders inert with its reason, it does not vanish from the menu (`ARC-ERR-9`, see 12-security-permissions.md). **[TBL-5]**

**TBL-6 — the detail sheet's data source is deliberate.** The detail sheet uses the **selected entity directly** when the row already has the fields it renders; it uses a **fetch by id** when the detail endpoint has extra fields, the row is intentionally slim, freshness matters, permissions differ, or the sheet needs independent loading/error. The choice shows up in the props contract — it is never an accident. A **deep-linkable** surface has no choice: whoever arrives by address arrives with no row in hand (the record may sit on another page, or under another filter), so it always fetches by id — see `RTE-9` in 04-routes-app-shell.md. The entity-directly path survives for surfaces that are not addressable, such as an inline expansion or a preview popover. **[TBL-6]**

**TBL-8 — opening a record is navigation, not local state.** A row that opens a detail sheet/modal emits `onSelect`/`onOpenDetails` and nothing else: the table does not own the surface, does not hold `selectedRow` and does not render the sheet. The route turns that callback into `resourceId` in the address and the feature's deep-link component renders the surface (`ARC-DEL-9`, `RTE-9`). The table stays a table — which is also why the same table mounts under a page whose detail is a full route instead of a sheet. **[TBL-8]**

**TBL-7 — sheet size follows density.** A compact single-column form/detail uses the compact size (`md`); the wide size (`lg`) is reserved for dense content: multiple sections, two columns, a nested table. A larger size demands a demonstrated content need. **[TBL-7]**

### Invariants (the law the gates enforce)


| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| TBL-1 | Every column entry declares `width` by content; variable text = fixed width + truncation; actions = explicit compact | constitutional | `manual` | Scenario 1 / ❌ |
| TBL-2 | Toolbar receives `value` = the route's search, exposes only `onSearchChange`, always emits the complete object | constitutional | `manual` | Scenario 2 / ❌ |
| TBL-3 | URL CSV ↔ array converted inside the toolbar/select; the route schema's shape never changes | constitutional | `manual` | Scenario 2 / ❌ |
| TBL-4 | Filter placeholder declares intent (`Filter by…`), never an option (`All…`) | constitutional | `manual` | Scenario 2 / ❌ |
| TBL-5 | `Options` menu: normal ones first, destructive ones separated by a divider; nothing hides the item — an unusable one renders inert with its reason | constitutional | `manual` | Scenario 3 / ❌ |
| TBL-6 | Detail sheet picks its source (selected entity vs fetch by id) by a declared criterion; a deep-linkable surface always fetches by id | constitutional | `manual` | Scenario 4 / ❌ |
| TBL-7 | Compact sheet (`md`) by default; wide (`lg`) only with proven density | constitutional | `manual` | Scenario 4 / ❌ |
| TBL-8 | The row emits `onSelect`; the table holds no `selectedRow` and renders no sheet — opening goes through the address | constitutional | `grit:no-selected-row-state` | Scenario 4 / ❌ |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into real code**: the standard is zero comments (see 00-overview.md).

**Mechanisms per rule:**

- **TBL-1** — `Table` from `@turystack/react-web`: every item of `TableColumns<T>` has `width?: number` (pixels). The type marks `width` as optional — **the law makes it mandatory**. Truncation via `Typography truncate` inside the fixed width; actions cell via `selector` with a compact `width` (~64) and `align="right"`.
- **TBL-2** — the search is born in the route (`validateSearch` from the `~sdk`, passed whole to the page — see 04-routes-app-shell.md). The toolbar types `value` with the `~sdk` query type (e.g. `ListUsersQuery`) and builds the next object with a spread: `onSearchChange({ ...value, status, page: 1 })`. Resetting `page` when a filter changes happens **in that merge** — it is only possible because the contract is the complete object.
- **TBL-3** — the route schema stores a multi-select as CSV; the toolbar does `value.status?.split(',')` to feed the `Select mode="multiple"` and `next.join(',')` on emit. The conversion never shows up in the route nor in the page component.
- **TBL-4** — the `placeholder` prop of `Select`/`Input`/domain select widgets, with intent copy in en-US. Domain selects (`OrganizationSelect` etc.) are business widgets — contract in 03-components-client-state.md.
- **TBL-5** — composed `DropdownMenu`: `DropdownMenu.Trigger asChild` with an `Options` `Button`, normal items, `DropdownMenu.Separator`, then `DropdownMenu.Item variant="destructive"`. Optional callbacks fire with `?.`; permission goes through `<Protected permissionIds={[...]}>`, which leaves the item inert with its reason instead of removing it (see 12-security-permissions.md).
- **TBL-6** — entity directly: prop `user: User` (type from the `~sdk`). Fetch by id: prop `invoiceId: string` + the `useGetInvoice` hook **inside** the sheet — that path gets loading/error of its own (states in 07-ui-states-and-feedback.md). When the surface is mounted by `{Entity}DeepLink`, the fetch already lives in the deep-link component and the sheet receives `user`, `loading` and `error` as props — one owner of the query, not two.
- **TBL-8** — `UserTable` receives `onSelect` and calls it in the row/`View details` item; `selectedUser`/`sheetOpen` do not exist in the table. The page mounts `<UserDeepLink resourceId={search.resourceId} onClose={...} />` next to the list — full wiring in Scenario 5 of 04-routes-app-shell.md.
- **TBL-7** — composed `Sheet` from react-web (`Sheet.Header`/`Sheet.Body`/`Sheet.Footer`). Content density decides the width variant; if the required variant does not exist in the lib, the path is to extend it (see the frontend-primitives-pattern skill) — never pin a width with a workaround in the app.

A table backed by a query takes `outcome`, not `items` — the two are mutually
exclusive in the type, and `items` is what remains for rows that came in as a
prop. With `outcome`, the header, the columns and the pagination stay in place
through all five states, which is the reason a failed read belongs *inside* the
table rather than in place of it: the screen does not shrink and grow between
two accounts. `loading` follows the same split — with `outcome` the primitive
decides between skeleton rows and an overlay over rows that are still valid, and
a real-time event turns neither on. **[UST-2, UST-10, UST-11]**

### ✅ How to do it

**Scenario 1 — columns with `width` + truncation + compact actions:** `[TBL-1]`
```tsx
// src/features/users/components/user-table/user-table.tsx
import { Table, type TableColumns, Typography } from '@turystack/react-web'

import type { User } from '@/~sdk/users'

import { UserRowActions } from '@/features/users/components/user-row-actions'
import { UserStatus } from '@/features/users/components/user-status'

import type { UserTableProps } from './user-table.types'

export function UserTable({ onDelete, onEdit, refreshing, users }: UserTableProps) {
  const columns: TableColumns<User> = [
    {
      key: 'name',
      label: 'Name',
      width: 220, // variable text: fixed width…
      selector: (user) => <Typography truncate>{user.name}</Typography>, // …+ truncation
    },
    {
      key: 'email',
      label: 'E-mail',
      width: 260,
      selector: (user) => <Typography truncate>{user.email}</Typography>,
    },
    {
      key: 'status',
      label: 'Status',
      width: 120, // predictable content: a tight width
      selector: (user) => <UserStatus status={user.status} />,
    },
    {
      align: 'right',
      key: 'actions',
      width: 64, // actions column: compact and explicit
      selector: (user) => (
        <UserRowActions onDelete={onDelete} onEdit={onEdit} user={user} />
      ),
    },
  ]

  // the query lives one level up and arrives as one value; the five states
  // are the primitive's to paint (UST-10, UST-11)
  return <Table columns={columns} itemKey="userId" outcome={outcome} />
}
```

**Scenario 2 — toolbar with a search contract, internal CSV and an intent placeholder:** `[TBL-2, TBL-3, TBL-4]`
```tsx
// src/features/users/components/users-toolbar/users-toolbar.types.ts
import type { ListUsersQuery } from '@/~sdk/users'

export type UsersToolbarProps = {
  value: ListUsersQuery // EXACTLY the route's search object
  onSearchChange: (next: ListUsersQuery) => void // the ONLY exposed callback
}
```
```tsx
// src/features/users/components/users-toolbar/users-toolbar.tsx
import { Flex, Input, Select } from '@turystack/react-web'

import { OrganizationSelect } from '@/features/organizations'

import type { UsersToolbarProps } from './users-toolbar.types'

const STATUS_OPTIONS = [
  { label: 'Active', value: 'active' },
  { label: 'Blocked', value: 'blocked' },
]

export function UsersToolbar({ onSearchChange, value }: UsersToolbarProps) {
  const statusValue = value.status?.split(',') // URL CSV → array, INSIDE the toolbar

  function handleSearchTermChange(term: string | null) {
    onSearchChange({ ...value, page: 1, search: term ?? undefined }) // always the COMPLETE object
  }

  function handleStatusChange(next: string[]) {
    onSearchChange({
      ...value,
      page: 1, // the page reset lives in the merge — that is why the contract is the complete object
      status: next.length > 0 ? next.join(',') : undefined, // array → CSV: the route's shape
    })
  }

  function handleOrganizationChange(organizationId: string | null) {
    onSearchChange({ ...value, organizationId: organizationId ?? undefined, page: 1 })
  }

  return (
    <Flex gap="md">
      <Input
        debounce
        onChange={handleSearchTermChange}
        placeholder="Search by name or e-mail"
        value={value.search}
      />
      <Select
        mode="multiple"
        onChange={handleStatusChange}
        optionLabel="label"
        optionValue="value"
        options={STATUS_OPTIONS}
        placeholder="Filter by status" // intent, never "All statuses"
        value={statusValue}
      />
      <OrganizationSelect
        mode="single"
        onChange={handleOrganizationChange}
        placeholder="Filter by organization"
        value={value.organizationId}
      />
    </Flex>
  )
}
```

**Scenario 3 — row actions menu (`Options`):** `[TBL-5]`
```tsx
// src/features/users/components/user-row-actions/user-row-actions.tsx
import { Button, DropdownMenu } from '@turystack/react-web'

import { Permission } from '@/~sdk/permissions'

import { Protected } from '@/ui/protected'

import type { UserRowActionsProps } from './user-row-actions.types'

export function UserRowActions({ onDelete, onDetails, onEdit, user }: UserRowActionsProps) {
  function handleDetails() {
    onDetails?.(user) // an optional callback does NOT hide the item — it fires with ?.
  }

  function handleEdit() {
    onEdit?.(user)
  }

  function handleDelete() {
    onDelete?.(user)
  }

  return (
    <DropdownMenu>
      <DropdownMenu.Trigger asChild>
        <Button size="sm" variant="ghost">
          Options
        </Button>
      </DropdownMenu.Trigger>
      <DropdownMenu.Content align="end">
        <DropdownMenu.Item onClick={handleDetails}>View details</DropdownMenu.Item>
        <DropdownMenu.Item onClick={handleEdit}>Edit</DropdownMenu.Item>
        <DropdownMenu.Separator /> {/* destructive always separated by a divider */}
        <Protected permissionIds={[Permission.USERS_DELETE]}>
          {/* without the permission the item stays here, inert, carrying its
              reason — it is never removed from the menu (ARC-ERR-9) */}
          <DropdownMenu.Item onClick={handleDelete} variant="destructive">
            Delete
          </DropdownMenu.Item>
        </Protected>
      </DropdownMenu.Content>
    </DropdownMenu>
  )
}
```

**Scenario 4 — detail sheet: deliberate source in the props contract + density:** `[TBL-6, TBL-7]`
```tsx
// src/features/users/components/user-details-sheet/user-details-sheet.types.ts
// the table row ALREADY HAS the fields the sheet shows → entity directly, no fetch
import type { User } from '@/~sdk/users'

export type UserDetailsSheetProps = {
  open?: boolean
  user: User
  onChange?: (open: boolean) => void
}

// src/features/invoices/components/invoice-details-sheet/invoice-details-sheet.types.ts
// the detail endpoint has extra fields / the row is slim → fetch by id inside the sheet
export type InvoiceDetailsSheetProps = {
  invoiceId: string
  open?: boolean
  onChange?: (open: boolean) => void
}
```
```tsx
// src/features/invoices/components/invoice-details-sheet/invoice-details-sheet.tsx
import { Sheet } from '@turystack/react-web'

import { useGetInvoice } from '@/~sdk/invoices'

import type { InvoiceDetailsSheetProps } from './invoice-details-sheet.types'

export function InvoiceDetailsSheet({ invoiceId, onChange, open }: InvoiceDetailsSheetProps) {
  const { data: invoice, error, isPending } = useGetInvoice(invoiceId)

  return (
    // compact single-column detail → md density; lg would demand dense sections/
    // two columns/a nested table (width variant: frontend-primitives-pattern skill)
    <Sheet onChange={onChange} open={open}>
      <Sheet.Header closable>
        <Sheet.Title>Invoice details</Sheet.Title>
      </Sheet.Header>
      <Sheet.Body>
        {/* fetch by id ⇒ the sheet gets loading/empty/error of its own —
            the five states in 07-ui-states-and-feedback.md */}
      </Sheet.Body>
    </Sheet>
  )
}
```

## Detail surfaces

Pick the surface by size and by the context it preserves:

| Case | Surface |
|---|---|
| Quick lookup while the list stays relevant | Sheet/drawer |
| Short edit of a few fields | Modal or compact sheet |
| Dense content, multiple sections or history | Dedicated route |
| Destructive confirmation | `Confirm` (with blast radius — see 07-ui-states-and-feedback.md) |

Sharing is **not** in that table on purpose: every surface above is addressable
(`ARC-DEL-9`), so "it needs a shareable URL" stopped being an argument for
choosing a route. What decides is density and how much of the list's context the
user needs to keep.

A backoffice tends to preserve table/filters with a sheet; a B2B admin alternates
between sheet and route depending on density; an end-user product prefers a route
when the detail is a main experience. This is a heuristic — volume, task and
navigation take precedence over the product's label.

If the row already contains what is needed, pass the entity. Fetch by id only
when the detail demands extra fields, freshness, permission or a slim contract.

### ❌ Never do

```tsx
// ❌ [TBL-1] a column with no width — layout left to the browser's auto-sizing
{ key: 'email', label: 'E-mail', selector: (user) => user.email }

// ❌ [TBL-1] variable text with a fixed width but no truncation — a UUID overflows the cell
{ key: 'userId', label: 'ID', width: 120, selector: (user) => user.userId }

// ❌ [TBL-2] per-field callbacks — the contract is ONE onSearchChange with the complete object
export type UsersToolbarProps = {
  value: ListUsersQuery
  onStatusChange: (status: string) => void
  onOrganizationChange: (organizationId: string) => void
}

// ❌ [TBL-2] emits a partial object — wipes every other filter from the route's search
onSearchChange({ status: next.join(',') })

// ❌ [TBL-3] a UI shape leaking into the route contract — the schema stores CSV, not an array
onSearchChange({ ...value, status: next }) // next: string[] straight into the URL

// ❌ [TBL-4] an option placeholder instead of intent (copy law: 09-content-i18n.md)
<OrganizationSelect placeholder="All organizations" />

// ❌ [TBL-5] destructive with no divider, before the normal ones, and an item hidden by a missing callback
<DropdownMenu.Content>
  <DropdownMenu.Item onClick={handleDelete}>Delete</DropdownMenu.Item>
  {onDetails && (
    <DropdownMenu.Item onClick={handleDetails}>View details</DropdownMenu.Item>
  )}
</DropdownMenu.Content>

// ❌ [TBL-6] fetch by id when the row already has every field the sheet renders,
// AND the surface is not addressable — an extra request + loading with no criterion
// (extra fields/freshness/permission/slim; a deep-linked surface always fetches — RTE-9)
export function UserDetailsSheet({ userId }: { userId: string }) {
  const { data: user } = useGetUser(userId)
  return <Sheet>{/* renders only name + email, which the row already had */}</Sheet>
}

// ❌ [TBL-7] a wide sheet for a compact 3-field single-column form
<Sheet size="lg">
  <UserForm mode="create" />
</Sheet>

// ❌ [TBL-8] the table owning the surface — selection in memory, sheet rendered inside,
// and the address none the wiser (ARC-DEL-9)
export function UserTable({ users }: UserTableProps) {
  const [selectedUser, setSelectedUser] = useState<User | null>(null)

  return (
    <>
      <Table columns={columns} items={users} />
      <UserDetailsSheet open={Boolean(selectedUser)} user={selectedUser} />
    </>
  )
}
```
