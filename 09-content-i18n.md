# Content & i18n

**Concept.** Copy is part of the contract, not decoration. The product picks ONE language (examples here are en-US) and every UI string follows it; a placeholder steers the action, a button starts with a verb, and empty/error text says what the user does now — never only what happened.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `TXT-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `TXT-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**TXT-1 — one language per product.** The product picks a single interface language and **every** UI string follows it — title, button, placeholder, empty, error, toast. A string stranded in another language is a violation, not a detail: the interface never mixes languages. Code identifiers (variables, props, `error.code`) follow the code's own convention and are never shown to the user. **[TXT-1]**

**TXT-2 — a placeholder declares intent, never an option.** A filter/search/select placeholder steers the action: `Filter by organization`, `Search by name or email`. Never text that looks like a selected option (`All organizations`) nor an empty label (`Select`) — the "all/none" state belongs to the control's clear, not to the placeholder. **[TXT-2]**

**TXT-3 — empty and error steer the next action.** Empty/error copy comes in a pair: what happened + what to do now (CTA). `No data` on its own is a dead end; `No users yet` + a `Create user` button is a path. **[TXT-3]**

**TXT-4 — buttons are verb-first; sentence case.** Every button starts with the verb of the action (`Create user`, `Save changes`, `Try again`). Titles, labels and buttons use sentence case (`Edit user profile`), never Title Case. **[TXT-4]**

**TXT-5 — an API error is never rewritten locally.** The API error message arrives translated and ready from the backend; the frontend shows it exactly as it came and **never** writes/translates/paraphrases an API error on its own. The full consumption law (message as-is, branch only by code) belongs to 11-error-handling.md. **[TXT-5]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| TXT-1 | One language per product; every UI string follows it — zero strings stranded in another language | constitutional | Scenarios 1–4 / ❌ |
| TXT-2 | A placeholder declares intent (`Filter by…`), never an option (`All…`) nor an empty label (`Select`) | constitutional | Scenario 1 / ❌ |
| TXT-3 | Empty/error in a pair: what happened + the CTA for what to do now | constitutional | Scenario 2 / ❌ |
| TXT-4 | Buttons verb-first; titles/labels/buttons in sentence case | constitutional | Scenarios 2–3 / ❌ |
| TXT-5 | API error shown as it came (`error.message`); never written/rewritten in the frontend | constitutional | Scenario 2 / ❌ (consumption law in 11-error-handling.md) |
| TXT-L1 | Primitives' internal strings come from `translations` in the root `Provider` — once, never per usage | stack lint | Scenario 4 / ❌ |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **TXT-1** — there is no dynamic i18n layer in the app: the UI string is a literal in the JSX, in the product's language. Review sweeps the JSX for visible text in another language; identifiers (`error.code`, prop names, routes) follow code convention, never visible to the user.
- **TXT-2** — toolbar placeholders follow `Filter by <dimension>` / `Search by <fields>`; the "all options" state is the control's cleared value (single-select at `null`), never the text. The toolbar's structural contract (whole search object, `onChange`) belongs to 06-data-surfaces.md — only the copy law lives here.
- **TXT-3** — the surface's five realities (loading/empty/error/partial/success) are the law of 07-ui-states-and-feedback.md; the copy of each one follows TXT-3: state the fact + steer. A list empty state always considers a creation CTA; an error always offers the way out (`Try again`).
- **TXT-4** — verb-first applies to buttons, action menu items and empty-state CTAs. Sentence case applies to page/modal/sheet titles, field labels and table columns.
- **TXT-5** — the SDK types every error as `Exception` (`{ statusCode, code, message, ...metadata }`); `message` arrives ready from the backend. Show `error.message` as it came; `error.code` decides **where** to show it (e.g. `setError('email', { message: error.message })`), never what the text is. Full law in 11-error-handling.md.
- **TXT-L1** — react-web has internal strings (`Select`'s empty, `Table`'s `noData`, `Confirm`'s buttons, `PasswordInput`'s strength…). They are configured **once** via `translations` in the `Provider` mounted in `__root.tsx` — never a second local `Provider`, never an override per usage.

### ✅ How to do it

**Scenario 1 — intent-driven toolbar placeholders:** `[TXT-1, TXT-2]`
```tsx
import { Flex, Input } from '@turystack/react-web'
import type { ListUsersQueryParams } from '@/~sdk/users'
import { OrganizationSelect } from '@/features/organizations'

type UsersToolbarProps = {
  value: ListUsersQueryParams
  onChange: (value: ListUsersQueryParams) => void
}

export function UsersToolbar({ value, onChange }: UsersToolbarProps) {
  function handleSearchChange(search: string | null) {
    onChange({ ...value, page: 1, search: search ?? undefined })
  }

  function handleOrganizationChange(organizationId: string | null) {
    onChange({ ...value, organizationId: organizationId ?? undefined, page: 1 })
  }

  return (
    <Flex gap="sm" wrap="wrap">
      {/* ✅ [TXT-2] search declares the intent and the fields it covers */}
      <Input
        debounce
        onChange={handleSearchChange}
        placeholder="Search by name or email"
        value={value.search}
      />
      {/* ✅ [TXT-2] "Filter by…" — the "all" state is the cleared value (null), not the text */}
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

**Scenario 2 — empty and error that steer the next action:** `[TXT-3, TXT-4, TXT-5]`
```tsx
if (users.isError) {
  return (
    <Alert variant="destructive">
      {/* ✅ [TXT-5] the API message shown as it came — never rewritten here */}
      <Alert.Description>{users.error.message}</Alert.Description>
      {/* ✅ [TXT-3, TXT-4] the error offers the way out, verb-first */}
      <Alert.Action>
        <Button onClick={handleRetry} variant="outline">
          Try again
        </Button>
      </Alert.Action>
    </Alert>
  )
}

if (users.data.data.length === 0) {
  return (
    <Flex align="center" direction="col" gap="sm">
      {/* ✅ [TXT-3] states the fact + steers: empty is never a dead end */}
      <Typography variant="muted">No users yet</Typography>
      {/* ✅ [TXT-4] verb-first CTA, sentence case */}
      <Button onClick={handleCreate}>Create user</Button>
    </Flex>
  )
}
```

**Scenario 3 — verb-first buttons and sentence-case titles:** `[TXT-1, TXT-4]`
```tsx
<Modal.Content>
  <Modal.Header closable>
    {/* ✅ [TXT-4] sentence case — never "Edit User Profile" */}
    <Modal.Header.Title>Edit user profile</Modal.Header.Title>
  </Modal.Header>
  <Modal.Body>
    <UserProfileForm mode="edit" onSuccess={close} userId={userId} />
  </Modal.Body>
  <Modal.Footer>
    <Button onClick={close} type="button" variant="ghost">
      Cancel
    </Button>
    {/* ✅ [TXT-4] verb first: the action the click performs */}
    <Button form="user-profile-form" type="submit">
      Save changes
    </Button>
  </Modal.Footer>
</Modal.Content>
```

**Scenario 4 — primitives' internal strings in the root `Provider`:** `[TXT-L1, TXT-1]`
```tsx
// src/routes/__root.tsx — the app's single Provider (see 04-routes-app-shell.md)
<Provider
  translations={{
    confirm: { cancel: 'Cancel', confirm: 'Confirm' },
    label: { optional: 'optional', required: 'required' },
    select: { empty: 'No options found', loadingMore: 'Loading more…' },
    table: { noData: 'No records found' },
  }}
>
  <Outlet />
</Provider>
```

### ❌ Never do

```tsx
// ❌ [TXT-1] a string stranded in another language inside an en-US product
<Button type="submit">Enviar</Button>

// ❌ [TXT-2] option-as-placeholder: looks like a selected value, does not steer the action
<OrganizationSelect placeholder="All organizations" />

// ❌ [TXT-2] empty label that does not say what the control filters
<StatusSelect placeholder="Select" />

// ❌ [TXT-3] dead end: states the fact and abandons the user
<Typography variant="muted">No data</Typography>

// ❌ [TXT-5] rewriting the API error — message already arrives translated and ready
if (users.isError) {
  return <Typography>Something went wrong, try again</Typography>
}

// ❌ [TXT-5] local translation dictionary by code — the backend owns the message
const errorMessages: Record<string, string> = {
  'user.not_found': 'User not found',
}

// ❌ [TXT-4] noun first and Title Case
<Button>New User</Button>
// should be: <Button>Create user</Button>

// ❌ [TXT-L1] a second Provider to "tweak" a translation locally — translations live
// in the single Provider of __root.tsx (which is also law: Provider mounts ONCE)
<Provider translations={{ table: { noData: 'No records' } }}>
  <UsersTable />
</Provider>
```
