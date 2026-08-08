# Testing

**Concept.** Two levels by the nature of the test: **unit** (pure logic — `support` utilities, hooks, mappers) and **component** (render + interaction as the user sees and uses it: role, accessible name, user-event). The component level gives teeth to this constitution's UX laws: render smoke, the five states, permission as absence in the DOM, clean `axe`. A test asserts the **observable contract** — never the implementation.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `TST-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `TST-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**TST-1 — two levels by the nature of the test.** **Unit** tests pure logic with no render: a `support` utility, a hook, a mapper — input → output, with no infrastructure mocking. **Component** tests render + interaction with the data dependencies mocked at the boundary. The levels are not interchangeable: extractable logic is not tested through a render. **[TST-1]**

**TST-2 — render smoke mandatory in a data-bearing component.** Every component that receives or loads data from the API has at least one test that mounts it without crashing using a minimal fixture and finds an element by semantics. That is the floor: it proves the props contract really mounts. **[TST-2]**

**TST-3 — an async component tests the five realities.** Mount with pending, empty list, error, partial data and success — one fixture per state — and assert **intentional non-empty DOM per branch**. This is the render contract behind `07-ui-states-and-feedback.md`: the test is what proves no state was left implicit. **[TST-3]**

**TST-4 — permission = absence in the DOM.** A permission gate is tested by asserting the element **does not exist** in the DOM without the permission (and does exist with it); never an assertion on `disabled`. The law hides, it does not disable (see `12-security-permissions.md`). **[TST-4]**

**TST-5 — semantics, not implementation.** Query by role and accessible name; interact through real user events (click, typing, keyboard); never assert on a CSS class, internal DOM structure or internal state. Refactoring the visuals must not break a test. **[TST-5]**

**TST-6 — clean `axe` in component tests.** The component test runs axe on the rendered state and fails on any violation (see `08-accessibility.md`). Accessibility has a gate, not an opinion. **[TST-6]**

**TST-7 — mock at the boundary the component consumes.** The data dependency is replaced at the exact boundary the component uses — the SDK hook — never at layers below it (HTTP interception) in a component test. Mocking below couples the test to the transport and lies about the contract. **[TST-7]**

**TST-8 — shared harness, never duplicated.** Render with providers, session/permission context and common fixtures are centralized in a single harness; no spec re-declares its own wrapper or setup. **[TST-8]**

> Tests of a **UI primitive** (prop contract, controlled/uncontrolled pair, modes, value delivered in `onChange`) belong to the **frontend-primitives-pattern** skill — this section covers the app: support, hooks and business components. The full API pyramid (integration/e2e) belongs to the backend (see the backend-pattern skill).

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| TST-1 | Two distinct levels: unit (pure logic, no render) and component (render + interaction, data mocked at the boundary) | constitutional | Scenarios 1–2 |
| TST-2 | Every data-bearing component has a render smoke (mounts without crashing + finds an element by semantics) | constitutional | Scenario 2 / ❌ |
| TST-3 | An async component tests the five states (loading/empty/error/partial/success) with intentional DOM per branch | constitutional | Scenario 2 / ❌ |
| TST-4 | A permission gate asserts absence in the DOM; never `disabled` | constitutional | Scenario 3 / ❌ |
| TST-5 | Query by role/accessible name + user-event; never class/internal structure/internal state | constitutional | Scenarios 2–4 / ❌ |
| TST-6 | `axe` runs clean in component tests | constitutional | Scenario 4 |
| TST-7 | Mock at the SDK hook boundary; never intercept HTTP in a component test | constitutional | Scenario 2 / ❌ |
| TST-8 | Shared harness (providers, session, fixtures) centralized; never duplicated per spec | constitutional | Scenario 3 / ❌ |
| TST-L1 | Vitest + Testing Library + user-event; `.test.ts`/`.test.tsx` colocated with the file it tests | stack lint | Scenarios 1–4 / ❌ |
| TST-L2 | SDK mocked via `vi.mock('@/~sdk/...')` + `vi.mocked(...).mockReturnValue(...)` | stack lint | Scenario 2 |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **TST-1** — Vitest with the `jsdom` environment; unit in `.test.ts` next to the
  file (`src/features/users/support/normalize-user-search/normalize-user-search.test.ts`);
  hooks with Testing Library's `renderHook`; components in `.test.tsx` inside the
  component's folder.
- **TST-2 / TST-3** — fixtures **typed by the `~sdk` types** (`makeUser(overrides)`) — never a hand-written shape (see `02-sdk.md`). The five states come out of `vi.mocked(useListUsers).mockReturnValue(...)`: `isPending: true` · `data: []` with `meta` · `isError` + `Exception` · partial data (missing optional) · full data. The asserted error is the `error.message` **as it came** (see `11-error-handling.md`).
- **TST-4** — the harness's `renderWithPermissions` helper mounts the session context with controllable `permissionIds`; assert with `screen.queryByRole(...)` → `not.toBeInTheDocument()`.
- **TST-5** — `screen.getByRole('button', { name: 'Save' })` + `await userEvent.click(...)`; `data-testid` only where there is no natural semantics (e.g. `Skeleton`).
- **TST-6** — `axe` from `vitest-axe`: `expect(await axe(container)).toHaveNoViolations()` on the success state (the densest one).
- **TST-7** — `vi.mock('@/~sdk/users')` at the top of the spec; no MSW/HTTP interception in a component test.
- **TST-8** — harness in `src/test/`: `render.tsx` (mounts `@turystack/react-web`'s `Provider` + session/permissions) and shared fixtures; specs import from `@/test/...`.
- **TST-L1** — suffixes configured in `vitest.config.ts`; colocated file: `user-list/user-list.test.tsx` next to `user-list.tsx`.
- **TST-L2** — `vi.mock` + `vi.mocked` + `mockReturnValue`; the query result stub is minimal, with a fixture cast (acceptable **only** in tests).

### ✅ How to do it

**Scenario 1 — unit: pure support logic, no render:** `[TST-1, TST-L1]`
```tsx
// src/features/users/support/normalize-user-search/normalize-user-search.test.ts
import { describe, expect, it } from 'vitest'

import { normalizeUserSearch } from './normalize-user-search'

describe('normalizeUserSearch', () => {
  it.each([
    { expected: 'Ana', input: '  Ana  ' },
    { expected: undefined, input: '   ' },
  ])('normalizes "$input"', ({ expected, input }) => {
    expect(normalizeUserSearch(input)).toBe(expected)
  })
})
```

**Scenario 2 — async component: mock at the hook boundary + five states + real interaction:** `[TST-1, TST-2, TST-3, TST-5, TST-7, TST-L1, TST-L2]`
```tsx
// src/features/users/components/user-list/user-list.test.tsx — colocated with the component
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, expect, it, vi } from 'vitest'

import type { Exception } from '@/~sdk/types'
import { type User, useListUsers } from '@/~sdk/users'

import { UserList } from './user-list'

// mock at the boundary the component consumes: the SDK hook, never HTTP
vi.mock('@/~sdk/users')

// fixture typed by the ~sdk type — never a hand-written shape (see 02-sdk.md)
const makeUser = (overrides?: Partial<User>): User => ({
  email: 'ana@company.com',
  firstName: 'Ana',
  lastName: 'Souza',
  phone: '+55 11 99999-0000',
  status: 'ACTIVE',
  userId: 'usr-1',
  ...overrides,
})

const pageMeta = {
  limit: 10,
  mode: 'page',
  page: 1,
  totalItems: 1,
  totalPages: 1,
} as const

const listUsersException: Exception = {
  code: 'user.list_failed',
  message: 'Could not load users.',
  statusCode: 500,
}

// minimal stub of the query result — a fixture cast is acceptable ONLY in tests
const makeListUsersResult = (
  overrides: Partial<ReturnType<typeof useListUsers>>,
) =>
  ({
    data: undefined,
    error: null,
    isError: false,
    isPending: false,
    ...overrides,
  }) as ReturnType<typeof useListUsers>

describe('UserList', () => {
  it('loading: shows skeleton', () => {
    vi.mocked(useListUsers).mockReturnValue(
      makeListUsersResult({ isPending: true }),
    )

    render(<UserList search={{ limit: 10, page: 1 }} />)

    // testid only because Skeleton has no natural role
    expect(screen.getByTestId('user-list-skeleton')).toBeInTheDocument()
  })

  it('empty: intentional empty state, not a blank screen', () => {
    vi.mocked(useListUsers).mockReturnValue(
      makeListUsersResult({
        data: { data: [], meta: { ...pageMeta, totalItems: 0, totalPages: 0 } },
      }),
    )

    render(<UserList search={{ limit: 10, page: 1 }} />)

    expect(screen.getByText('No users found')).toBeInTheDocument()
  })

  it('error: shows error.message as it came from the API', () => {
    vi.mocked(useListUsers).mockReturnValue(
      makeListUsersResult({ error: listUsersException, isError: true }),
    )

    render(<UserList search={{ limit: 10, page: 1 }} />)

    expect(screen.getByRole('alert')).toHaveTextContent(
      'Could not load users.',
    )
  })

  it('partial: missing optional field renders an intentional fallback', () => {
    vi.mocked(useListUsers).mockReturnValue(
      makeListUsersResult({
        data: { data: [makeUser({ phone: undefined })], meta: pageMeta },
      }),
    )

    render(<UserList search={{ limit: 10, page: 1 }} />)

    expect(screen.getByRole('cell', { name: '—' })).toBeInTheDocument()
  })

  it('success: accessible row + real interaction delivers the domain value', async () => {
    const user = makeUser()
    const onUserSelect = vi.fn()
    vi.mocked(useListUsers).mockReturnValue(
      makeListUsersResult({ data: { data: [user], meta: pageMeta } }),
    )

    render(
      <UserList onUserSelect={onUserSelect} search={{ limit: 10, page: 1 }} />,
    )
    await userEvent.click(screen.getByRole('button', { name: 'Ana Souza' }))

    expect(onUserSelect).toHaveBeenCalledWith(user)
  })
})
```

**Scenario 3 — permission: absence in the DOM via the shared harness:** `[TST-4, TST-5, TST-8]`
```tsx
// src/features/users/components/user-actions/user-actions.test.tsx
import { screen } from '@testing-library/react'
import { expect, it } from 'vitest'

// single harness in src/test/ — never re-declare a wrapper per spec
import { renderWithPermissions } from '@/test/render'
import { makeUser } from '@/test/fixtures/user'

import { UserActions } from './user-actions'

it('hides Delete without permission (hides, does not disable)', () => {
  renderWithPermissions(<UserActions user={makeUser()} />, {
    permissionIds: ['users.read'],
  })

  expect(
    screen.queryByRole('button', { name: 'Delete' }),
  ).not.toBeInTheDocument()
})

it('shows Delete with permission', () => {
  renderWithPermissions(<UserActions user={makeUser()} />, {
    permissionIds: ['users.read', 'users.delete'],
  })

  expect(screen.getByRole('button', { name: 'Delete' })).toBeInTheDocument()
})
```

**Scenario 4 — clean axe on the rendered state:** `[TST-5, TST-6]`
```tsx
it('has no accessibility violations', async () => {
  vi.mocked(useListUsers).mockReturnValue(
    makeListUsersResult({ data: { data: [makeUser()], meta: pageMeta } }),
  )

  const { container } = render(<UserList search={{ limit: 10, page: 1 }} />)

  expect(await axe(container)).toHaveNoViolations()
})
```

### ❌ Never do

```tsx
// ❌ [TST-5] asserting implementation — class/internal structure break on a visual refactor
expect(button.className).toContain('bg-primary')
expect(container.querySelector('div > span > svg')).toBeTruthy()

// ❌ [TST-5] synthetic dispatch instead of a real user event
fireEvent.click(screen.getByRole('button'))

// ❌ [TST-7] intercepting HTTP in a component test (the mock goes at the hook boundary)
server.use(http.get('/api/users', () => HttpResponse.json({ data: [] })))

// ❌ [TST-4] permission tested as disabled (the law is absence in the DOM — see 12-security-permissions.md)
expect(screen.getByRole('button', { name: 'Delete' })).toBeDisabled()

// ❌ [TST-3] async component on the happy path only — loading/empty/error/partial missing
it('renders users', () => {})

// ❌ [TST-2] data-bearing component with no mount test at all
// user-list/ with no user-list.test.tsx

// ❌ [TST-8] provider wrapper duplicated in the spec (it comes from the harness in src/test/)
const wrapper = ({ children }: PropsWithChildren) => (
  <Provider>{children}</Provider>
)

// ❌ [TST-L1] wrong suffix / test far from the file it tests
// src/__tests__/user-list.spec.tsx
```

> ⚠️ Project stance: during feature delivery, broad coverage may be **deferred** — the minimum validation is `pnpm typecheck` + `pnpm check`. The standards above hold from the moment a test is written: when it exists, the smoke, the five states and the absence-in-the-DOM are not optional.
