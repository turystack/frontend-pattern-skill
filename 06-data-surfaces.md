# Tables & Detail Surfaces

**Concept.** A data table and its filter toolbar are the URL-driven list surface: the columns declare width by the expected content (never the browser's auto-sizing), and the toolbar is a pure function of the route's search object — it receives the whole search, it emits the next whole search. Row actions and detail sheets follow predictable contracts, not per-page improvisation.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `TBL-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `TBL-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**TBL-1 — every column declares a width.** The table layout is never left to the browser's auto-sizing: every column declares a width sized by the expected content. Long or variable text (UUID, e-mail, name, description) uses a fixed width **+ truncation**; an actions column uses an explicit compact width. **[TBL-1]**

**TBL-2 — the toolbar is a function of the route's search.** The page toolbar receives `value` = **exactly** the route's search object and exposes **a single** change callback (`onSearchChange`). Every filter change emits the **next complete search object** — never a partial one, never per-field callbacks (`onStatusChange`, `onTypeChange`). It is this contract that allows cross-cutting decisions (e.g. resetting the page when a filter changes) to live in a single place. **[TBL-2]**

**TBL-3 — the URL shape is stable; UI conversion is internal.** A multi-select filter serialized in the URL (e.g. CSV `'active,blocked'`) converts to the control's shape (array) **inside** the toolbar/select and emits back **in the route schema's shape**. A UI-only shape never leaks into the search contract. **[TBL-3]**

**TBL-4 — a filter placeholder declares intent.** A filter placeholder is action-oriented: `Filter by organization`, `Filter by status` — never option copy like `All organizations`. The empty/all behavior belongs to the filter's clear, not to the placeholder. Copy law detail in 09-content-i18n.md. **[TBL-4]**

**TBL-5 — a predictable row actions menu.** The row actions menu carries the label `Options`; normal actions (`View details`, `Edit`) come before the destructive ones; destructive ones are separated by a divider. A missing optional callback does **not** hide the item — it is called with `?.`; hiding is an explicit permission/visibility decision (see 12-security-permissions.md). **[TBL-5]**

**TBL-6 — the detail sheet's data source is deliberate.** The detail sheet uses the **selected entity directly** when the row already has the fields it renders; it uses a **fetch by id** when the detail endpoint has extra fields, the row is intentionally slim, freshness matters, permissions differ, or the sheet needs independent loading/error. The choice shows up in the props contract — it is never an accident. **[TBL-6]**

**TBL-7 — sheet size follows density.** A compact single-column form/detail uses the compact size (`md`); the wide size (`lg`) is reserved for dense content: multiple sections, two columns, a nested table. A larger size demands a demonstrated content need. **[TBL-7]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| TBL-1 | Every column entry declares `width` by content; variable text = fixed width + truncation; actions = explicit compact | constitutional | Scenario 1 / ❌ |
| TBL-2 | Toolbar receives `value` = the route's search, exposes only `onSearchChange`, always emits the complete object | constitutional | Scenario 2 / ❌ |
| TBL-3 | URL CSV ↔ array converted inside the toolbar/select; the route schema's shape never changes | constitutional | Scenario 2 / ❌ |
| TBL-4 | Filter placeholder declares intent (`Filter by…`), never an option (`All…`) | constitutional | Scenario 2 / ❌ |
| TBL-5 | `Options` menu: normal ones first, destructive ones separated by a divider; an optional callback does not hide the item | constitutional | Scenario 3 / ❌ |
| TBL-6 | Detail sheet picks its source (selected entity vs fetch by id) by a declared criterion | constitutional | Scenario 4 / ❌ |
| TBL-7 | Compact sheet (`md`) by default; wide (`lg`) only with proven density | constitutional | Scenario 4 / ❌ |

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
- **TBL-5** — composed `DropdownMenu`: `DropdownMenu.Trigger asChild` with an `Options` `Button`, normal items, `DropdownMenu.Separator`, then `DropdownMenu.Item variant="destructive"`. Optional callbacks fire with `?.`; hiding only via `<Protected permissionIds={[...]}>` (see 12-security-permissions.md).
- **TBL-6** — entity directly: prop `user: User` (type from the `~sdk`). Fetch by id: prop `invoiceId: string` + the `useGetInvoice` hook **inside** the sheet — that path gets loading/error of its own (states in 07-ui-states-and-feedback.md).
- **TBL-7** — composed `Sheet` from react-web (`Sheet.Header`/`Sheet.Body`/`Sheet.Footer`). Content density decides the width variant; if the required variant does not exist in the lib, the path is to extend it (see the frontend-primitives-pattern skill) — never pin a width with a workaround in the app.

`Table loading` always means a loading overlay over a table that already has valid content. On the first load, the parent component renders a skeleton; filter, sort and pagination preserve the previous result and turn `loading` on. A real-time event updates the items in-place and does not turn that prop on. **[UST-2]**

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

  return <Table columns={columns} itemKey="userId" items={users} loading={refreshing} />
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
        <Protected permissionIds={['users.delete']}>
          {/* hiding is a PERMISSION decision, never a missing-callback one */}
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
| Dense content, multiple sections, history or a shareable URL | Dedicated route |
| Destructive confirmation | `Confirm` |

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

// ❌ [TBL-6] fetch by id when the row already has every field the sheet renders —
// an extra request + loading with no criterion at all (extra fields/freshness/permission/slim)
export function UserDetailsSheet({ userId }: { userId: string }) {
  const { data: user } = useGetUser(userId)
  return <Sheet>{/* renders only name + email, which the row already had */}</Sheet>
}

// ❌ [TBL-7] a wide sheet for a compact 3-field single-column form
<Sheet size="lg">
  <UserForm mode="create" />
</Sheet>
```
