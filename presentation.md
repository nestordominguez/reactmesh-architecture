# 🎨 `presentation/`

The `presentation/` layer is where UI-specific formatting logic lives. It helps keep `view/` components declarative and clean by moving all logic related to **how things should look** out of the render tree.

---

## 🧭 Responsibilities

- Format data for display (e.g., labels, text casing, date/time formatting)
- Map values to presentation formats (e.g., status → color, code → label)
- Keep UI transformations separate from domain and business rules

---

## 📂 Structure

```text
src/features/x/presentation/
└── formatX.ts     # One function per file, named after what it does
```

---

## 🔑 Where a Helper Lives: The Domain Rule

**The domain of the input data determines where the helper lives.**

| Input type | Where the helper lives |
|---|---|
| Primitives only (`string`, `number`, `Date`) | `src/shared/presentation/` |
| A domain type from feature X | `features/x/presentation/` |

### ✅ Examples

```ts
// shared/presentation/formatTime.ts
// Input is a plain string — no domain knowledge required
export const formattedTime = (isoDate: string): string => format(parseISO(isoDate), 'HH:mm');
```

```ts
// features/doctor/presentation/formatDoctorName.ts
// Input is a Doctor domain type — belongs to the doctor feature
export const formatDoctorName = (doctor: Doctor): string =>
  `${doctor.title} ${doctor.surname}, ${doctor.name}`;
```

---

## 🔑 Cross-Feature Display: Use Components, Not Helpers

If feature A needs to display data owned by feature B, **import a `view/` component** from feature B — never its `presentation/` helper.

```tsx
// ✅ medicalRecord imports a Patient view component
import PatientSummary from '@/features/patient/view/patientSummary/patientSummary';

const MedicalRecordHeader = () => (
  <PatientSummary />
);
```

```tsx
// ❌ medicalRecord imports patient's presentation helper — breaks feature isolation
import { formatPatientName } from '@/features/patient/presentation/formatPatientName';
```

The `presentation/` layer is **private to the feature**. The only public surface of a feature is `view/` components and `domain/*Facade`.

---

## 🔑 `shared/presentation/`

Pure helpers that operate only on primitives belong in `src/shared/presentation/`. One function per file, same rules as feature `presentation/`.

```text
src/shared/presentation/
├── formatTime.ts      # formattedTime, formatStartTime, timeStringToDayjs...
└── noop.ts
```

`shared/presentation/` is not a miscellaneous drawer. The moment a helper receives a domain type as a parameter, it no longer belongs there — move it to the owning feature's `presentation/`.

---

## ✅ Examples

### Helper: Format time

```ts
// src/shared/presentation/formatTime.ts
export const formattedTime = (isoDate: string): string =>
  format(parseISO(isoDate), 'HH:mm');
```

### Transformer: Map status to color

```ts
// src/features/appointment/presentation/statusToColor.ts
export const statusToColor = (status: AppointmentStatus): string => {
  switch (status) {
    case 'confirmed': return 'green';
    case 'cancelled': return 'red';
    case 'pending': return 'orange';
    default: return 'gray';
  }
}
```

---

## 🚫 Don't

- Put business logic or validations here (those belong in `domain/`)
- Format data for backend or persistence (use `serializer/` instead)
- Trigger async operations (belongs in `store/`)
- Import another feature's `presentation/` helper — import their `view/` component instead
- Put helpers that receive domain types in `shared/presentation/`

---

## 🔗 Related

- [🖼️ `view/`](./view.md) — Consumes helpers from presentation
- [🧱 `domain/`](./domain.md) — Handles validations and business logic
- [📦 `store/`](./store.md) — Coordinates async operations and global state
