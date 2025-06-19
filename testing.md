# 🧪 Testing in ReactMesh

Testing in ReactMesh follows the same modular philosophy as the architecture itself. Each layer should be tested in isolation, using the most appropriate strategy for its purpose. This enables:

* Fast and focused tests
* Clear separation of concerns
* Easy debugging and maintenance

---

## 📂 Where Tests Live

```text
src/features/x/
├── domain/
│   └── __tests__/         # Model, facade, and struct unit tests
├── store/
│   └── __tests__/         # Reducers, sagas, or async logic
├── hooks/
│   └── __tests__/         # Hook logic
└── view/                  # No tests here (views are declarative)
```

---

## ✅ What to Test by Layer

| Layer           | Purpose                                | Test Type             |
| --------------- | -------------------------------------- | --------------------- |
| `domain/`       | Business logic and validations         | Unit tests            |
| `store/`        | Async flows, reducers                  | Unit + Integration    |
| `hooks/`        | UI handlers (e.g. toggles, form logic) | Unit tests            |
| `presentation/` | Visual formatting helpers              | Pure function tests   |
| `view/`         | Stateless rendering only               | Optional snapshot/E2E |

---

## 🧱 Example: Testing a Model

```ts
// src/features/shift/domain/__tests__/shiftModel.test.ts

import { ShiftModel } from "../shiftModel"

describe("ShiftModel", () => {
  it("is valid when it has day, startTime, and endTime", () => {
    const model = new ShiftModel({ day: ["Mon"], startTime: "08:00", endTime: "12:00" })
    expect(model.isValid()).toBe(true)
  })

  it("is invalid if any field is missing", () => {
    const model = new ShiftModel({ day: [], startTime: null, endTime: null })
    expect(model.isValid()).toBe(false)
  })
})
```

---

## 🚫 Don’ts

* Don’t test rendering in hooks or domain
* Don’t mock store just to test a view
* Don’t couple tests across layers (e.g. a hook test shouldn’t depend on store logic)

---

## ✅ Do

* Keep tests scoped to the file’s responsibility
* Mock dependencies explicitly (e.g., when a facade uses the store)
* Use file co-location under `__tests__` for fast discovery
* Favor unit tests; use integration only when multiple layers interact

---

## 🔗 Related

* [🏗️ `architecture.md`](./architecture.md)
* [🧱 `domain/`](./domain.md)
* [📦 `store/`](./store.md)
* [📏 `conventions.md`](./conventions.md)
