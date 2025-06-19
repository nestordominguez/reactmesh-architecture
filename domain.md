# 🧱 `domain/`

The `domain/` layer is the brain of your application. It encapsulates **business logic**, **validations**, and **structuring rules**, ensuring all decisions and operations follow well-defined rules.

This layer does **not** deal with rendering, async operations, or UI logic — it simply defines how data behaves and what’s allowed.

---

## 🧭 Responsibilities

* Encapsulate business rules and validations
* Build domain-specific data structures via factories
* Delegate orchestration to `facade/`, keeping logic clean
* Avoid any dependency on UI, presentation, or async mechanisms

---

## 📂 Structure

```text
src/features/x/domain/
├── featureNameModel.ts       # Validation and domain rules
├── struct/                   # Factories to build structured data
└── featureNameFacade.ts      # Orchestrates calls to model and struct
```

---

## 📑 model/

Responsible for:

* Validating input data
* Enforcing business rules
* Exposing validation status or derived domain decisions

> Note: If a value needs to be normalized, standardized, or built from defaults, that is a responsibility of `struct/` or a dedicated transformer — not the model.

### ✅ Example

```ts
// src/features/shift/domain/shiftModel.ts

export class ShiftModel {
  constructor(private shift: Shift) {}

  isValid(): boolean {
    return !!this.shift.startTime && !!this.shift.endTime && this.shift.day.length > 0
  }

  static createEmpty(): Shift {
    return { startTime: null, endTime: null, day: [] }
  }
}
```

> All validation logic stays here, not in the view or the hook.

---

## 🔟 struct/

Factories for structured data objects used in view, store, or facade.

### ✅ Example

```ts
// src/features/office/domain/struct/buildOffice.ts

export const buildOffice = (officeData: Partial<Office>): Office => ({
  id: officeData.id ?? crypto.randomUUID(),
  address: officeData.address ?? "",
  floor: officeData.floor ?? "",
  ...
})
```

> `struct/` doesn’t validate — it builds data from defaults and inputs.

---

## 🔟 facade/

A thin orchestrator that wires together `model/` and `struct/` calls.
It does **not** contain logic itself.

### ✅ Example

```ts
// src/features/shift/domain/shiftFacade.ts

export const validateAndPrepareShift = (shift: Shift): Result<Shift> => {
  const model = new ShiftModel(shift)

  if (!model.isValid()) return { error: "Invalid shift" }

  return { data: model.shift }
}
```

> A `facade` handles the sequence, not the logic itself.

---

## ❌ Don’t

* Put UI logic, hooks, or presentation helpers in `domain/`
* Access `store/`, `serializer/`, or anything async
* Build rendering data or format for display
* Normalize or structure data **using `struct/`**, which is the designated place within the domain layer for shaping and standardizing values.


---

## 🔗 Related

* [🖼️ `view/`](./view.md) — Uses domain via hooks or directly through handlers
* [⚙️ `hooks/`](./hooks.md) — Calls facade/model to perform operations
* [📦 Store Layer](./store.md)
