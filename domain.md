# 🧱 `domain/`

The `domain/` layer is the brain of your application. It encapsulates **business logic**, **validations**, and **structuring rules**, ensuring all decisions and operations follow well-defined rules.

This layer does **not** deal with rendering, async operations, or UI logic — it simply defines how data behaves and what's allowed.

---

## 🧭 Responsibilities

- Encapsulate business rules and validations
- Build domain-specific data structures via factories
- Delegate orchestration to `facade/`, keeping logic clean
- Avoid any dependency on UI, presentation, or async mechanisms

---

## 📂 Structure

```text
src/features/x/domain/
├── xDataTypes.ts         # Domain interfaces and types (camelCase, no logic)
├── xModel.ts             # Validation and domain rules
├── xFacade.ts            # Orchestrates calls to model and struct — internal to the feature
├── xPortal.ts            # Curated public API for other features to consume
└── struct/               # Factories to build structured data
    ├── builders.ts
    ├── mutators.ts
    └── selectors.ts
```

---

## 📑 `*DataTypes.ts`

Domain interfaces and types only — no logic. Always camelCase. snake_case never appears here; that belongs in serializer `Api*` types.

---

## 📑 `*Model.ts`

Responsible for:

- Validating input data
- Enforcing business rules
- Exposing validation status or derived domain decisions

> Note: If a value needs to be normalized, standardized, or built from defaults, that is a responsibility of `struct/` — not the model.

### ✅ Example

```ts
// src/features/shift/domain/shiftModel.ts

const PHONE_REGEX = /^\d+$/;

export const validateStartTime = (time: string): string | undefined => {
  if (!time.trim()) return 'El horario de inicio es requerido';
  return undefined;
};

export const validateEndTime = (start: string, end: string): string | undefined => {
  if (!end.trim()) return 'El horario de fin es requerido';
  if (end <= start) return 'El horario de fin debe ser posterior al inicio';
  return undefined;
};
```

---

## 🔟 `struct/`

Pure data-structure helpers used by the facade. `struct/` does not validate — that's the model's job — and it does not format for display.

`struct/` has three roles:

- `builders.ts` — creates new structures from defaults or partial input.
- `mutators.ts` — transforms or reshapes existing structures into another structure or payload.
- `selectors.ts` — derives or extracts values from existing structures without changing their shape.

### ✅ Example

```ts
// src/features/office/domain/struct/builders.ts

export const buildOffice = (officeData: Partial<Office>): Office => ({
  address: officeData.address ?? '',
  floor: officeData.floor ?? '',
  shifts: officeData.shifts ?? [],
});
```

### ✅ Selector example

```ts
// src/features/medicalRecordEntry/domain/struct/selectors.ts

type OpenEntryCandidate = {
  id: number;
  status: 'open' | 'closed';
};

export const findOpenEntry = <T extends OpenEntryCandidate>(entries: T[]): Maybe<T> =>
  entries.find((entry) => entry.status === 'open') ?? null;
```

---

## 🔟 `*Facade.ts`

A thin orchestrator that wires together `model/` and `struct/` calls. It does **not** contain logic itself — specifically:

- **No validation logic in the facade.** Validation lives in the model. The facade may re-export model functions so hooks have a single import surface, but it must not WRITE validation rules.
- **No methods with action verb names.** The facade does not perform actions; it composes. Names like `submit`, `save`, `delete`, `send` are misleading because the actual side-effect lives in the saga / hook. Use past-participles or descriptive nouns (`validated`, `payload`, `prepared`) for composition methods that return a `FacadeResult`.

Returns a `FacadeResult<T, E>` so the hook can act on success or failure without knowing the internals.

The facade is **internal to the feature** — only the feature's own hooks may import from it. Other features must go through the portal.

### ✅ Example

```ts
// src/features/shift/domain/shiftModel.ts

// Validation — atomic + aggregate + derived queries — ALL in the model.
export const validateStartTime = (time: string): string | undefined => { ... };
export const validateEndTime = (start: string, end: string): string | undefined => { ... };

export const validate = (shift: Shift): FieldErrors => {
  const errors: FieldErrors = {};
  const startError = validateStartTime(shift.startTime);
  if (startError) errors.startTime = startError;
  const endError = validateEndTime(shift.startTime, shift.endTime);
  if (endError) errors.endTime = endError;
  return errors;
};

export const isValid = (shift: Shift): boolean => Object.keys(validate(shift)).length === 0;
```

```ts
// src/features/shift/domain/shiftFacade.ts

import { isValid as modelIsValid, validate as modelValidate } from './shiftModel';
import { buildShiftPayload } from './struct/mutators';

type FieldErrors = Partial<Record<keyof Shift, string>>;

// Composition only — no validation logic, no action-named methods.
// `validated` is a past-participle: the facade is NOT performing the submit,
// it returns the validated payload (or errors) for the hook to act on.
const validated = (shift: Shift): FacadeResult<Shift, FieldErrors> => {
  const errors = modelValidate(shift);
  if (Object.keys(errors).length > 0) return { success: false, error: errors };
  return { success: true, data: buildShiftPayload(shift) };
};

export const shiftFacade = {
  validate: modelValidate, // re-export from model — single import surface for hooks
  isValid: modelIsValid,
  validated,
};
```

### ❌ Anti-pattern (do not copy)

```ts
// src/features/shift/domain/shiftFacade.ts — WRONG

// validation aggregator inside the facade
const validate = (shift) => { ... aggregate atomic rules ... };

// action-named method that does not perform the action
const submit = (shift) => { ... validate + build ... };
```

The aggregator and the query belong in the model. The action name lies: the function doesn't submit, it just returns data for the hook (which then dispatches to the saga, which then performs the actual side-effect).

---

## 🚪 `*Portal.ts`

> **The portal is optional.** Like every other layer in ReactMesh, it activates only when needed. A feature without cross-feature consumers does not have a Portal — the Facade is enough because nothing outside the feature imports it. The Portal appears the moment another feature needs to import from this one; only then is it decided what to expose. Never write an empty Portal "just in case."

The portal is the **curated public API** of a feature's domain. When it exists, it is the only file other features are allowed to import from this layer.

The portal exposes a deliberate subset of facade functions — the ones that make sense to be public. Having something in the facade does not automatically mean it belongs in the portal. Conversely, in rare cases a portal may expose something not present in the facade if it serves a specific cross-feature contract.

### Why portal instead of exposing the facade directly

The facade may contain internal methods not intended for outside use. The portal makes the public contract explicit and intentional, rather than accidentally exposing everything.

### ✅ Portal definition example

```ts
// src/features/shift/domain/shiftPortal.ts
import { shiftFacade } from './shiftFacade';

// Only expose what other features should consume
export const ShiftPortal = {
  isValid: shiftFacade.isValid,
  toPayload: shiftFacade.toPayload,
};
```

### How portals are consumed

Hooks and view components are the same UI layer — both can import from a portal directly. The portal is the public entry point for cross-feature domain logic; importing it in a hook or a view component is equally valid, regardless of the store library you use.

```ts
// ✅ Direct import in a hook (works with Redux, RTK Query, Zustand, anything)
import { ShiftPortal } from '@/features/shift/domain/shiftPortal';

const useAppointmentForm = () => {
  const valid = ShiftPortal.isValid(shift);
};

// ✅ Direct import in a view component
import { ShiftPortal } from '@/features/shift/domain/shiftPortal';

const AppointmentForm = () => {
  const isValid = ShiftPortal.isValid(shift);
  // ...
};
```

#### When using the Redux + Container pattern (Option B in `view.md`)

If you keep a Redux Container (`connect()` + `mapStateToProps`/`mapDispatchToProps`), the Container is **not** the right place for portal imports — `mapDispatchToProps` is reserved for Redux action creators. Cross-feature domain logic goes through the hook or directly in the view, not through the Redux wiring layer.

```ts
// ❌ Portal import in the container — wrong layer
const mapDispatchToProps = () => ({
  isShiftValid: ShiftPortal.isValid, // mapDispatchToProps is for Redux actions, not domain logic
});
```

### ❌ Don't

- Import from `*Facade.ts` in another feature — always go through the portal
- Import portal functions in containers — `mapDispatchToProps` is for Redux actions only
- Expose internal helpers (field-level validators, error mutators) in the portal unless explicitly needed cross-feature

---

## 🔑 New / Base Type Pattern

Every entity that gets persisted follows this two-type pattern:

```ts
type NewShift = { /* all fields except id */ };   // not yet saved — no id
type Shift    = NewShift & { id: Id };            // persisted — id guaranteed
```

**When to use each:**
- `NewShift` — when creating. The absence of `id` is enforced by the type.
- `Shift` — when the entity is known to come from the DB (API response, Redux state after a successful save).

**Why two types, not three:**
A third type (`Shift` with `id?: Id`) was the original "unknown state" base. It was eliminated because in practice that state never exists: you are either building a new entity (no id) or working with one from the DB (id guaranteed). The optional-id variant only exists when the type system is not being used precisely. Having three types also means every reader must decide which variant applies — the two-type pattern removes that decision.

**Why this matters — and why consistency is the real value:**
The primary benefit is not TypeScript catching a bug in one specific place. It is the elimination of mental fatigue across the entire codebase. When every persisted entity follows this pattern without exception, a developer opening any file never has to ask "does this `id` exist here or not?" — the type already answers the question. That deduction is small on its own, but accumulated over hours of work it drains focus and reduces the capacity to solve real problems.

Apply this pattern to **all persisted entities, without exceptions**. The value of the pattern comes from its universality, not from its per-case optimality.

**Intersection works without `Omit`:** `Shift = NewShift & { id: Id }` correctly narrows `id` to required because the intersection of `Id | undefined` and `Id` is `Id`. `Omit` is only needed when the new type is incompatible with the base (e.g., changing `id` from `string` to `number`).

**Note:** This pattern applies to domain types, not serializer `Api*` types. In the serializer, `id?: Id` is sufficient since the type lives in one file and is used in two functions.

---

## ❌ Don't

- Put UI logic, hooks, or presentation helpers in `domain/`
- Access `store/` (which now includes the serializer) or anything async
- Build rendering data or format for display
- Import from `struct/` or `*Model` in hooks — hooks only import from the facade
- Import from another feature's `*Facade.ts` — always go through its `*Portal.ts`

---

## 🔗 Related

- [🖼️ `view/`](./view.md) — Uses domain via hooks or containers
- [⚙️ `hooks/`](./hooks.md) — Calls facade to perform operations
- [📦 `store/`](./store.md) — Uses domain types; never imports model or struct directly
