# Accessibility

**Concept.** Accessibility is not final polish — keyboard, screen reader and focus are part of the contract of every interactive element, in every layer. Half the law is mechanical (a11y lint + axe in component tests); the rest is review judgment about focus and motion.

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `ACC-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `ACC-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**ACC-1 — universal accessible name.** Every interactive element has an accessible name: visible text, an associated label or an explicit name. An icon-only button with no name does not ship — the name describes the action together with its target ("Delete user", not "Delete"). **[ACC-1]**

**ACC-2 — real control, never a clickable container.** Click/keyboard interaction lives in a genuinely semantic control (button, link, menu item) — never a click handler on a `div`/`span`. A real control delivers focus, keyboard and semantics for free; a clickable container delivers none of the three. **[ACC-2]**

**ACC-3 — input with an associated label.** Every input is programmatically associated with a label. A placeholder is not a label: it disappears as you type and a screen reader does not announce it as the field name. **[ACC-3]**

**ACC-4 — focus order is document order.** Positive `tabindex` is banned. If the tab order is wrong, the fix is reordering the structure, never hijacking the sequence with a manual index. **[ACC-4]**

**ACC-5 — focus in an overlay: trap, order and restore.** An overlay (modal, sheet, menu) traps focus while open, opens at a predictable point and returns focus to the element that opened it when it closes. The overlay primitive is what implements this — the law in the app is not to fight it. **[ACC-5]**

**ACC-6 — motion respects `prefers-reduced-motion`.** Every non-essential animation reduces or turns off when the user asked for less motion. **[ACC-6]**

**ACC-7 — clean axe as the mechanical floor.** A component test of an interactive surface runs axe and passes with no violations. The mechanical floor does not replace ACC-5 (focus is judgment above the lint), but nothing ships below it. **[ACC-7]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| ACC-1 | Every interactive element has an accessible name; icon-only demands an explicit name | constitutional | Scenarios 1, 3 / ❌ |
| ACC-2 | Click/keyboard live in a real control (button, link, menu item); never `div`/`span` with a handler | constitutional | Scenario 1 / ❌ |
| ACC-3 | Every input programmatically associated with a label; a placeholder is not a label | constitutional | Scenario 2 / ❌ |
| ACC-4 | Positive `tabindex` banned; focus order = document order | constitutional | ❌ |
| ACC-5 | An overlay traps focus, opens at a predictable point and restores to the trigger on close | constitutional | Scenario 3 / ❌ |
| ACC-6 | Non-essential motion respects `prefers-reduced-motion` | constitutional | ❌ |
| ACC-7 | An interactive surface passes axe clean in a component test | constitutional | (mechanics in 13-testing.md) |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **ACC-1** — the primary source of the accessible name is the control's **visible text** (`<Button>Edit</Button>`, `leftSection` + label). An icon-only control (`size="icon-*"`) needs the explicit name: wrap it with `Tooltip content="…"` (the visible affordance) and guarantee the accessible name — if the `Button` from `@turystack/react-web` does not yet expose a name channel for icon-only, that is a gap to close in the lib (UIP-4, see 03-components-client-state.md), never ship an icon-only button with no name. Icons (`lucide-react`) inside a primitive are decorative — the name comes from the text/Tooltip, never from the icon.
- **ACC-2** — composing the react-web inventory (`Button variant="ghost"`, `DropdownMenu.Item`, `Tabs`) delivers semantics, focus and keyboard for free. A clickable `div` is a double violation: of ACC-2 and of the composition law (raw HTML where a primitive exists — see 03-components-client-state.md).
- **ACC-3** — `Form.Field` with `name` + `label` wires `htmlFor` to the registered input automatically (see 05-forms.md); a field outside a form uses an explicit `Label htmlFor`. Placeholder copy has its own law in 09-content-i18n.md.
- **ACC-4** — Biome's a11y domain (`@turystack/frontend-config`) flags positive `tabindex`; the fix is always to reorder the JSX.
- **ACC-5** — react-web's `Modal`, `Sheet`, `DropdownMenu` and `Popover` already implement trap + restore (Radix underneath). In the app: dismissible always through those primitives, never a positioned `div`; zero manual focus management (`document.querySelector(...).focus()`, competing `autoFocus`) on top of the primitive.
- **ACC-6** — animation lives inside the primitives' `tv()`, and every motion has a `motion-reduce:` counterpart. Missing counterpart in a lib primitive → extend the lib (skill `frontend-primitives-pattern`); an app-local primitive in `src/ui/` follows the same rule in its own `tv()`.
- **ACC-7** — the component test renders the surface and runs axe with no violations; setup, scope and tooling in 13-testing.md.

### ✅ How to do it

**Scenario 1 — actions with an accessible name, in real controls:** `[ACC-1, ACC-2]`
```tsx
import { Button, DropdownMenu } from '@turystack/react-web'
import { EllipsisVertical, Pencil, Trash2 } from 'lucide-react'

type UserRowActionsProps = {
  onEdit: () => void
  onRemove: () => void
}

export function UserRowActions({ onEdit, onRemove }: UserRowActionsProps) {
  return (
    <DropdownMenu>
      {/* ✅ [ACC-1] the name comes from the visible text "Options"; the icon is decorative */}
      <DropdownMenu.Trigger asChild>
        <Button leftSection={<EllipsisVertical />} size="sm" variant="ghost">
          Options
        </Button>
      </DropdownMenu.Trigger>
      {/* ✅ [ACC-2] menu items are real controls — focus, keyboard and semantics for free */}
      <DropdownMenu.Content>
        <DropdownMenu.Item onClick={onEdit}>
          <Pencil />
          Edit
        </DropdownMenu.Item>
        <DropdownMenu.Separator />
        <DropdownMenu.Item onClick={onRemove}>
          <Trash2 />
          Delete
        </DropdownMenu.Item>
      </DropdownMenu.Content>
    </DropdownMenu>
  )
}
```

**Scenario 2 — input always wired to a label via `Form.Field`:** `[ACC-3]`
```tsx
// inside an RHF+Zod form (full contract in 05-forms.md)
<Form onSubmit={form.handleSubmit(handleSubmit)}>
  {/* ✅ [ACC-3] Form.Field wires htmlFor="email" to the registered input — programmatic label */}
  <Form.Field
    error={form.formState.errors.email?.message}
    label="Email"
    name="email"
  >
    <Input type="email" {...form.register('email')} />
  </Form.Field>
</Form>
```

**Scenario 3 — overlay with focus handled by the primitive:** `[ACC-5, ACC-1]`
```tsx
import { useDisclosure } from '@turystack/react-hooks'
import { Button, Modal } from '@turystack/react-web'

export function UserInviteModal() {
  const { opened, open, close } = useDisclosure()

  return (
    <>
      <Button onClick={open}>Invite user</Button>
      {/* ✅ [ACC-5] Modal traps focus while open and returns it to the trigger on close —
          zero manual focus management in the app */}
      <Modal onChange={close} open={opened}>
        <Modal.Content>
          <Modal.Header closable>
            <Modal.Header.Title>Invite user</Modal.Header.Title>
          </Modal.Header>
          <Modal.Body>
            <UserInviteForm onSuccess={close} />
          </Modal.Body>
        </Modal.Content>
      </Modal>
    </>
  )
}
```

### ❌ Never do

```tsx
// ❌ [ACC-2] clickable container — no semantics, no keyboard, no focus
<div onClick={handleOpen}>View details</div>

// ❌ [ACC-1] icon-only button with no accessible name
<Button size="icon-sm" variant="ghost">
  <Trash2 />
</Button>

// ❌ [ACC-3] placeholder playing the role of a label — input with no associated label
<Input placeholder="Email" {...form.register('email')} />

// ❌ [ACC-4] positive tabindex hijacks focus order — reorder the JSX
<Input tabIndex={2} {...form.register('name')} />

// ❌ [ACC-5] manual focus management on top of the primitive (Modal already does trap/restore)
useEffect(() => {
  document.querySelector<HTMLInputElement>('#email')?.focus()
}, [opened])

// ❌ [ACC-5] homemade overlay with no trap/restore instead of Modal/Sheet
{opened && <div>{children}</div>}

// ❌ [ACC-6] motion with no reduced-motion counterpart in an app-local primitive's tv()
const styles = tv({
  slots: {
    root: 't:animate-bounce',
  },
})
// should carry t:motion-reduce:animate-none alongside
```
