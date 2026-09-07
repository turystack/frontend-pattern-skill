# File uploads

**Concept.** An upload is the one interaction where the client holds something
the server does not have yet, over a channel that fails halfway. That makes it
three problems wearing one control: **who is allowed to store the bytes**,
**what happens when the transfer dies mid-flight**, and **what the record
references while the file is still travelling**.

> **How to read this file.** 🌐 Generic pattern is the portable law; 🛠️
> Project-specific is that same law expressed as code in TypeScript · React ·
> TanStack · @turystack. The split, `XXX-n` versus `XXX-Ln`, and why an
> `ARC-…` law is cited and never restated: `turystack-frontend-pattern` › *How
> a section is written*.

---

**Rules defined here:** `UPL-1` · `UPL-2` · `UPL-3` · `UPL-4` · `UPL-5` ·
`UPL-6` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

## 🌐 Generic pattern (portable — stack-independent)

**UPL-1 — the client never holds the storage credential.**

The browser receives permission to perform **one** upload — scoped, expiring,
issued by the backend — and never a key that could write anything else. Anything
shipped to the client is public (`ARC-SEC-7`), so a storage credential in the
frontend is a storage credential in the hands of every user. **[UPL-1]**

**UPL-2 — the file gets an identity before the bytes move.**

The backend names the file first; the transfer then fills that name. Naming
afterwards means the moment between "bytes stored" and "record written" has no
owner, and whatever lands there when the tab closes is an orphan nobody will
ever find. **[UPL-2]**

**UPL-3 — the client validates for feedback, the backend validates for truth.**

Size and type are checked in the client so the user is told in the file picker
instead of after a two-minute upload. That check is a courtesy, not a control:
the backend re-decides, from the content rather than the extension or the
client-declared type, because the client was never the protection
(`ARC-SEC-2`). **[UPL-3]**

**UPL-4 — an upload has its own states, and they are not the form's.**

Selected, transferring with progress, failed-with-retry, cancelled, stored: five
outcomes of a remote operation (`ARC-ERR-8`), owned by the file control. A form
that shows only its own submitting state leaves the user watching a frozen
screen with no idea whether anything is moving, and no way to stop it.
**[UPL-4]**

**UPL-5 — the record commits after the bytes land.**

The write that references the file happens once the file exists. Writing first
produces rows pointing at nothing whenever the transfer fails — and those rows
render as broken images long after everyone forgot why. **[UPL-5]**

**UPL-6 — a stored file inherits a lifetime.**

Every uploaded file is a copy of something with a retention decision behind it
(`ARC-DAT-5`). An upload flow that never says how long the file lives, or what
erasing the parent record does to it, is a store that only grows. **[UPL-6]**

### Invariants (the law the gates enforce)

| ID | Law (one line) | Class | Gate | Detector (🛠️) |
|---|---|---|---|---|
| UPL-1 | The client receives a scoped, expiring permission for one upload — never a storage credential | constitutional | `gate:no-storage-credential` | Flow / ❌ |
| UPL-2 | The backend assigns the file's identity before the transfer starts | constitutional | `manual` | Flow / ❌ |
| UPL-3 | Client validation is feedback; the backend re-decides from content | constitutional | `manual` | Validation / ❌ |
| UPL-4 | The control owns selected / transferring / failed / cancelled / stored | constitutional | `test:upload-states` | States / ❌ |
| UPL-5 | The referencing write commits only after the bytes are stored | constitutional | `manual` | Flow / ❌ |
| UPL-6 | A stored file declares its lifetime and what erasing its parent does to it | constitutional | `manual` | Lifetime |

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.
What follows is how the Turystack frontend expresses them.

| ID | Law | How this stack expresses it |
|---|---|---|
| `ARC-SEC-2` | The backend is the authorization authority. | the client's size/type check is UX; the backend decides |
| `ARC-SEC-7` | Secrets never reach the bundle. | the reason `UPL-1` exists at all |
| `ARC-SEC-11` | A credential has a single owner. | the permission is issued per upload and expires |
| `ARC-ERR-8` | A remote read has five outcomes, all decided. | `UPL-4` is that law applied to a transfer |
| `ARC-ERR-9` | Unavailability is stated, never hidden. | over quota or wrong type disables the control **with the reason**, never removes it |
| `ARC-CON-1` | Every write declares its consistency strategy. | `UPL-5`: bytes first, then the record |
| `ARC-DAT-5` | A derived copy inherits the lifetime of its source. | `UPL-6` |

---

## 🛠️ Project-specific (TypeScript · React · TanStack · @turystack)

### Flow

The frontend never talks to the storage provider on its own terms. The backend
(`@turystack/nestjs-storage`) issues the permission and names the file; the
client transfers and then reports back:

```text
1. client asks the API to start an upload   name + declared size/type
2. API answers                              file identity + scoped, expiring permission
3. client transfers the bytes               progress, cancellable  (UPL-4)
4. client submits the form                  referencing the identity from step 2
5. API commits the record                   after confirming the object exists (UPL-5)
```

Step 2 is what keeps `UPL-1` and `UPL-2` true at the same time: the identity and
the permission are issued together, by the side that owns both.

### Validation

The generated contract already declares the accepted types and the size ceiling
(`ARC-CTR-1`); read them from there instead of re-typing constants into the
component. The client uses them to filter the picker and to explain a rejection
immediately. The backend uses the file's actual content.

### States

The control renders all five, and each has intentional UI — a spinner is not a
state (`ARC-ERR-8`):

| State | What the user gets |
|---|---|
| selected | file name, size, remove action |
| transferring | determinate progress **and** a cancel affordance |
| failed | the reason, plus retry — the selection is not lost |
| cancelled | back to selected; nothing was written |
| stored | the reference, and a way to replace it |

Progress belongs to the file, not to the form: two files upload at their own
pace, and the submit button waits for both.

### Lifetime

State it where the file is introduced (`UPL-6`): how long the object lives, and
whether erasing the parent record erases it. If the answer is "the file
survives", that is a decision someone made — write it down rather than
discovering it during an erasure request.

### ❌ Never do

```tsx
// ❌ [UPL-1] a storage credential in the client — public by construction
const client = new StorageClient({ accessKey: import.meta.env.VITE_S3_KEY })

// ❌ [UPL-5] writing the record before the bytes exist
await createDocument({ url: expectedUrl }) // the transfer may still fail
await upload(file)

// ❌ [UPL-3] trusting the client-declared type as the decision
if (file.type === 'application/pdf') { /* the backend must re-decide */ }

// ❌ [UPL-4] the form's submitting state standing in for the transfer's
<Button loading={form.formState.isSubmitting}>Send</Button> // no progress, no cancel

// ❌ [UPL-4, ARC-ERR-8] a failed transfer that discards the selection
onError: () => setFile(null)

// ❌ [ARC-ERR-9] hiding the control when the account is over quota
{withinQuota && <FileField … />} // it stays, inert, with the reason

// ❌ [ARC-CTR-1] re-typing the contract's limits into the component
const MAX_SIZE = 5 * 1024 * 1024 // already declared by the API schema
```
