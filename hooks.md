# 🪝 `hooks/`

The `hooks/` directory contains custom hooks that expose UI behavior to the `view/` components.

These hooks do not render anything themselves — they encapsulate logic like user input handling, local state, or coordination between other layers such as `domain/`, `store/`, and `presentation/`.

---

## 🧭 Responsibilities

- Expose **handlers and state** to the view
- Orchestrate calls to `domain/`, `store/`, or `presentation/`
- Stay **stateless** beyond temporary UI state
- Improve **testability** and **reuse** across features or components

---

## 🔑 Core Rule 1: BE Calls Live in `store/`

All backend communication — URL strings, HTTP verbs, fetch invocation, websocket setup, GraphQL clients, any I/O that crosses the FE/BE boundary — is **defined in or invoked from `store/`**.

Hooks may **trigger** a backend interaction (dispatching an action, calling `useQuery`/`useMutation`, invoking a Zustand async action), but they cannot **contain** the URL, the verb, or the fetch logic.

### Why

Backend contracts are the most volatile boundary in any frontend. The BE team renames an endpoint, changes a payload shape, adds an auth header — and those changes happen on a schedule that has nothing to do with UI work. If the URL lives inside a hook, every BE rename forces you to grep across `hooks/` and `view/`. If the URL lives in `store/`, you change one place and the rest of the codebase doesn't notice.

This rule is **not about Redux**. It applies the same way regardless of state library:

| Library         | Where the BE call is defined                                       | What the hook does                                  |
|-----------------|--------------------------------------------------------------------|-----------------------------------------------------|
| Redux Saga      | `store/<feature>Saga.ts` (`call(apiClient, url, payload)`)         | `dispatch(xStart(...))`                             |
| RTK Query       | `store/<feature>Api.ts` (`createApi({ endpoints: ... })`)          | `useGetXQuery(...)` / `useCreateXMutation(...)`     |
| TanStack Query  | `store/queries/<feature>.ts` (exported `queryFn`)                  | `useQuery({ queryKey, queryFn })`                   |
| Zustand         | `store/<feature>Store.ts` (async action inside the store)          | `useXStore(s => s.fetchX)` then invoke              |
| Plain thunk     | `store/<feature>Thunks.ts`                                         | `dispatch(fetchX())`                                |

In every case, **the definition of the call lives in `store/`**. The hook only consumes the abstraction.

### ✅ Allowed in a hook

```ts
// Saga / Redux
const dispatch = useDispatch();
useEffect(() => { dispatch(loadPatientsStart()); }, [dispatch]);

// RTK Query
const { data, isLoading } = useGetPatientsQuery({ doctorId });

// TanStack Query — queryFn imported from store/
import { fetchPatients } from '@/features/patient/store/queries/fetchPatients';
const { data } = useQuery({ queryKey: ['patients'], queryFn: fetchPatients });

// Zustand
const fetchPatients = usePatientsStore((s) => s.fetchPatients);
useEffect(() => { fetchPatients(); }, [fetchPatients]);
```

### ❌ Forbidden in a hook

```ts
// ❌ URL string in the hook
useEffect(() => { fetch('/api/v1/patients').then(...) }, []);

// ❌ axios call in the hook
const response = await axios.get('/patients');

// ❌ apiClient invocation in the hook
const apiClient = ApiClient.getInstance(Verb.get);
const response = await apiClient(PathManager.doctor.patients.list());
```

### Verification

Grep `fetch\|axios\|apiClient\.` inside `features/**/hooks/` and `features/**/view/` should return zero matches. The only legitimate I/O calls in those directories are calls to abstractions that themselves live in `store/`.

> **Exception — PathManager.** `src/api/pathManager.ts` constructs URLs (it's a helper); it does not invoke the BE. PathManager lives outside `store/` because it mirrors the BE contract — it's plumbing for the FE/BE boundary. See [`store.md`](./store.md) for the full rationale.

---

## 🔑 Core Rule 2: Hooks Don't Hide Domain Logic

Hooks may access the store. What they cannot do is **contain domain logic** — validation, transformation, decision rules, or selection logic that belongs in the feature's brain.

Domain logic lives in `domain/` (specifically in `*Facade.ts`, `*Model.ts`, or `struct/*`). The hook orchestrates and reacts; it never decides.

### ✅ Allowed in a hook

```ts
// ✅ Read state, dispatch on event
export const useAssignAppointment = () => {
  const dispatch = useDispatch();
  const patient = useSelector((state: RootState) => state.patient.selected);

  const handleAssign = (shift: Shift) => {
    const result = appointmentFacade.submit({ patient, shift });
    if (!result.success) return;
    dispatch(createAppointmentStart(result.data));
  };

  return { patient, handleAssign };
};
```

The hook reads state, the facade decides validity, the hook dispatches. Each layer does its job.

### ❌ Forbidden in a hook

```ts
// ❌ Validation inline — belongs in facade/model
useEffect(() => {
  if (data.age >= 18 && data.consent === true && !data.blacklisted) {
    dispatch(approveStart(data));
  }
}, [data]);

// ❌ Complex selector inline — belongs in store/selectors or struct/selectors
const activeAdults = useSelector((state) =>
  state.patients.list
    .filter((p) => p.status === 'active' && p.age >= 18)
    .sort((a, b) => a.lastName.localeCompare(b.lastName))
);

// ❌ Payload transformation inline — belongs in facade or struct/mutators
dispatch(saveStart({
  ...form,
  fullName: `${form.firstName} ${form.lastName}`.trim(),
  normalizedPhone: form.phone.replace(/\D/g, ''),
}));
```

In all three cases the hook is **doing the feature's thinking**. Move the thinking to `domain/` and let the hook orchestrate.

---

## 🔑 One Responsibility Per Hook

Never mix concerns in one hook. A hook that manages form state must not also sync wizard steps and dispatch save actions.

```
use<Feature>Form    → local state + facade calls only
use<Feature>Sync    → only syncs derived state (e.g. wizard steps, route params)
use<Feature>Action  → only triggers a dispatch / mutation
```

Each hook is independently testable. A form hook needs no QueryClient mock; an action hook needs no form-state setup. Splitting by responsibility is what makes hooks reusable across features and trivial to reason about.

---

## ✅ Example: `useAssignAppointmentForm`

```ts
import { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { appointmentFacade } from '../domain/appointmentFacade';
import { createAppointmentStart, updateAppointmentStart } from '../store/appointmentSlice';

export const useAssignAppointmentForm = (appointmentId: Maybe<string>) => {
  const dispatch = useDispatch();
  const patient = useSelector((state: RootState) => state.patient.selected);
  const [form, setForm] = useState<NewAppointment>(defaultForm);

  const handleConfirm = () => {
    const result = appointmentFacade.submit(form);
    if (!result.success) return;
    if (appointmentId) dispatch(updateAppointmentStart({ id: appointmentId, ...result.data }));
    else dispatch(createAppointmentStart(result.data));
  };

  return { patient, form, setForm, handleConfirm };
};
```

The hook accesses the store directly (Rule 1 satisfied — the BE call is in the saga, the hook only triggers it). The validation and payload shaping live in `appointmentFacade.submit` (Rule 2 satisfied — no domain logic inline).

---

## 🚫 Don't

- Define URLs, HTTP verbs, or fetch calls in a hook (violates Rule 1)
- Validate, transform, or decide inline (violates Rule 2 — move to `domain/facade`)
- Mix form state, sync, and dispatch in the same hook (violates SRP)
- Perform rendering or return JSX
- Re-derive types from another hook with `Parameters<typeof hook>[n]` — use named types from the facade

---

## 🔗 Related

- [🖼️ `view/`](./view.md) — Where hooks are consumed
- [🏗️ `architecture.md`](./architecture.md) — Overview of layer responsibilities
- [📦 `store/`](./store.md) — Where BE calls actually live (and why PathManager is the documented exception)
- [🧱 `domain/`](./domain.md) — Where validation and business logic should live
