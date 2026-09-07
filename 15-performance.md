# Performance

**Concept.** Every other section decides what the user sees. This one decides
**what they wait for**. Performance is not a pass at the end: three of the four
decisions below are made when the route, the list and the image are written, and
are expensive to reverse afterwards.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> TanStack · Vite. The split, `XXX-n` versus `XXX-Ln`, and why an `ARC-…` law
> is cited and never restated: `turystack-frontend-pattern` › *How a section
> is written*.

---

**Rules defined here:** `PRF-1` · `PRF-2` · `PRF-3` · `PRF-4` · `PRF-5` ·
`PRF-L1` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

## 🌐 Generic pattern (portable — stack-independent)

**PRF-1 — a route pays for itself, not for its neighbours.**

Code reaches the user per route, so a screen nobody opened costs nothing to open
the app. The default is one bundle per route; the exceptions are what the shell
genuinely needs on every screen. Without this, every feature added anywhere
slows down the login page, and the cost is invisible because it is spread across
everyone. **[PRF-1]**

**PRF-2 — a collection with no ceiling in the contract is windowed.**

If the API can return ten thousand rows, the surface renders a window of them,
not all of them. "It is fine today" is a statement about the seed data, not
about the contract. The decision belongs with the surface, because retrofitting
windowing means rewriting how selection, keyboard and sticky headers work.
**[PRF-2]**

**PRF-3 — media is sized at the source.**

Dimensions, format and compression are decided where the file is produced or
served, never by rendering a large image into a small box. Scaling in the layout
costs the full download, the full decode and the memory of the original, and it
happens on the device least able to pay for it. **[PRF-3]**

**PRF-4 — a budget is a number that fails a build.**

"Keep it fast" changes nothing. A budget — bundle size for the entry, an
interaction latency target for the surface — is stated, checked mechanically,
and breaking it is a red gate, not a comment in review. A budget nobody enforces
is documentation of an intention. **[PRF-4]**

**PRF-5 — the cache is configured, not worked around.**

Refetching to force a re-render, or copying server data locally to avoid a
refetch, are the same mistake from opposite ends: both are answers to staleness
that should have been a cache setting. Staleness is a decision per resource,
made once, alongside the invalidation the write already declares (`ARC-CON-9`).
**[PRF-5]**

**PRF-L1 — memoization answers a measurement, not a suspicion.**

Wrapping components and callbacks by reflex adds allocation and comparison cost
on every render, hides the real cause, and makes dependency lists a maintenance
surface of their own. Measure, find the render that hurts, then memoize that
one. **[PRF-L1]**

### Invariants (the law the gates enforce)

| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| PRF-1 | Routes are code-split by default; only genuine shell dependencies load eagerly | constitutional | `gate:route-splitting` | Splitting / ❌ |
| PRF-2 | A collection unbounded by the contract is windowed, decided when the surface is written | constitutional | `manual` | Lists / ❌ |
| PRF-3 | Media dimensions and format are decided at the source, never by layout scaling | constitutional | `grit:no-layout-scaled-media` | Media / ❌ |
| PRF-4 | Budgets are numbers checked by a gate; breaking one is red | constitutional | `gate:bundle-budget` | Budgets |
| PRF-5 | Staleness is a cache decision per resource; never a manual refetch or a local copy | constitutional | `grit:no-server-state-copy` | Cache / ❌ |
| PRF-L1 | Memoization follows a measurement; it is not the default wrapper | stack lint | `manual` | ❌ |

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-CTR-7` | Contract data stays derived, never copied into state of its own. | the reason `PRF-5` forbids the local mirror |
| `ARC-CON-9` | A replica is derived; the write declares what it invalidates. | `staleTime` and invalidation are the same decision, made together |
| `ARC-ERR-8` | A remote read has five outcomes, all decided. | splitting adds a pending state to the route itself — it is decided, not blank |
| `ARC-SEC-7` | Secrets and PII stay out of the bundle. | a budget review reads what is in the bundle, which is also a security read |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · Vite)

### Splitting

The router already supports splitting per route file; the work is to use it and
to keep the shell honest. What loads eagerly is the app shell, the session and
the design tokens — nothing that belongs to one screen.

```text
eager      router, providers, session, theme, error boundary
lazy       every route's component, its heavy dependencies, every modal
           that is not open on first paint
```

A chart library imported at module scope by one screen is in the entry bundle of
every screen. The gate catches it as a budget break (`PRF-4`), not as a review
comment.

### Lists

`06-data-surfaces.md` owns how a table behaves; this section owns when it must
be windowed. The trigger is the **contract**, not the current data: a list
endpoint that accepts `cursor` and has no server-side ceiling is unbounded, so
its surface is windowed from the first commit.

### Media

Decide dimensions and format where the asset is produced or served. In the app,
the rendered box declares the size it needs so the browser can reserve it — an
image that arrives without reserved space causes layout shift, which is a
correctness problem for the user's click, not a cosmetic one.

### Cache

`staleTime` and `gcTime` per resource are the whole of `PRF-5` in this stack:

```text
reference data that rarely moves   long staleTime — the refetch was the waste
a list the user is editing         short staleTime + invalidation on write
a real-time backed resource        the socket writes the cache; no polling
```

### Budgets

Two numbers, both in CI (`PRF-4`): the entry bundle's size ceiling, and the
number that matters for the surface being changed. Record them where the gate
reads them, not in a document — a budget outside the gate is a wish.

### ❌ Never do

```tsx
// ❌ [PRF-1] a route's heavy dependency imported at module scope in the shell
import { HeavyChart } from 'some-charts' // now in every screen's entry bundle

// ❌ [PRF-2] rendering an unbounded collection because today it is small
{orders.map((order) => <OrderRow key={order.orderId} order={order} />)}

// ❌ [PRF-3] a full-size asset scaled by the layout
<img src={originalPhoto} className="h-10 w-10" />

// ❌ [PRF-5] refetching to force a re-render
useEffect(() => { query.refetch() }, [somethingUnrelated])

// ❌ [PRF-5, ARC-CTR-7] copying server data locally to avoid a refetch
const [orders, setOrders] = useState(query.data)

// ❌ [PRF-L1] memoizing by reflex, with no measured cost
const value = useMemo(() => ({ id }), [id]) // passed to a component that renders once
```
