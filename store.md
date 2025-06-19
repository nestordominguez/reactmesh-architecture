# 📦 `store/`

The `store/` layer handles **state management** and **async orchestration**. It serves as the bridge between the UI and backend services or other side-effects.

This layer is **store-agnostic** — you can use Redux, Zustand, Jotai, or any state library. What matters is where and how orchestration happens.

---

## 🧭 Responsibilities

* Centralize and manage global state
* Orchestrate asynchronous flows (e.g. fetch, update, delete)
* Use the appropriate `serializer/` to communicate with the backend
* Trigger side-effects in response to user actions or lifecycle events

---

## 📂 Structure

```text
src/features/x/store/
├── xSlice.ts             # Redux slice or Zustand store
├── xSaga.ts              # Saga or async orchestration logic (if needed)
└── __tests__/            # Unit tests for async flows and reducers
```

---

## 📦 Serializer Use

When making API calls, `store/` uses functions from `serializer/` to adapt request and response formats.

ReactMesh recommends:

* Always implement a **FE → BE** serializer to send well-structured payloads.
* When needed, create a **BE → FE** serializer if the backend response format is not frontend-ready.
* Ideally, implement serializers **on both ends**: the frontend adapts data before sending, and the backend adapts it before responding. This keeps boundaries clear and minimizes coupling.

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

```ts
// src/features/office/serializer/officeSerializer.ts

export const officeSerializer = {
  fromBe: (payload: OfficeApiResponse): Office => ({
    id: payload.id,
    address: payload.addr,
    city: payload.city_name,
    ...
  }),

  toBe: (office: Office): OfficePayload => ({
    id: office.id,
    addr: office.address,
    city_name: office.city,
    ...
  })
}
```

---

## ✅ Do

* Keep orchestration logic in sagas, thunks, or async handlers — not in the component or domain
* Use serializers to adapt data before calling the backend and after receiving a response
* Separate orchestration logic from UI and business logic

---

## ❌ Don’t

* Trigger async operations from inside components, hooks, or facades
* Bypass the store for shared/global data flows
* Mix data formatting, presentation, or validation into `store/` logic

---

## 🔗 Related

* [📑 `domain/`](./domain.md) — For business logic and validations
* [📦 `view/`](./view.md) — Where store state is consumed
