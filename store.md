# 📦 `store/`

The `store/` layer handles **state management** and **async orchestration**. It is the bridge between the UI and backend services or other side-effects.

This layer is **store-agnostic** — Redux Toolkit + Saga, RTK Query, Zustand, Jotai, TanStack Query, or any combination per feature is acceptable. ReactMesh defines *where* I/O happens, not *which* library does it.

---

## 🧭 Responsibilities

- Centralize and manage feature state
- Orchestrate asynchronous flows (fetch, update, delete, websocket, polling)
- **Define and invoke all backend calls** (Rule 1 from [`hooks.md`](./hooks.md))
- Adapt request and response payloads via the feature's serializer
- Trigger side-effects in response to user actions or lifecycle events

---

## 📂 Structure

A feature with async needs a `store/` directory. The exact files depend on the chosen library; below are the common shapes.

### With Redux Toolkit + Saga

```text
src/features/x/store/
├── xSlice.ts             # Redux Toolkit slice (state + sync reducers + action creators)
├── xSaga.ts              # Async orchestration — API calls, side effects
├── xStoreType.ts         # State type (never inline)
├── xSerializer.ts        # API ↔ domain mapping (see "Serializer Lives Inside store/" below)
└── xSerializerType.ts    # Local Api* type + domain output type
```

### With RTK Query

```text
src/features/x/store/
├── xApi.ts               # createApi({ endpoints: { getX: builder.query(...), createX: builder.mutation(...) } })
├── xStoreType.ts         # Local UI state type (filters, selection — non-server state)
├── xSlice.ts             # Optional — only if the feature has client-only state alongside server state
└── xSerializer.ts        # API ↔ domain mapping, used inside transformResponse / transformErrorResponse
```

### With Zustand

```text
src/features/x/store/
├── xStore.ts             # create(set => ({ items: [], fetchItems: async () => { ... } }))
└── xSerializer.ts        # API ↔ domain mapping, called inside the async action
```

### Hybrid

A feature can mix: Zustand for local UI state, RTK Query for server state, Saga for legacy flows. The directory holds whatever the feature needs — **the rule is that all I/O definitions live here, not the choice of library**.

---

## 📦 Serializer Lives Inside `store/`

**Decision (2026-05):** Serializer files live inside `store/`, not as a sibling `serializer/` directory.

```
store/
  xSerializer.ts          # ✅ here
  xSerializerType.ts      # ✅ here
```

### Why

The serializer's only consumer is the saga (or RTK Query's `transformResponse`, or Zustand's async action) of the same feature — and ReactMesh's own rules already prohibit it from being imported anywhere else. A folder for code with a single co-located consumer is ceremony. Keeping the serializer next to the saga that uses it removes one mental decision ("is it in `serializer/` or `store/`?") and makes the feature self-contained: everything that touches the BE lives in one place.

### Migration callout

> **Decision landed; existing `features/*/serializer/` folders migrate in a separate task — both locations are valid until then.** Code following the old layout is not a violation; new features should use the new layout. When the migration task runs, all existing serializers move into `store/`.

### Serializer rules (unchanged)

- Always called from inside the orchestration code (saga, `transformResponse`, async action) before the data reaches the slice/store.
- snake_case → camelCase mapping is **manual, field by field**. No `humps`, no lodash `camelCase`, no automatic converters. Automatic conversion produces silent bugs when a field is renamed.
- Define a local `Api*` type that mirrors the raw response (snake_case). The output is the domain model (camelCase).
- Outgoing payloads (FE → API) are also mapped manually before invoking the API call.

### Example — Saga

```ts
// store/officeSaga.ts
import { officeFromBe, newOfficeToBe } from './officeSerializer';
import type { ApiOffice } from './officeSerializerType';

function* fetchOfficesSaga() {
  try {
    const apiClient = ApiClient.getInstance(Verb.get);
    const url = PathManager.doctor.offices.main();
    const response: AxiosResponse<ApiOffice[]> = yield call(apiClient, url);
    const offices = response.data.map(officeFromBe);
    yield put(setOffices(offices));
  } catch (error) {
    const apiError = handleApiError(error);
    yield put(setError(apiError));
  }
}
```

### Example — RTK Query

```ts
// store/officeApi.ts
import { officeFromBe, newOfficeToBe } from './officeSerializer';
import type { ApiOffice } from './officeSerializerType';

export const officeApi = createApi({
  reducerPath: 'officeApi',
  baseQuery: customBaseQuery,
  endpoints: (builder) => ({
    getOffices: builder.query<Office[], void>({
      query: () => PathManager.doctor.offices.main(),
      transformResponse: (response: ApiOffice[]) => response.map(officeFromBe),
    }),
    createOffice: builder.mutation<Office, NewOffice>({
      query: (office) => ({
        url: PathManager.doctor.offices.main(),
        method: 'POST',
        body: newOfficeToBe(office),
      }),
      transformResponse: (response: ApiOffice) => officeFromBe(response),
    }),
  }),
});

export const { useGetOfficesQuery, useCreateOfficeMutation } = officeApi;
```

In both shapes the serializer runs before the data leaves `store/`. Raw API shapes never reach the rest of the app.

---

## 🛣️ PathManager — Documented Exception

`src/api/pathManager.ts` is a global, central helper that constructs URLs by name (`PathManager.doctor.appointments.main()`, `PathManager.doctor.patients.byId(id)`).

By the principle of locality of change, you might expect each feature to define its own paths in `<feature>/store/<feature>Paths.ts`. **PathManager is intentionally an exception.**

### Why the exception

PathManager is **plumbing for the FE/BE boundary**, not feature logic. It mirrors the contract the BE defines, not decisions the FE makes. The same way a generated OpenAPI client is a single global file (because it reflects the BE schema), PathManager lives outside the feature tree because it represents the BE's surface area.

### What this preserves

The rule "BE calls live in `store/`" still holds: PathManager **constructs** URLs (it's a string helper), but it never **invokes** the BE. The actual call happens inside `store/` — saga, `createApi`'s `query`, async action. PathManager being global doesn't put I/O outside `store/`; it just centralizes the URL strings.

### Convention for new features

When adding a new feature, add its paths to `PathManager` in a dedicated section by domain. Do not create `<feature>/store/<feature>Paths.ts` — keeping all paths in one place lets you see the entire BE surface at a glance and matches how the BE team owns the contract.

---

## 🔑 Async Orchestration Must Not Call UI Libraries Directly

Code in `store/` orchestrates business logic and side-effects — it must not render UI or trigger UI side-effects (toasts, alerts, modals, navigation) by importing a UI library directly. This applies to **sagas, RTK Query callbacks, Zustand actions, thunks** — any orchestration code in `store/`.

### Why

UI side-effects are presentation concerns. Coupling them into orchestration code creates two problems: (1) every test needs to mock the UI library, (2) swapping the library (toast → snackbar, react-router → tanstack-router) requires touching every feature.

### The pattern: dispatch a domain action, let a dedicated feature handle the UI

```ts
// ❌ Saga coupled to react-hot-toast
import toast from 'react-hot-toast';

function* createAppointmentSaga(action) {
  try {
    yield call(api.post, '/appointments', action.payload);
    yield put(createAppointmentSuccess());
    toast.success('Turno asignado correctamente'); // ❌ presentation concern in store/
  } catch (error) {
    toast.error('Error al asignar el turno'); // ❌
  }
}
```

```ts
// ✅ Saga dispatches an action — no UI library import
import { showToast } from '@/features/notifications/store/notificationsActions';

function* createAppointmentSaga(action) {
  try {
    yield call(api.post, '/appointments', action.payload);
    yield put(createAppointmentSuccess());
    yield put(showToast({ type: 'success', message: 'Turno asignado correctamente' }));
  } catch (error) {
    yield put(showToast({ type: 'error', message: 'Error al asignar el turno' }));
  }
}
```

```ts
// ✅ The same pattern with RTK Query
createOffice: builder.mutation<Office, NewOffice>({
  query: (office) => ({ url: PathManager.doctor.offices.main(), method: 'POST', body: newOfficeToBe(office) }),
  async onQueryStarted(_arg, { dispatch, queryFulfilled }) {
    try {
      await queryFulfilled;
      dispatch(showToast({ type: 'success', message: 'Office created' }));
    } catch {
      dispatch(showToast({ type: 'error', message: 'Failed to create office' }));
    }
  },
}),
```

The `notifications` feature owns the actual toast call:

```ts
// src/features/notifications/store/notificationsSaga.ts
// The only place react-hot-toast is imported in the entire app
import toast from 'react-hot-toast';
import { takeEvery } from 'redux-saga/effects';
import { showToast } from './notificationsActions';

function handleShowToast(action: ReturnType<typeof showToast>) {
  if (action.payload.type === 'success') toast.success(action.payload.message);
  else toast.error(action.payload.message);
}

export function* watchNotificationsSaga() {
  yield takeEvery(showToast.type, handleShowToast);
}
```

### Benefits

- Swapping `react-hot-toast` for another library touches one file (`notificationsSaga.ts`)
- Orchestration code in every feature stays testable without UI library mocks
- The concern is explicit: `store/` says **what happened**; `notifications` decides **how to show it**

---

## ✅ Do

- Keep all I/O definitions in `store/` regardless of library (Saga, RTK Query, Zustand, etc.)
- Run the serializer (or `transformResponse`) before data leaves `store/`
- Build outgoing payloads with the serializer, not by passing domain objects directly
- Dispatch domain actions to express UI side-effects; never call UI libraries from `store/`
- Add new BE paths to `PathManager` in a dedicated domain section

## ❌ Don't

- Trigger BE calls from inside components, hooks, facades, or anywhere outside `store/`
- Bypass the serializer — raw API shapes must never reach the slice/store/state
- Auto-convert snake_case ↔ camelCase (humps, lodash) — always manual mapping
- Call `toast`, `alert`, `history.push`, `useNavigate`, or any UI library from a saga/RTK callback/Zustand action
- Create `<feature>/store/<feature>Paths.ts` — paths go in `PathManager`

---

## 🔗 Related

- [🪝 `hooks/`](./hooks.md) — How hooks invoke the store without containing I/O or domain logic
- [🧱 `domain/`](./domain.md) — Where business rules and validation live
- [🖼️ `view/`](./view.md) — Where store state is consumed (with or without Container)
