# Responsive & Density

**Concept.** A backoffice tool has to stay usable in a narrow window — mobile-first, breakpoints by token and every wide piece of content with a defined collapse behavior. Density is a decision: the container (Confirm, Modal, Sheet, route) is sized by the content, not by habit.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `RSP-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `RSP-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**RSP-1 — mobile-first.** The base layout is the narrow-viewport one; more width is progressive enhancement on top. Desktop-first (a wide base "fixed" with overrides for small screens) inverts the burden and always forgets a case. **[RSP-1]**

**RSP-2 — breakpoint by token, never raw px.** Every breakpoint comes from the design system tokens. A media query with an invented px value (`min-width: 900px`) creates a phantom breakpoint no other component knows about — the layout breaks at different points across screens. **[RSP-2]**

**RSP-3 — wide content defines its own collapse.** Every wide surface (table, toolbar, card grid) explicitly declares how it collapses, stacks or scrolls in a narrow viewport. A fixed px width that overflows the viewport is banned — what scrolls is the content's container, never the page. **[RSP-3]**

**RSP-4 — touch target respects the minimum.** Every interactive target keeps the design system's minimum touch area, even when the visible icon is small. Shrinking the hit area to "make it fit" is banned. **[RSP-4]**

**RSP-5 — container sized by density.** The container of an interaction is chosen by the weight of the content: confirmation → confirmation dialog; short form → modal; detail/vertical column → sheet; dense multi-section editing → dedicated route. Dense content squeezed into a small container (or a lone field lost in a giant one) is a violation. **[RSP-5]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| RSP-1 | Mobile-first: narrow base, width is enhancement — never the other way around | constitutional | Scenario 1 / ❌ |
| RSP-2 | Breakpoints only from design system tokens; raw px in a media query is banned | constitutional | ❌ |
| RSP-3 | Wide content declares collapse/stacking/scroll; a fixed px width that overflows the viewport is banned | constitutional | Scenario 2 / ❌ |
| RSP-4 | Touch target keeps the design system minimum; the hit area never shrinks | constitutional | Scenario 2 / ❌ |
| RSP-5 | Container chosen by density: Confirm → Modal → Sheet → dedicated route | constitutional | Scenario 3 / ❌ (ruler applied in 06-data-surfaces.md) |
| RSP-L1 | `useIsMobile` only for a structural switch CSS cannot express; visual variation stays in the primitives' CSS | stack lint | Scenario 1 / ❌ |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **RSP-1** — the `@turystack/react-web` primitives are mobile-first by construction (e.g. Modal's footer stacks the buttons by default and becomes a row at `sm:`); composing the inventory inherits that for free. An app-local primitive (`src/ui/`) writes the narrow-base `tv()` and widens with a token prefix — never a wide base with a `max-*` override.
- **RSP-2** — the only legal breakpoints are the Tailwind preset tokens (`sm:`/`md:`/`lg:`…) inside `tv()` (lib or `src/ui/`) and the single breakpoint encapsulated by `useIsMobile`. An arbitrary `min-[900px]:`, loose `@media` in CSS and manual `window.matchMedia` are banned.
- **RSP-3** — `Table`: every column declares `width` + truncation and the table container scrolls horizontally (the column law is in 06-data-surfaces.md); toolbar: `Flex wrap="wrap"` declares the stacking; page: max width via `Container`, never a fixed width. There is no `className`/`style` in the app — if a primitive does not collapse the way you need, extend the lib (skill `frontend-primitives-pattern`), never work around it.
- **RSP-4** — touch target = use the primitives' sizes: even `Button`'s `size="icon-sm"` keeps the design system's minimum hit area. A raw clickable icon is a double violation — of RSP-4 and of name/semantics (see 08-accessibility.md).
- **RSP-5** — the density ruler in this stack: `Confirm` to confirm a destructive action; `Modal` for a short form (up to ~2 fields/1 section); `Sheet side="right"` for a record detail/vertical column; a dedicated route for dense multi-section editing. The ruler applied to tables and detail sheets is in 06-data-surfaces.md.
- **RSP-L1** — `useIsMobile` (`@turystack/react-web`) comes in **only** when the switch is structural — a different component tree per viewport (inline filters vs `Sheet`, table vs card list). Gap, padding, direction, columns: the primitives' CSS handles it, and CSS neither re-renders nor flickers on resize.

### ✅ How to do it

**Scenario 1 — structural switch with `useIsMobile`, visuals stay in CSS:** `[RSP-L1, RSP-1]`
```tsx
import { useIsMobile } from '@turystack/react-web'
import type { ListUsersQueryParams } from '@/~sdk/users'

type UsersFiltersProps = {
  value: ListUsersQueryParams
  onChange: (value: ListUsersQueryParams) => void
}

export function UsersFilters({ value, onChange }: UsersFiltersProps) {
  const mobile = useIsMobile()

  {/* ✅ [RSP-L1] a different tree per viewport — this is what CSS cannot express */}
  if (mobile) {
    return <UsersFiltersSheet onChange={onChange} value={value} />
  }

  return <UsersToolbar onChange={onChange} value={value} />
}
```

**Scenario 2 — wide content with a declared collapse:** `[RSP-3, RSP-4]`
```tsx
const columns: TableColumns<User> = [
  {/* ✅ [RSP-3] every column declares width; the Table container scrolls horizontally
      in a narrow viewport — the page never overflows (column law in 06-data-surfaces.md) */}
  { key: 'name', label: 'Name', selector: (user) => user.name, width: 220 },
  { key: 'email', label: 'Email', selector: (user) => user.email, width: 260 },
  // ✅ [RSP-4] actions via Button icon — minimum hit area guaranteed by the primitive
  {
    align: 'right',
    key: 'actions',
    selector: (user) => <UserRowActions userId={user.userId} />,
    width: 64,
  },
]

return (
  <Flex direction="col" gap="md">
    {/* ✅ [RSP-3] the toolbar declares its collapse: wrap stacks the filters when narrow */}
    <Flex gap="sm" wrap="wrap">
      <UsersToolbar onChange={handleSearchChange} value={search} />
    </Flex>
    <Table columns={columns} itemKey="userId" items={users.data.data} />
  </Flex>
)
```

**Scenario 3 — container chosen by density:** `[RSP-5]`
```tsx
{/* ✅ [RSP-5] destructive confirmation → Confirm (see 07-ui-states-and-feedback.md) */}
<Confirm
  description="This action cannot be undone."
  onCancel={removeDisclosure.close}
  onConfirm={handleRemove}
  open={removeDisclosure.opened}
  title="Remove user"
/>

{/* ✅ [RSP-5] record detail (vertical reading column) → side Sheet */}
<Sheet onChange={detailDisclosure.close} open={detailDisclosure.opened} side="right">
  <Sheet.Header closable>
    <Sheet.Title>User details</Sheet.Title>
  </Sheet.Header>
  <Sheet.Body>
    <UserDetail userId={userId} />
  </Sheet.Body>
</Sheet>

{/* ✅ [RSP-5] dense multi-section editing → dedicated route, never an overlay */}
<Button asChild variant="outline">
  <Link params={{ userId }} to="/users/$userId/edit">
    Edit user
  </Link>
</Button>
```

### ❌ Never do

```tsx
// ❌ [RSP-3] fixed px width in an app-local primitive — overflows any smaller viewport
const styles = tv({
  slots: {
    root: 't:w-[960px] t:p-4',
  },
})

// ❌ [RSP-2] phantom breakpoint with raw px instead of the design system token
const styles = tv({
  slots: {
    root: 't:min-[900px]:flex-row',
  },
})

// ❌ [RSP-1] desktop-first: wide base "fixed" for narrow with max-*
const styles = tv({
  slots: {
    root: 't:flex-row t:max-md:flex-col',
  },
})

// ❌ [RSP-L1] viewport hook for a purely visual variation — that is primitive CSS
const mobile = useIsMobile()
return <Flex gap={mobile ? 'sm' : 'lg'}>{children}</Flex>

// ❌ [RSP-2] re-implementing the breakpoint by hand — useIsMobile already encapsulates the token
const [narrow, setNarrow] = useState(window.matchMedia('(max-width: 900px)').matches)

// ❌ [RSP-4] shrunken hit area: raw clickable icon (also violates 08-accessibility.md)
<Trash2 onClick={handleRemove} size={16} />

// ❌ [RSP-5] dense multi-section editing squeezed into a Modal — density asks for a dedicated route
<Modal onChange={close} open={opened}>
  <Modal.Content>
    <Modal.Body>
      <OrderFullEditForm orderId={orderId} />
    </Modal.Body>
  </Modal.Content>
</Modal>
```
