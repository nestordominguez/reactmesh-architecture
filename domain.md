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
    └── mutators.ts
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

Factories for structured data objects used in view, store, or facade. Does not validate — that's the model's job.

### ✅ Example

```ts
// src/features/office/domain/struct/builders.ts

export const buildOffice = (officeData: Partial<Office>): Office => ({
  address: officeData.address ?? '',
  floor: officeData.floor ?? '',
  shifts: officeData.shifts ?? [],
});
```

---

## 🔟 `*Facade.ts`

A thin orchestrator that wires together `model/` and `struct/` calls. It does **not** contain logic itself. Returns a `FacadeResult<T, E>` so the hook can act on success or failure without knowing the internals.

The facade is **internal to the feature** — only the feature's own hooks may import from it. Other features must go through the portal.

### ✅ Example

```ts
// src/features/shift/domain/shiftFacade.ts

type FieldErrors = Partial<Record<keyof Shift, string>>;

const validate = (shift: Shift): FieldErrors => {
  const errors: FieldErrors = {};
  const startError = validateStartTime(shift.startTime);
  if (startError) errors.startTime = startError;
  const endError = validateEndTime(shift.startTime, shift.endTime);
  if (endError) errors.endTime = endError;
  return errors;
};

const isValid = (shift: Shift): boolean => Object.keys(validate(shift)).length === 0;

const submit = (shift: Shift): FacadeResult<Shift, FieldErrors> => {
  const errors = validate(shift);
  if (Object.keys(errors).length > 0) return { success: false, error: errors };
  return { success: true, data: buildShiftPayload(shift) };
};

export const shiftFacade = { validate, isValid, submit };
```

---

## 🚪 `*Portal.ts`

The portal is the **curated public API** of a feature's domain. It is the only file other features are allowed to import from this layer.

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

### How portals are consumed — dependency injection

Portal functions must be consumed via **dependency injection**, not imported directly in hooks. The **view component** is the injection point: it imports from the portal and passes the functions as props to its hooks.

The container is not the injection point for portal functions. Its only responsibility is Redux wiring (state + actions). `mapDispatchToProps` is reserved for Redux action creators — putting non-action logic there violates its semantic purpose.

```ts
// ❌ Direct import in a hook — creates coupling
import { ShiftPortal } from '@/features/shift/domain/shiftPortal';

const useAppointmentForm = () => {
  const valid = ShiftPortal.isValid(shift); // hook is coupled to the portal
};

// ❌ Portal import in the container — wrong layer
// appointmentFormContainer.ts
const mapDispatchToProps = () => ({
  isShiftValid: ShiftPortal.isValid, // mapDispatchToProps is for Redux actions, not domain logic
});

// ✅ DI — hook receives the function as a parameter
type Props = {
  isShiftValid: (shift: Shift) => boolean;
};

const useAppointmentForm = ({ isShiftValid }: Props) => {
  const valid = isShiftValid(shift); // hook has no dependency on any feature
};

// ✅ The view imports the portal and injects it into the hook
// appointmentForm.tsx
import { ShiftPortal } from '@/features/shift/domain/shiftPortal';

const AppointmentForm: React.FC<PropsFromRedux> = ({ ... }) => {
  const { ... } = useAppointmentForm({
    isShiftValid: ShiftPortal.isValid,
  });
  ...
};
```

### ❌ Don't

- Import from `*Facade.ts` in another feature — always go through the portal
- Import from a portal directly inside a hook — inject via the view component
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
- Access `store/`, `serializer/`, or anything async
- Build rendering data or format for display
- Import from `struct/` or `*Model` in hooks — hooks only import from the facade
- Import from another feature's `*Facade.ts` — always go through its `*Portal.ts`

---

## 🔗 Related

- [🖼️ `view/`](./view.md) — Uses domain via hooks or containers
- [⚙️ `hooks/`](./hooks.md) — Calls facade to perform operations
- [📦 `store/`](./store.md) — Uses domain types; never imports model or struct directly
