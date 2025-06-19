# 🎨 `presentation/`

The `presentation/` layer is where UI-specific logic lives. This includes helpers, display transformations, and visual mappers only.

It helps keep `view/` components declarative and clean by moving all logic related to **how things should look or behave visually** out of the render tree.

---

## 🧭 Responsibilities

* Format data for display (e.g., labels, text casing, colors)
* Provide UI-specific helpers that assist rendering (e.g., sorting lists, formatting addresses)
* Map values to presentation formats (e.g., status → color)
* Keep UI transformations separate from domain and business rules

---

## 📂 Structure

```text
src/features/x/presentation/
├── helpers/              # Stateless UI helpers (e.g., formatText.ts)
└── transformers/         # Visual mappers (e.g., statusToColor.ts)
```

---

## ✅ Examples

### ✅ Helper: Capitalize a name

```ts
// src/features/user/presentation/helpers/formatName.ts

export const formatName = (name: string) => {
  return name.charAt(0).toUpperCase() + name.slice(1).toLowerCase()
}
```

### ✅ Transformer: Map status to color

```ts
// src/features/appointment/presentation/transformers/statusToColor.ts

export const statusToColor = (status: AppointmentStatus): string => {
  switch (status) {
    case 'confirmed': return 'green'
    case 'cancelled': return 'red'
    case 'pending': return 'orange'
    default: return 'gray'
  }
}
```

---

## 🚫 Don’t

* Put business logic or validations here (those belong in `domain/`)
* Format data for backend or persistence (use `serializer/` instead)
* Trigger async operations (belongs in `store/`)
* Manage scoped UI state — use `hooks/` for that

---

## 🔗 Related

* [🖼️ `view/`](./view.md) — Consumes helpers and transformers from presentation
* [🧱 `domain/`](./domain.md) — Handles validations and business logic
* [📦 `store/`](./store.md) — Coordinates async operations and global state

---
