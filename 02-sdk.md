# API Contract & Server State

**Concept.** Every frontend consumes an API; OpenAPI generates its typed door.
The SDK's types, schemas and hooks are canonical; TanStack Query is the only
owner of server state.

## Invariants

| ID | Law |
|---|---|
| API-1 | Every call goes through the client configured in the SDK; `fetch`/`axios` at the call site is forbidden. |
| API-2 | `src/api/` and `src/~sdk/` are mandatory; `~sdk/` is regenerated and never edited. |
| API-6 | Pending, refreshing and real-time are distinct states. |
| API-7 | Pagination preserves the discriminated mode returned by the contract. |

## Governed by the constitution

These laws live in `tury-stack-architecture-pattern` and are not restated here.
What follows in this section is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-CTR-1` | The contract is the single source; never redeclared. | types and schemas from `~sdk` |
| `ARC-CTR-4` | A generated artifact is immutable. | `~sdk/` regenerated, never edited |
| `ARC-CTR-5` | A missing symbol is a contract blocker, not a license to work around it. | `api:generate` → typecheck → only then consume |
| `ARC-CTR-7` | Contract data stays derived; never copied into state of its own. | `query.data` read directly; no `useState`/context/store mirroring the server |
| `ARC-CON-9` | A replica is derived; the write declares what it invalidates. | the mutation invalidates the resource's generated query keys |
| `ARC-CON-10` | An optimistic write declares snapshot, rollback and reconciliation. | `setQueryData` only with the three steps written out |


## Flow

```text
backend OpenAPI
  → Kubb
  → types + Zod schemas + query/mutation hooks
  → api/http-client
  → business component
```

`api/http-client.ts` concentrates base URL, authentication, parsing of the
single error contract and network behavior. `api/query-client.ts` holds the
cache instance and its defaults. The generator references that client; no
feature creates a second transport.

These files are app-local infrastructure, not a feature. Every project receives
them. The audience, when one exists, chooses the OpenAPI surface consumed; with
no specific audience, the project uses the `auth` audience. The default runtime
base is the backend origin, `http://localhost:3000`.

## State

- Server state → generated hook/TanStack Query.
- Route params/search → router.
- Form input → React Hook Form.
- Ephemeral presentation state → local state or Turystack hook.
- Session/permissions → `auth/` module.

Never sync a query with local state through `useEffect`.

## Loading, refresh and real-time

```typescript
const query = useListOrders(search)

if (query.isPending) {
  return <OrderTableSkeleton />
}

return (
  <LoadingOverlay visible={query.isFetching}>
    <OrderTable orders={query.data.data} />
  </LoadingOverlay>
)
```

- First load uses a skeleton.
- A refetch that preserves content uses an overlay.
- A list whose pagination fully replaces the layout may go back to the skeleton.
- A table preserves header/structure and uses an overlay on the changed region.
- A real-time event updates/invalidates the cache with no visible loading state.

## Mutations

After success, invalidate the resource's generated keys — that is `ARC-CON-9` in
this stack: the write says which derived reads just went stale.

An optimistic `setQueryData` is allowed only with the three steps of
`ARC-CON-10` written out: snapshot before, reconciliation with the response,
restoration **and** a visible error on failure. Without all three, invalidate and
wait. The mutation's pending state belongs to the control that fired the action.

## Pagination

Do not convert a discriminated response into a shape of your own. `mode: 'page'`
uses page/total; `mode: 'cursor'` uses cursor/hasMore. The route search and the
toolbar preserve the same mode.

## Verification

1. run `api:generate`;
2. confirm the generated symbol;
3. run typecheck;
4. only then implement the consumer.

Concrete Kubb and plugin configuration belongs to the CLI template and to the
tooling documentation, not to this skill.

## Never do

- Editing Kubb output.
- Hand-writing a `User` type or a request schema when it already exists in the
  SDK.
- Using a temporary `fetch` because the endpoint has not shown up.
- Copying `query.data` into local state.
- Treating a pending `undefined` as an empty list.
- Showing loading on a real-time update.
