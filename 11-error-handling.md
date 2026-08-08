# Error Handling

**Concept.** The frontend does not invent errors: it consumes the single `Exception` contract the backend publishes — showing `message` as it came, branching only on `code` — and guarantees that no failure disappears: every thrown or rejected error becomes visible feedback (field, toast or boundary).

> **How to read this file.** Two parts, deliberately separated:
> - **🌐 Generic pattern** — the **portable law**. Holds in any stack (React, Vue, etc.); the text stays true even after swapping frameworks. This is what review enforces as an invariant.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack (TypeScript · React · TanStack · @turystack). Swapping stacks rewrites **only** this part; the law above does not change.
>
> Each rule carries an id `ERR-n` linking the law (generic) to the code (specific). Gates bind by **id**, not by file/line. `ERR-L*` rules are **stack lints**: they exist only because of the current language's ergonomics (they invert in another stack), but they stay id-registered and enforced in this project.

---

## 🌐 Generic pattern (portable — stack-independent)

**ARC-ERR-2 — single `Exception` contract.** Every API error body is one single model: `{ statusCode, code, message, ...metadata }`. The client consumes **one** error type derived from the contract — never an error shape invented per endpoint, never an ad-hoc parse of the body. **[ARC-ERR-2]**

**ERR-2 — `message` is shown as it came.** `error.message` arrives ready for display — the backend already produced the final text. The frontend renders it as is; it never rewrites, re-translates nor replaces the API message with a string of its own. **[ERR-2]**

**ARC-ERR-6 — branch only on `code`.** A special reaction to an error (targeting a field, changing flow, retrying, suppressing a toast) branches exclusively on `error.code` — a stable `snake_case` key prefixed by module (`order.not_found`). The default path of a mutation is a form root error or a toast, always showing `error.message`. **[ARC-ERR-6]**

**ARC-ERR-7 — an error never vanishes.** Every thrown/rejected error has a visible outcome: inline feedback, toast or boundary. An error with no display surface is a bug, not a "rare case". **[ARC-ERR-7]**

**ARC-ERR-7 — no swallowed `catch`.** An empty `catch` (or error callback), or one that only logs, is banned. Handling = displaying, reacting, or re-throwing to the layer that displays. **[ARC-ERR-7]**

**ERR-6 — a boundary per route subtree.** Every route subtree is covered by an error boundary; the mandatory minimum is the boundary at the root. A render error never takes the app down to a white screen. **[ERR-6]**

**ERR-7 — expected ≠ unexpected.** An expected business error (`Exception` 4xx) → inline/field feedback at the point of the action. An unexpected failure (5xx, network) → toast + boundary + report to observability (see `14-telemetry.md`). Hand-written copy is only legitimate when **no** `Exception` exists (network failure, render error) — there is no API message to betray there. **[ERR-7]**

### Invariants (the law the gates enforce)

> Bind by **id**. `constitutional` = portable invariant (holds in any stack). `stack lint` = exists only because of the current language's ergonomics and inverts in another stack — still id-registered and enforced here. The **detector** (how the gate catches the violation) is project-specific and lives in the 🛠️ part below.

| ID | Law (one line) | Type | Detector (🛠️) |
|---|---|---|---|
| ERR-2 | `error.message` renders as it came; never re-translate/rewrite API error text | constitutional | Scenarios 1–2 / ❌ |
| ERR-6 | Route subtree under an error boundary; the root at minimum | constitutional | Scenario 3 / ❌ |
| ERR-7 | Expected 4xx → inline/field; 5xx/network → toast + boundary + report | constitutional | Scenarios 3–4 |
| ERR-L1 | An error outside the SDK is `unknown`: narrow before touching `.message`; never a blind cast | stack lint | ❌ |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law |
|---|---|
| `ARC-ERR-2` | The code is the contract; the message is human. |
| `ARC-ERR-5` | Internal detail never reaches the client. |
| `ARC-ERR-6` | The consumer branches on code, never on message. |
| `ARC-ERR-7` | Every error becomes visible feedback or a propagated failure. |


---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into the code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **ARC-ERR-2** — this file is the consumer mirror of **Error Handling** from the `backend-pattern` skill: there the catalogue (`createExceptions` from `@turystack/exceptions`) produces the codes; `Server.create`'s `AppErrorTransform` serializes every error as `{ statusCode, code, message, ...metadata }` and registers the **`Exception`** model in the OpenAPI `components.schemas`. Kubb then generates **one** `Exception` type in `src/~sdk/` (the existence of the symbol is law in `02-sdk.md`). Metadata per category: `errors[]` (400, validation issues with `path`), `resourceId`/`resourceIds` (404). The API may also expose the union of all possible codes (`InferExceptionCodes` in the backend) — when exposed, the branch on `code` becomes exhaustive in the type system.
- **ERR-2 / ARC-ERR-6** — in the SDK's mutation hooks, `onError` receives the `Exception`. In a form, the default is `form.setError('root', { message: error.message })` rendered as an `Alert` above the `<Form>` (layout is law in `05-forms.md`); outside a form, `toast.error(error.message)` (`toast` from `@turystack/react-web`). Branching on `code` covers the rest: `setError('email', ...)`, redirecting, retrying.
- **ARC-ERR-7 / ARC-ERR-7** — a query with an error lands in the screen's `error` reality (matrix of the five realities — see `07-ui-states-and-feedback.md`); a mutation always declares a destination for the error (`onError` → root/toast). A local `try/catch` only exists when there is a local reaction; with no reaction, you do not capture — let it propagate to the boundary.
- **ERR-6** — TanStack Router: `errorComponent` in `__root.tsx` is the minimum; subtrees with their own recovery declare `errorComponent` on the parent route (`createFileRoute('/_app/orders')({ errorComponent })`).
- **ERR-7** — `statusCode < 500` = expected business error → inline/field with `error.message`. `statusCode >= 500` or a network failure = unexpected → toast/boundary + report (the report mechanics belong to `14-telemetry.md`). With no `Exception` (network, render) there is no API message — generic product copy is legitimate there, and only there.
- **ERR-L1** — an error coming from the SDK hooks already arrives typed as `Exception`; an error captured outside the SDK (`catch` in a util, browser API) is `unknown` — narrow (`instanceof Error`, guard) before touching any field; a blind `as Exception` is banned.

### ✅ How to do it

**Scenario 1 — form mutation: default `root`, branch only on `code`:** `[ARC-ERR-2, ERR-2, ARC-ERR-6]`
```tsx
// src/features/users/components/user-form/user-form.tsx — the message comes ready from the backend;
// the stable code decides ONLY the destination (field vs root)
import { standardSchemaResolver } from '@hookform/resolvers/standard-schema'
import { useForm } from 'react-hook-form'
import type { z } from 'zod'

import { Alert, Button, Form, Input } from '@turystack/react-web'

import type { Exception } from '@/~sdk/types'
import { createUserSchema, useCreateUser } from '@/~sdk/users'

import type { UserFormProps } from './user-form.types'

type UserFormValues = z.infer<typeof createUserSchema>

export function UserForm({ onSuccess }: UserFormProps) {
  const form = useForm<UserFormValues>({
    defaultValues: { email: '', name: '' },
    resolver: standardSchemaResolver(createUserSchema),
  })

  function handleError(error: Exception) {
    if (error.code === 'user.email_already_taken') {
      form.setError('email', { message: error.message })
      return
    }
    form.setError('root', { message: error.message })
  }

  const mutation = useCreateUser({
    mutation: { onError: handleError, onSuccess },
  })

  function handleSubmit(values: UserFormValues) {
    mutation.mutate({ data: values })
  }

  const submitting = form.formState.isSubmitting || mutation.isPending

  return (
    <>
      {form.formState.errors.root && (
        <Alert variant="destructive">
          <Alert.Description>{form.formState.errors.root.message}</Alert.Description>
        </Alert>
      )}
      <Form onSubmit={form.handleSubmit(handleSubmit)}>
        <Form.Field error={form.formState.errors.name?.message} label="Name">
          <Input {...form.register('name')} />
        </Form.Field>
        <Form.Field error={form.formState.errors.email?.message} label="Email">
          <Input type="email" {...form.register('email')} />
        </Form.Field>
        <Button disabled={submitting} loading={submitting} type="submit">
          Save
        </Button>
      </Form>
    </>
  )
}
```

**Scenario 2 — action outside a form: toast with the API message:** `[ARC-ERR-2, ERR-2, ARC-ERR-7]`
```tsx
// src/features/invoices/components/invoice-actions/invoice-actions.tsx — custom method already generated
// by the SDK; the error ALWAYS has a visible destination
import { toast } from '@turystack/react-web'

import { usePayInvoice } from '@/~sdk/invoices'
import type { Exception } from '@/~sdk/types'

export function useInvoicePay(onSuccess?: () => void) {
  function handleError(error: Exception) {
    toast.error(error.message)
  }

  function handleSuccess() {
    toast.success('Invoice paid.')
    onSuccess?.()
  }

  return usePayInvoice({
    mutation: { onError: handleError, onSuccess: handleSuccess },
  })
}
```

**Scenario 3 — boundary at the root: an unexpected failure never becomes a white screen:** `[ARC-ERR-7, ERR-6, ERR-7]`
```tsx
// src/routes/__root.tsx — mandatory minimum coverage; subtrees with their own
// recovery declare their errorComponent on the parent route
import { createRootRoute, type ErrorComponentProps } from '@tanstack/react-router'

import { Alert, Button } from '@turystack/react-web'

export const Route = createRootRoute({
  component: RootLayout,
  errorComponent: RootError,
})

function RootError({ reset }: ErrorComponentProps) {
  return (
    <Alert variant="destructive">
      <Alert.Title>Something went wrong</Alert.Title>
      <Alert.Description>
        {/* unexpected error: there is NO Exception here — product copy is legitimate */}
        Reload the page or try again.
      </Alert.Description>
      <Alert.Action>
        <Button onClick={reset} variant="outline">
          Try again
        </Button>
      </Alert.Action>
    </Alert>
  )
}
```

**Scenario 4 — failure outside the SDK: visible local reaction, never silence:** `[ARC-ERR-7, ARC-ERR-7, ERR-7]`
```tsx
// src/features/invoices/components/invoice-number/invoice-number.tsx — a browser API fails with no
// Exception: the catch exists BECAUSE there is a local reaction, and the reaction is visible
import { Button, toast } from '@turystack/react-web'

export function InvoiceNumberCopy({ number }: InvoiceNumberCopyProps) {
  async function handleCopy() {
    try {
      await navigator.clipboard.writeText(number)
      toast.success('Number copied.')
    } catch {
      toast.error('Could not copy the number.')
    }
  }

  return (
    <Button onClick={handleCopy} variant="ghost">
      Copy number
    </Button>
  )
}
```

### ❌ Never do

```tsx
// ❌ [ERR-2] re-translating/rewriting the message the API sent ready
if (error.code === 'user.email_already_taken') {
  form.setError('email', { message: 'Email is already in use.' })
}
form.setError('root', { message: 'Could not save.' })

// ❌ [ARC-ERR-6] branching on the message — text changes; the stable contract is the code
if (error.message.includes('already taken')) {
  form.setError('email', { message: error.message })
}

// ❌ [ARC-ERR-2] error shape invented per endpoint / ad-hoc parse of the body
// (on top of violating 02-sdk: fetch outside the SDK)
const body = (await response.json()) as { error: string }
toast.error(body.error)

// ❌ [ARC-ERR-7] swallowed catch — the error disappears with no feedback at all
try {
  await mutation.mutateAsync({ data: values })
} catch {}

// ❌ [ARC-ERR-7, ARC-ERR-7] "handling" = only logging — nothing visible to the user
const mutation = useDeleteUser({
  mutation: { onError: (error) => console.log(error) },
})

// ❌ [ERR-L1] blind cast from unknown — outside the SDK nobody guarantees the shape
try {
  await navigator.clipboard.writeText(number)
} catch (error) {
  toast.error((error as Exception).message)
}

// ❌ [ERR-6] app with no boundary at all — __root.tsx with no errorComponent and no
// route in the subtree defining its own: any render error becomes a white screen
export const Route = createRootRoute({
  component: RootLayout,
})
```
