# Forms

**Concept.** A form is a business component: it validates with the Zod schema from `~sdk`, submits through the generated mutation and emits success to the parent. The form knows fields and errors; navigation, flow reset and the destination of the result belong to whoever mounts it.

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
  - [❌ Never do](#never-do)

**Rules defined here:** `FRM-1` · `FRM-2` · `FRM-3` · `FRM-4` · `FRM-5` · `FRM-6` · `FRM-7` · `FRM-8` · `FRM-9` · `FRM-10` · `FRM-11` · `FRM-L1` · `FRM-L2` — the law itself is the *Invariants* table below; every ❌ item cites the id it violates.

---

## 🌐 Generic pattern (portable — stack-independent)

**FRM-1 — schema and types come from the API contract.** A form that maps an endpoint validates with the schema imported from the generated SDK; redeclaring the shape by hand (an inline validation schema or a hand-written interface for an API payload) is banned. The form values are typed by **inference** from the schema, never by a manual type. **[FRM-1]**

**FRM-2 — a form is a feature component, never inline in the route.** Every form
lives in `src/features/{feature}/components/{name}-form/`. Routes orchestrate;
they do not render fields nor instantiate form state. A form carries domain
meaning, so it belongs to the feature, never to `ui/`. **[FRM-2]**

**FRM-3 — one component owns the form instance.** A single form-state instance per form, created in a single component. A form with multiple sections shares the instance through context (provider), never by prop-drilling `register`/`control`. **[FRM-3]**

**FRM-4 — create/edit is ONE component.** A domain form is a single one, with `mode="create" | "edit"` — never duplicated `CreateXForm` + `EditXForm`. Conditionals only on the fields whose presence differs between the SDK's create/update schemas. **[FRM-4]**

**FRM-5 — initial values go in at construction.** `defaultValues` is passed straight into the form's creation; never synced afterwards through an effect (post-mount `reset`) nor wrapped in memoization without a measured expensive computation. Async data → the form only mounts once the data exists. **[FRM-5]**

**FRM-6 — submission only through the generated mutation.** The submit calls the SDK's mutation hook; a manual `fetch`/HTTP client inside a form is banned. The hook already handles auth, error normalization and cache invalidation. **[FRM-6]**

**FRM-7 — submit gated while in flight.** The submit button disables while the form **or** the mutation is in flight: `formState.isSubmitting || mutation.isPending`. **[FRM-7]**

**FRM-8 — field name = schema key.** No remapping at the field layer: if the API expects `priceInCents`, the field is `priceInCents`. **[FRM-8]**

**FRM-9 — form-wide server error above the form.** An error that affects the whole form renders **above** the form element; a field error sits next to the field. A form-wide error never shows up after the fields or in the footer. **[FRM-9]**

**FRM-10 — success belongs to the parent.** The form exposes `onSuccess?: (result) => void` and calls it when the mutation completes. Flow reset, navigation and the destination toast belong to whoever mounts it; a form never imports navigation. **[FRM-10]**

**FRM-11 — API error: `message` as it came, branch only on `code`.** Default path: a form-wide error with `error.message` untouched. Branch on `error.code` only for a non-default reaction (targeting a specific field, changing the flow). Never rewrite/re-translate the message the backend sent ready to use. Full contract in 11-error-handling.md. **[FRM-11]**

**FRM-L1 — spread a compatible field.** When the field's contract matches the input primitive, spread it (`{...form.register('name')}` / `{...field}`); never repeat `name`/`onChange`/`onBlur`/`value` by hand. Unwrap manually only when the component's contract requires value normalization. **[FRM-L1]**

**FRM-L2 — client-side validation is composed onto the SDK schema, never written beside it.** The generated schema carries only what OpenAPI can express, so a body the API rejects for a blank name still submits from the browser. The form closes that gap by composing `@turystack/fields` **on top of** the SDK schema — `schema.extend({...})` for a stronger field or a client-only one (`passwordConfirmation`, terms acceptance) — a client-only field is still a field a human types, so it is still a field schema, plus the cross-field refinements. Replacing the SDK schema with a hand-written one is still `FRM-1`. A form with no endpoint at all — a local filter — validates entirely with `@turystack/fields`. **[FRM-L2]**

> Three neighboring laws apply to every form and live in their own sections: a form inside a dismissible container (Sheet/Modal/edit page) wires the unsaved guard — law and contract in 03-components-client-state.md; fields and layout compose the design system's form primitives, never raw HTML — consumption law in 03-components-client-state.md; a field the backend constrains to be unique is not validated by the form but by its own `{Entity}{Field}Input`, which owns the contract's availability read — law in `COM-10`, and the form's part of it is mapping the verdict to `setError`/`clearErrors` and adding `checking` to the `FRM-7` gate.

### Invariants (the law the gates enforce)


| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| FRM-1 | Form schema and types come from `~sdk`; hand-written Zod for an API payload and manual interfaces banned | constitutional | `gate:sdk-shadow` | Scenario 1 / ❌ |
| FRM-2 | Form lives in `features/{feature}/components/{name}-form/`; never inline in a route | constitutional | `gate:folder-shape` | Scenario 2 / ❌ |
| FRM-3 | One component owns `useForm`; multi-section via `FormProvider`, never prop-drilling `register`/`control` | constitutional | `manual` | Scenario 3 |
| FRM-4 | Create/edit = one component with `mode`; conditional only on a field that differs between schemas | constitutional | `manual` | Scenario 1 / ❌ |
| FRM-5 | `defaultValues` straight into construction; no `useEffect`+`reset`, no `useMemo` | constitutional | `grit:no-reset-effect` | Scenario 1 / ❌ |
| FRM-6 | Submission only through the SDK's generated mutation; never `fetch` | constitutional | `gate:no-fetch-outside-client` | Scenario 1 / ❌ |
| FRM-7 | Submit disabled while `isSubmitting \|\| isPending` | constitutional | `manual` | Scenario 1 / ❌ |
| FRM-8 | Field name = SDK schema key; no remapping | constitutional | `manual` | Scenario 1 / ❌ |
| FRM-9 | Form-wide error above the `<Form>`; field error next to the field | constitutional | `manual` | Scenario 1 / ❌ |
| FRM-10 | Success emitted to the parent via `onSuccess?`; the form never navigates | constitutional | `grit:no-navigate-in-form` | Scenarios 1–2 / ❌ |
| FRM-11 | `error.message` as it came; branch only on `error.code` | constitutional | `grit:no-branch-on-message` | Scenario 1 / ❌ |
| FRM-L1 | Spread of a compatible RHF field; manual wiring banned | stack lint | `manual` | Scenario 1 / ❌ |
| FRM-L2 | Client-side rules composed onto the SDK schema with `@turystack/fields`; never a parallel hand-written schema | stack lint | `manual` | Scenario 4 / ❌ |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

> Code that implements the rules above in this stack. **Swapping stacks rewrites only this part.** Each block inherits the id of the rule it demonstrates.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated. **Never copy a comment into real code**: the standard is zero comments (see Overview).

**Mechanisms per rule:**

- **FRM-1** — Kubb generates a Zod schema + type per endpoint in `src/~sdk/` (`createUserSchema`, `updateUserSchema`); resolver = `standardSchemaResolver(schema)` from `@hookform/resolvers/standard-schema`; values = `z.infer<typeof schema>`. A schema missing after `api:generate` = **backend blocker** (see 02-sdk.md). A form with no endpoint (e.g. a purely local filter) may have its own Zod — an API payload, never.
- **FRM-2** — the `src/features/users/components/user-form/` folder with
  `user-form.tsx` + `index.ts`; `features/users/index.ts` re-exports it and the
  route imports `@/features/users`. A form knows the domain → feature, never `src/ui/`.
- **FRM-3** — the owner renders `<FormProvider {...form}>`; each section consumes it with `useFormContext<Values>()`.
- **FRM-4** — `mode` discriminates the props (`{ mode: 'edit'; userId: string }`), picks schema and mutation; a create-only field renders under `props.mode === 'create'`.
- **FRM-5** — edit with async data: the parent only mounts the form once the query has resolved (Skeleton before that — see 07-ui-states-and-feedback.md); the form is born with `defaultValues` ready.
- **FRM-6** — Kubb's mutation hooks (`useCreateUser`, `useUpdateUser`; a custom method comes generated too: `usePayInvoice`, never a manual URL). Cache invalidation in 02-sdk.md.
- **FRM-7** — `const submitting = form.formState.isSubmitting || mutation.isPending`; `<Button disabled={submitting} loading={submitting} type="submit">`. A form with a uniqueness-checked field adds that field's `checking` status to the same expression (`COM-10`) — otherwise the user submits before the verdict lands.
- **FRM-8** — `register` uses the schema's exact key; format conversion happens in the input primitive (`CurrencyInput`, `MaskInput`…), never by renaming the field.
- **FRM-9** — `<Alert variant="destructive">` with `errors.root.message` **before** the `<Form>`; field error via the `error` prop of `Form.Field`.
- **FRM-10** — `onSuccess?.(result)` inside the mutation's `onSuccess`; the route turns it into `navigate` (see 04-routes-app-shell.md). A form never imports `useNavigate`/`Route`.
- **FRM-11** — every API error body is the `Exception` model `{ statusCode, code, message, ...metadata }` with a stable snake_case `code`. Default: `form.setError('root', { message: error.message })`; branch on `error.code` only for a non-default reaction (`setError('email', …)`). Full contract in 11-error-handling.md.
- **FRM-L1** — `<Input {...form.register('name')} />`; `Controller` only when the primitive requires a controlled value, and even then `<Select {...field} />` spreading the field.
- **FRM-L2** — `createUserSchema.extend({ document: CpfSchema(), passwordConfirmation: RequiredStringSchema() }).superRefine(MatchFieldRefine('password', 'passwordConfirmation'))`; the resolver keeps taking one schema. Client-side messages resolve from `FieldIssueCode` through the app's own table — that is **not** `FRM-11`, which governs the message the **server** sent and forbids rewriting it. A validation error that never reached the API has no server message to preserve.
- **Composition** — fields use `Form` / `Form.Field` / `Form.FieldSet` from `@turystack/react-web`: `label`, `description` and `error` are props of `Form.Field`; never raw `<form>`/`<label>` (see 03-components-client-state.md).
- **Dismissible container** — a Sheet/Modal/edit page with a form wires `useUnsaved` fed by `form.formState.isDirty` — law and contract in 03-components-client-state.md.

### ✅ How to do it

**Scenario 1 — single create/edit form with `mode`:** `[FRM-1, FRM-4, FRM-5, FRM-6, FRM-7, FRM-8, FRM-9, FRM-10, FRM-11, FRM-L1]`
```tsx
// src/features/users/components/user-form/user-form.tsx
import { standardSchemaResolver } from '@hookform/resolvers/standard-schema'
import { Alert, Button, Flex, Form, Input, PasswordInput } from '@turystack/react-web'
import { useForm } from 'react-hook-form'
import type { z } from 'zod'

import type { Exception } from '@/~sdk/types'
import {
  createUserSchema,
  updateUserSchema,
  useCreateUser,
  useUpdateUser,
} from '@/~sdk/users'
import type { User } from '@/~sdk/users'

// FRM-1: values inferred from the ~sdk schema — never a hand-written interface
type UserFormValues = z.infer<typeof createUserSchema>

// FRM-4: mode discriminates the props — edit requires userId, create does not
type UserFormProps = {
  defaultValues?: Partial<UserFormValues>
  onCancel?: () => void
  onSuccess?: (user: User) => void
} & ({ mode: 'create' } | { mode: 'edit'; userId: string })

export function UserForm(props: UserFormProps) {
  const { defaultValues, onCancel, onSuccess } = props

  const form = useForm<UserFormValues>({
    // FRM-5: defaultValues straight into construction — no useEffect+reset, no useMemo
    defaultValues: {
      email: defaultValues?.email ?? '',
      name: defaultValues?.name ?? '',
      password: '',
    },
    // FRM-1: schema from ~sdk; FRM-4: the mode picks create vs update
    resolver: standardSchemaResolver(
      props.mode === 'create' ? createUserSchema : updateUserSchema,
    ),
  })

  function handleSuccess(user: User) {
    // FRM-10: the form only emits; flow reset and navigation belong to the parent
    onSuccess?.(user)
  }

  function handleError(error: Exception) {
    // FRM-11: message shown as it came; branch only on the code (see 11-error-handling.md)
    if (error.code === 'user.email_already_taken') {
      form.setError('email', { message: error.message })
      return
    }
    form.setError('root', { message: error.message })
  }

  // FRM-6: generated mutations only; hooks always at the top — the mode branch
  // happens in the handler, never in the hook call (see 03-components-client-state.md)
  const createUser = useCreateUser({
    mutation: { onError: handleError, onSuccess: handleSuccess },
  })
  const updateUser = useUpdateUser({
    mutation: { onError: handleError, onSuccess: handleSuccess },
  })

  function handleSubmit(values: UserFormValues) {
    if (props.mode === 'create') {
      createUser.mutate({ data: values })
      return
    }
    updateUser.mutate({ data: values, userId: props.userId })
  }

  // FRM-7: in-flight gate — form OR mutation
  const submitting =
    form.formState.isSubmitting || createUser.isPending || updateUser.isPending
  const rootError = form.formState.errors.root

  return (
    <>
      {rootError && (
        // FRM-9: form-wide error ABOVE the <Form>; a field error stays in the Form.Field
        <Alert variant="destructive">
          <Alert.Content>
            <Alert.Description>{rootError.message}</Alert.Description>
          </Alert.Content>
        </Alert>
      )}
      <Form onSubmit={form.handleSubmit(handleSubmit)}>
        <Form.Field
          error={form.formState.errors.name?.message}
          label={{ content: 'Name', required: true }}
          name="name"
        >
          {/* FRM-L1 + FRM-8: register spread with the schema's exact key */}
          <Input id="name" {...form.register('name')} />
        </Form.Field>

        <Form.Field
          error={form.formState.errors.email?.message}
          label={{ content: 'Email', required: true }}
          name="email"
        >
          <Input id="email" type="email" {...form.register('email')} />
        </Form.Field>

        {props.mode === 'create' && (
          // FRM-4: conditional only on a field that differs between createUserSchema and updateUserSchema
          <Form.Field
            error={form.formState.errors.password?.message}
            label={{ content: 'Password', required: true }}
            name="password"
          >
            <PasswordInput id="password" {...form.register('password')} />
          </Form.Field>
        )}

        <Flex gap="sm" justify="end">
          <Button onClick={onCancel} type="button" variant="ghost">
            Cancel
          </Button>
          <Button disabled={submitting} loading={submitting} type="submit">
            Save
          </Button>
        </Flex>
      </Form>
    </>
  )
}
```

**Scenario 2 — the route mounts the form and owns navigation:** `[FRM-2, FRM-10]`
```tsx
// src/routes/_app/users/new.tsx — the page shell (Page, breadcrumbs) is in 04-routes-app-shell.md
import { createFileRoute } from '@tanstack/react-router'

import { UserForm } from '@/features/users'
import type { User } from '@/~sdk/users'

export const Route = createFileRoute('/_app/users/new')({
  component: NewUserPage,
})

function NewUserPage() {
  const navigate = Route.useNavigate()

  function handleCancel() {
    navigate({ to: '/users' })
  }

  function handleSuccess(user: User) {
    // FRM-10: the form emitted; the route decides the destination
    navigate({ params: { userId: user.userId }, to: '/users/$userId' })
  }

  // FRM-2: the route has no useForm and no fields — it only orchestrates
  return <UserForm mode="create" onCancel={handleCancel} onSuccess={handleSuccess} />
}
```

**Scenario 3 — multi-section form via `FormProvider`:** `[FRM-1, FRM-3]`
```tsx
// src/features/organizations/components/organization-form/organization-form.tsx — the single owner of the instance
import { standardSchemaResolver } from '@hookform/resolvers/standard-schema'
import { Form } from '@turystack/react-web'
import { FormProvider, useForm } from 'react-hook-form'
import type { z } from 'zod'

import { createOrganizationSchema } from '@/~sdk/organizations'

export type OrganizationFormValues = z.infer<typeof createOrganizationSchema>

export function OrganizationForm({ onSuccess }: OrganizationFormProps) {
  const form = useForm<OrganizationFormValues>({
    defaultValues: { city: '', legalName: '', tradeName: '' },
    resolver: standardSchemaResolver(createOrganizationSchema),
  })

  return (
    // FRM-3: there is a single instance; sections consume it by context, never by prop-drilling
    <FormProvider {...form}>
      <Form onSubmit={form.handleSubmit(handleSubmit)}>
        <OrganizationIdentitySection />
        <OrganizationAddressSection />
        ...
      </Form>
    </FormProvider>
  )
}
```
```tsx
// src/features/organizations/components/organization-form/organization-address-section.tsx
import { Form, Input } from '@turystack/react-web'
import { useFormContext } from 'react-hook-form'

import type { OrganizationFormValues } from './organization-form'

export function OrganizationAddressSection() {
  const form = useFormContext<OrganizationFormValues>()

  return (
    <Form.FieldSet legend="Address">
      <Form.Field
        error={form.formState.errors.city?.message}
        label={{ content: 'City', required: true }}
        name="city"
      >
        <Input id="city" {...form.register('city')} />
      </Form.Field>
    </Form.FieldSet>
  )
}
```

**Scenario 4 — the SDK schema strengthened for the browser:** `[FRM-1, FRM-L2, FRM-8, FRM-9]`

```tsx
import {
  MatchFieldRefine,
  MustAcceptSchema,
  PasswordSchema,
  RequiredStringSchema,
} from '@turystack/fields'
import { CpfSchema } from '@turystack/fields/br'
import { standardSchemaResolver } from '@hookform/resolvers/standard-schema'
import { z } from 'zod'

import { createUserSchema } from '~sdk'

// FRM-1: the payload shape still comes from the SDK — this composes onto it,
// it does not redeclare it
const formSchema = createUserSchema
  .extend({
    // FRM-L2: OpenAPI cannot express a CPF check digit, so the browser would
    // otherwise post an invalid document and learn about it from a 422
    document: CpfSchema(),
    password: PasswordSchema(),
    // client-only: never leaves the browser, so it has no SDK counterpart
    acceptedTerms: MustAcceptSchema(),
    // FRM-L2: still a field a human types, so it is still a field schema —
    // z.string() accepts '   ' and reports `too_small` instead of `required`
    passwordConfirmation: RequiredStringSchema(),
  })
  .superRefine(MatchFieldRefine('password', 'passwordConfirmation'))

type Values = z.infer<typeof formSchema>

const form = useForm<Values>({ resolver: standardSchemaResolver(formSchema) })

// FRM-9: the error sits next to its field — MatchFieldRefine reports on
// passwordConfirmation, not on the form root
<Form.Field error={form.formState.errors.passwordConfirmation?.message} label="Confirm password">
  <Input type="password" {...form.register('passwordConfirmation')} />
</Form.Field>
```

The submit sends only what the endpoint declares: strip the client-only keys
(`passwordConfirmation`, `acceptedTerms`) at the mutation call, never by making
the form's shape diverge from the SDK's.

### ❌ Never do

```tsx
// ❌ [FRM-1] redeclaring the schema the ~sdk already exports
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
})

// ❌ [FRM-L2] hand-rolled field rules in a form — each one has a case it misses
name: z.string().min(1)        // '   ' submits; use RequiredStringSchema()
price: z.coerce.number()       // '' becomes 0; use MoneySchema()
birthDate: z.coerce.date()     // the day moves in a negative-offset zone; use DateOnlySchema()
document: z.string().length(11) // 111.111.111-11 passes; use CpfSchema()

// ❌ [FRM-L2] a second schema beside the SDK's instead of composing onto it
const clientSchema = z.object({ ...everythingAgain })  // use createUserSchema.extend({...})

// ❌ [FRM-L2] matching a validation message instead of its code
if (error.message.includes('Too small')) { /* breaks on the next zod release */ }

// ❌ [FRM-1] hand-written interface for API form values — the type is inferred from the schema
interface UserFormValues {
  email: string
  name: string
}

// ❌ [FRM-2] useForm inside the route file — a route orchestrates, it does not render fields
function UsersNewPage() {
  const form = useForm<UserFormValues>({ resolver: standardSchemaResolver(createUserSchema) })
  return <Form onSubmit={form.handleSubmit(handleSubmit)}>...</Form>
}

// ❌ [FRM-4] one component per mode — duplication that drifts apart
export function CreateUserForm() { ... }
export function EditUserForm() { ... }

// ❌ [FRM-5] defaultValues synced through an effect
useEffect(() => {
  form.reset(user)
}, [user])

// ❌ [FRM-5] useMemo just to stabilize simple defaultValues
const defaultValues = useMemo(() => ({ name: user.name }), [user])

// ❌ [FRM-6] manual fetch in the submit — the generated mutation is the only door
function handleSubmit(values: UserFormValues) {
  fetch('/api/users', { body: JSON.stringify(values), method: 'POST' })
}

// ❌ [FRM-8] remapping the field name — the API expects priceInCents
<Input {...form.register('price')} />

// ❌ [FRM-10] a form navigating on its own — success belongs to the parent
const navigate = useNavigate()
const createUser = useCreateUser({
  mutation: { onSuccess: () => navigate({ to: '/users' }) },
})

// ❌ [FRM-11] re-translating the message the backend sent ready to use
if (error.code === 'user.email_already_taken') {
  form.setError('email', { message: 'This email is already in use.' })
}

// ❌ [FRM-L1] manual wiring where the spread does the job
<Input
  name="name"
  onBlur={() => form.trigger('name')}
  onChange={(event) => form.setValue('name', event.target.value)}
  value={form.watch('name')}
/>

// ❌ [FRM-7, FRM-9] submit with no in-flight gate and a form-wide error after the fields
<Form onSubmit={form.handleSubmit(handleSubmit)}>
  ...
  {rootError && <Alert variant="destructive">...</Alert>}
  <Button type="submit">Save</Button>
</Form>
```
