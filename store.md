# 📦 `store/`

The `store/` layer handles **state management** and **async orchestration**. It serves as the bridge between the UI and backend services or other side-effects.

This layer is **store-agnostic** — you can use Redux, Zustand, Jotai, or any state library. What matters is where and how orchestration happens.

---

## 🧭 Responsibilities

- Centralize and manage global state
- Orchestrate asynchronous flows (e.g. fetch, update, delete)
- Use the appropriate `serializer/` to communicate with the backend
- Trigger side-effects in response to user actions or lifecycle events

---

## 📂 Structure

```text
src/features/x/store/
├── xSlice.ts             # Redux slice or Zustand store
├── xSaga.ts              # Saga or async orchestration logic (if needed)
└── xStoreType.ts         # State type definition
```

---

## 📦 Serializer Use

When making API calls, `store/` uses functions from `serializer/` to adapt request and response formats.

ReactMesh recommends:

- Always implement a **FE → BE** serializer to send well-structured payloads.
- When needed, create a **BE → FE** serializer if the backend response format is not frontend-ready.
- Ideally, implement serializers **on both ends**: the frontend adapts data before sending, and the backend adapts it before responding. This keeps boundaries clear and minimizes coupling.

### ✅ Example

```ts
// src/features/office/store/officeSaga.ts

function* fetchOfficesSaga() {
  try {
    const response: ApiResponse = yield call(api.get, '/offices')
    const offices = response.map(officeSerializer.fromBe) // ✅ BE → FE
    yield put(setOffices(offices))
  } catch (error) {
    yield put(setError(error.message))
  }
}
```

---

## 🔑 Sagas Must Not Call UI Libraries Directly

Sagas orchestrate business logic — they must not render or trigger UI side-effects like toasts, alerts, or modals by importing a UI library directly.

```ts
// ❌ Saga coupled to react-hot-toast
import toast from 'react-hot-toast';

function* createAppointmentSaga(action) {
  try {
    yield call(api.post, '/appointments', action.payload);
    yield put(createAppointmentSuccess());
    toast.success('Turno asignado correctamente'); // ❌ presentation concern in saga
  } catch (error) {
    toast.error('Error al asignar el turno'); // ❌
  }
}
```

Instead, dispatch a domain action that expresses **what happened**. A dedicated feature (`notifications`) listens to that action and handles the UI side-effect.

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
// src/features/notifications/store/notificationsSaga.ts
// The only place react-hot-toast is imported
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

**Why this matters:**
- Swapping `react-hot-toast` for another library requires touching only `notificationsSaga.ts`
- Sagas remain testable without mocking UI libraries
- The concern is explicit: sagas say *what* happened; notifications decide *how* to show it

---

## ✅ Do

- Keep orchestration logic in sagas, thunks, or async handlers — not in the component or domain
- Use serializers to adapt data before calling the backend and after receiving a response
- Separate orchestration logic from UI and business logic
- Dispatch actions to communicate UI side-effects; never call UI libraries directly

---

## ❌ Don't

- Trigger async operations from inside components, hooks, or facades
- Bypass the store for shared/global data flows
- Mix data formatting, presentation, or validation into `store/` logic
- Call `toast`, `alert`, or any UI library directly from a saga

---

## 🔗 Related

- [📑 `domain/`](./domain.md) — For business logic and validations
- [📦 `view/`](./view.md) — Where store state is consumed
