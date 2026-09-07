# API Contract & Server State

**Concept.** Every frontend consumes an API; OpenAPI generates its typed door.
The SDK's types, schemas and hooks are canonical; TanStack Query is the only
owner of server state.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> TanStack Query · Kubb. The split, `XXX-n` versus `XXX-Ln`, and why an
> `ARC-…` law is cited and never restated: `turystack-frontend-pattern` › *How
> a section is written*.

---

**Rules defined here:** `API-1` · `API-2` · `API-6` · `API-7` · `API-8` — the
law is the *Invariants* table below; every ❌ item cites the id it violates.

**Retired ids:** `API-3` · `API-4` · `API-5` — retired, not renumbered. A
review or commit citing one points at a rule that no longer exists; the number
is never reused.

## 🌐 Generic pattern (portable — stack-independent)

**API-1 — one door to the network.**

Every request leaves through the single configured client. Base URL, auth
header, error-contract parsing and network behavior are decided once, so a
change to any of them lands everywhere at the same time. A `fetch` at the call
site is a second client with none of those decisions. **[API-1]**

**API-2 — the contract has an owner, and it is not this repository.**

The generated surface and its local infrastructure are mandatory and
regenerated; the app configures transport and cache around them and edits
neither. A hand-edited generated file survives exactly until the next
generation. **[API-2]**

**API-6 — pending, refreshing and real-time are three different states.**

First load has no content to preserve; a refresh has content that must not
flicker; a real-time update changes data nobody asked to reload. Collapsing them
into one boolean produces either a spinner over readable content or a silent
swap the user never notices. **[API-6]**

**API-8 — a live update enters through the cache, and a gap is assumed.**

A real-time channel is a second source of the same data, so it has one owner —
the module that already owns transport — and it lands in the **same cache** the
fetch writes to. A parallel store means two answers to one question, and the
screen shows whichever rendered last.

A connection also drops, and while it was down the client missed events it will
never be told about. So reconnection is not "resume": the client refetches what
it is showing, because the only honest assumption after a gap is that it no
longer knows. **[API-8]**

**API-7 — pagination keeps the mode the contract returned.**

A paginated response declares how it is paginated. Converting page/total into
cursor/hasMore (or the reverse) in the client invents a shape the server never
promised, and the next page is requested with parameters the endpoint does not
accept. **[API-7]**

### Invariants (the law the gates enforce)

| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| API-1 | Every call goes through the configured SDK client; `fetch`/`axios` at the call site is forbidden | constitutional | `gate:no-fetch-outside-client` | Flow / ❌ |
| API-2 | `src/api/` and `src/~sdk/` are mandatory; `~sdk/` is regenerated and never edited | constitutional | `gate:generated-untouched` | Flow / ❌ |
| API-6 | Pending, refreshing and real-time are distinct states | constitutional | `manual` | Loading, refresh and real-time / ❌ |
| API-7 | Pagination preserves the discriminated mode returned by the contract | constitutional | `manual` | Pagination / ❌ |
| API-8 | The live channel has one owner, writes into the same cache as the fetch, and refetches after a reconnect | constitutional | `gate:realtime-owner` | Real-time / ❌ |

Ids are stable across versions; a gap is a law that moved to the constitution.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-CTR-1` | The contract is the single source; never redeclared. | types and schemas from `~sdk` |
| `ARC-CTR-4` | A generated artifact is immutable. | `~sdk/` regenerated, never edited |
| `ARC-CTR-5` | A missing symbol is a contract blocker, not a license to work around it. | `api:generate` → typecheck → only then consume |
| `ARC-CTR-7` | Contract data stays derived; never copied into state of its own. | `query.data` read directly; no `useState`/context/store mirroring the server |
| `ARC-CON-9` | A replica is derived; the write declares what it invalidates. | the mutation invalidates the resource's generated query keys |
| `ARC-CON-10` | An optimistic write declares snapshot, rollback and reconciliation. | `setQueryData` only with the three steps written out |
| `ARC-ERR-8` | A remote read has five outcomes, all decided. | pending/empty/partial/error/success — the branch law lives in `07-ui-states-and-feedback.md` |

---

## 🛠️ Project-specific (TypeScript · React · TanStack Query · Kubb)

### Flow

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

### State

- Server state → generated hook/TanStack Query.
- Route params/search → router.
- Form input → React Hook Form.
- Ephemeral presentation state → local state or Turystack hook.
- Session/permissions → `auth/` module.

Never sync a query with local state through `useEffect`.

### Loading, refresh and real-time

```typescript
const query = useListOrders(search)
const outcome = useDataOutcome({
  query,
  select: (page) => page.data,
})

return <OrderTable outcome={outcome} />
```

The three states are still three — they are decided by `useDataOutcome` and
painted by the surface, in the order `UST-10` fixes, instead of by an `if` per
screen. Skeleton on the first load, overlay over rows that are still valid, and
nothing at all for a live update, are the primitive's answers to `API-6` and
`UST-2`. The full law is in 07-ui-states-and-feedback.md.

- First load uses a skeleton.
- A refetch that preserves content uses an overlay.
- A list whose pagination fully replaces the layout may go back to the skeleton.
- A table preserves header/structure and uses an overlay on the changed region.
- A real-time event updates/invalidates the cache with no visible loading state.

### Mutations

After success, invalidate the resource's generated keys — that is `ARC-CON-9` in
this stack: the write says which derived reads just went stale.

An optimistic `setQueryData` is allowed only with the three steps of
`ARC-CON-10` written out: snapshot before, reconciliation with the response,
restoration **and** a visible error on failure. Without all three, invalidate and
wait. The mutation's pending state belongs to the control that fired the action.

### Real-time

`api/` owns the connection, the same way it owns the HTTP client — one instance,
configured once, never a socket opened inside a feature (`API-8`). An arriving
event does exactly one of two things, decided per resource:

| Event carries | Do | Because |
|---|---|---|
| the full new state | write it into the cache | the payload already is the answer (`ARC-IDM-5`) |
| only that something changed | invalidate the resource's keys | the server still owns the shape (`ARC-CON-9`) |

Never both, and never a `useState` alongside the cache holding the "live"
version — that is `ARC-CTR-7` with a socket attached.

Three behaviours follow from the connection being unreliable:

- **On reconnect, refetch what is mounted.** Events that happened during the gap
  are gone; the cache is stale in a way no event will correct.
- **A live update shows no loading.** It replaces data the user is already
  reading, so a spinner would flash content that never left (`API-6`).
- **An update the user did not ask for does not steal focus or scroll.** A row
  changing under an open menu is a bug, not freshness.

### Pagination

Do not convert a discriminated response into a shape of your own. `mode: 'page'`
uses page/total; `mode: 'cursor'` uses cursor/hasMore. The route search and the
toolbar preserve the same mode.

### Verification

1. run `api:generate`;
2. confirm the generated symbol;
3. run typecheck;
4. only then implement the consumer.

Concrete Kubb and plugin configuration belongs to the CLI template and to the
tooling documentation, not to this skill.

### ❌ Never do

- `[ARC-CTR-4]` Editing Kubb output.
- `[ARC-CTR-1]` Hand-writing a `User` type or a request schema when it already exists in the SDK.
- `[ARC-CTR-5]` Using a temporary `fetch` because the endpoint has not shown up.
- `[API-1]` Calling `fetch`/`axios` from a component, hook or feature.
- `[ARC-CTR-7]` Copying `query.data` into local state, or syncing a query with `useEffect`.
- `[ARC-ERR-8, UST-11]` Treating a pending `undefined` as an empty list — `query.data ?? []` at a call site, instead of handing the surface the outcome.
- `[API-6]` Showing loading on a real-time update, or a skeleton over content the user was already reading.
- `[ARC-CON-9]` Finishing a mutation without invalidating the reads it just made stale.
- `[ARC-CON-10]` Writing optimistically without snapshot, reconciliation and a visible failure.
- `[API-7]` Rewriting a paginated response into a shape of your own.
- `[API-8]` Opening a socket inside a feature, or keeping live data in a store beside the cache.
- `[API-8]` Treating a reconnect as a resume, so a gap in events becomes a screen that is quietly wrong.
