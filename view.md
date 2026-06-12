# 🖼️ `view/` – UI Rendering Layer

## 🎯 Purpose

The `view/` folder is the **entry point for user interaction** in ReactMesh.  
It holds all visual components and defines what the user sees and interacts with.

It’s a **declarative, logic-free layer** focused exclusively on rendering. All logic — business rules, side-effects, UI handlers — must be delegated.

> In ReactMesh, `view/` is the central pillar where the user meets the system.

---

## 🧭 Responsibilities

- Rendering UI components
- Calling `hooks/` to receive handlers or local state
- Delegating state, business logic, and UI formatting to `store/`, `domain/`, and `view/presentation/`
- Keeping all components **stateless and declarative**

---

## 🔑 Wiring the View to the Store

The view component itself is always pure: it renders from props (or from a hook). How that data and those callbacks reach the component is **store-agnostic** — choose the pattern that fits your library.

### Option A — Hooks-direct (recommended for new code)

The component calls a feature hook; the hook accesses the store directly (`useDispatch`/`useSelector`/`useQuery`/`useStore`). No Container file. Works with Redux Toolkit, RTK Query, Zustand, TanStack Query, etc.

```tsx
// src/features/appointment/view/Appointment.tsx
import { useAppointmentList } from '../hooks/useAppointmentList';

const Appointment = () => {
  const { appointments, loading, loadAppointments, deleteAppointment } = useAppointmentList();
  // ...render
};
```

The hook owns the store wiring. The component is pure rendering. Two files instead of three.

### Option B — Component + Container Pattern (Redux + `connect()`, optional)

Valid when you prefer to keep the view fully Redux-unaware via `connect()`. Each component splits into two files:

```text
view/
└── <componentName>/
    ├── <componentName>.tsx          # Pure presentational component — receives props
    └── <componentName>Container.ts  # connect() only — no logic
```

```ts
// appointmentContainer.ts
const mapStateToProps = (state: RootState) => ({
  appointments: state.appointment.list,
  loading: state.appointment.loading,
});

const mapDispatchToProps = {
  loadAppointments: loadAppointmentsStart,
  deleteAppointment: deleteAppointmentStart,
};

export default connect(mapStateToProps, mapDispatchToProps)(Appointment);
```

This pattern is **no longer required** — it remains valid where it already exists, and is one acceptable choice for new Redux features. Hooks-direct (Option A) is simpler and works with any store library.

**The non-negotiable rule, regardless of pattern**: Rule 1 from [`hooks.md`](./hooks.md) — BE calls live in `store/`. The hook (or container) only triggers them; never contains them.

---

## ✅ Do

- Use only props or `hooks/` to get handlers and state
- If using the Redux + Container pattern (Option B), split each connected component into component + container
- Keep render logic clean and focused

## 🚫 Don’t

- **Define multiple components in a single file.** A component with its own props type is a distinct unit — even if it’s only used by one parent and is not exported. Extract it to its own file (in the same feature’s `view/`, or in the design system if it’s generic).
- Call business logic, validations, or **boolean/domain predicates** directly inside the view component. Any `x.length > 0`, `status === 'active'`, or `is*`/`has*` derivation is a domain query → it lives in `domain/` (the model/facade) and reaches the view as a prop.
- Perform async operations or BE calls inside the view component (move to `store/`)
- Format or transform data inside components — **including computed display strings in JSX, in any form**: template literals, ternaries that return labels, and inline lookup maps (`value → label`). All of these belong in `presentation/`.
- **Style inline.** No `style={{}}`, no `sx`, no styling prop carrying layout/color/spacing/typography (icon size included). Visual styling lives in the feature stylesheet as classes, or via component props. The only inline style allowed is a genuinely runtime-dynamic value (e.g. `width: ${pct}%`).
- Use `useSelector` or `useDispatch` directly inside the view component — the component should be pure; route store access through the hook (Option A) or the container (Option B). Hooks themselves *may* use `useSelector`/`useDispatch` — see [`hooks.md`](./hooks.md).

> **Rules are invariants, not syntax bans.** Each “Don’t” above names a *responsibility that does not belong in the view*; the listed syntaxes (`sx`, template literals, ternaries, lookup maps, `style={{}}`) are **non-exhaustive examples**. When a new syntax appears, ask “is this view computing, styling, or deciding?” — not “is this the exact token the rule named?”. A rule phrased as “no `sx`” silently stops applying the moment the stack drops MUI; a rule phrased as “no inline styling” keeps holding.

> **Architecture overrides “match existing style”.** If the surrounding view already violates these rules, do **not** propagate the violation into new or edited code. “Surgical change” refers to the *scope* of the edit, never to copying a local anti-pattern. Follow the rule in what you add; fix the construct you touch.

---

## 🧪 Example: Offices View

This is a stateless component that renders a list of offices and delegates the behavior to a hook.

```tsx
// src/features/office/view/Offices.tsx

const OfficesPage = () => {
  const { offices, handleAddOffice } = useOfficeHandlers()

  return (
    <section>
      <h1>My Offices</h1>
      <OfficeList offices={offices} />
      <button onClick={handleAddOffice}>Add Office</button>
    </section>
  )
}
```

---

## 🔧 Anti-pattern: OfficesPage

```tsx
// ⚠️ Incorrect example: Violates the architecture by calling logic directly

const OfficesPage = () => {
  const [offices, setOffices] = useState([])

  useEffect(() => {
    fetch("/api/offices")
      .then(res => res.json())
      .then(data => setOffices(data))
  }, [])

  return (
    <section>
      <h1>My Offices</h1>
      <OfficeList offices={offices} />
      <button onClick={() => setOffices([...offices, {}])}>Add Office</button>
    </section>
  )
}

```
❌ This view handles async logic (`fetch`) and mutates internal state directly (`useState`), which breaks the ReactMesh principles. The async I/O belongs in `store/` and the state should come from a hook.

> This view only handles UI and user actions. It does **not** manage state or logic directly.

---

## 🧪 Example: Shift Form

```tsx
// src/features/shift/view/ShiftForm.tsx

const ShiftForm = () => {
  const { shift, updateShift, saveShift } = useShiftForm()

  return (
    <form onSubmit={saveShift}>
      <TimePicker value={shift.startTime} onChange={value => updateShift('startTime', value)} />
      <TimePicker value={shift.endTime} onChange={value => updateShift('endTime', value)} />
      <DaysSelector value={shift.day} onChange={value => updateShift('day', value)} />
      <button type="submit">Save Shift</button>
    </form>
  )
}
```

## 🔧 Anti-pattern: ShiftForm

```tsx
// ⚠️ Incorrect example: Validates directly and formats data inside the view

const ShiftForm = () => {
  const [shift, setShift] = useState({ startTime: "", endTime: "", day: [] })

  const handleSubmit = () => {
    if (!shift.startTime || !shift.endTime) {
      alert("All fields required")
      return
    }

    const payload = {
      ...shift,
      startTime: shift.startTime + ":00", // formatting inside view ❌
    }

    fetch("/api/shifts", {
      method: "POST",
      body: JSON.stringify(payload),
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <TimePicker value={shift.startTime} onChange={value => setShift(s => ({ ...s, startTime: value }))} />
      <TimePicker value={shift.endTime} onChange={value => setShift(s => ({ ...s, endTime: value }))} />
      <DaysSelector value={shift.day} onChange={value => setShift(s => ({ ...s, day: value }))} />
      <button type="submit">Save Shift</button>
    </form>
  )
}
```
---
❌ This implementation mixes validation, formatting, and API calls directly inside the view. Validation belongs in `domain/facade`, formatting in `presentation/`, and the API call in `store/`.

## 🧪 Example: Appointment Assignment Modal

```tsx
// src/features/appointments/view/AssignAppointmentModal.tsx

const AssignAppointmentModal = () => {
  const { patient, availableShifts, assignAppointment } = useAppointmentAssignment()

  return (
    <Modal>
      <h2>Assign Appointment for {patient.name}</h2>
      <ShiftSelector shifts={availableShifts} onSelect={assignAppointment} />
    </Modal>
  )
}
```

## 🔧 Anti-pattern: AssignAppointmentModal

```tsx
// ⚠️ Incorrect example: Handles side effects and state directly

const AssignAppointmentModal = () => {
  const [patient, setPatient] = useState(null)
  const [shifts, setShifts] = useState([])

  useEffect(() => {
    fetch("/api/current_patient")
      .then(res => res.json())
      .then(data => setPatient(data))

    fetch("/api/available_shifts")
      .then(res => res.json())
      .then(data => setShifts(data))
  }, [])

  const assignAppointment = shift => {
    fetch("/api/assign", {
      method: "POST",
      body: JSON.stringify({ patientId: patient.id, shiftId: shift.id }),
    })
  }

  return (
    <Modal>
      <h2>Assign Appointment for {patient?.name}</h2>
      <ShiftSelector shifts={shifts} onSelect={assignAppointment} />
    </Modal>
  )
}
```
---
❌ This view performs multiple `fetch` calls, holds state, and orchestrates operations that should be handled by `store/`, `hooks/`, and `domain/`.

## 🔧 Anti-pattern: Multiple Components in One File

```tsx
// ⚠️ Incorrect: Field is a separate component with its own props, defined in the same file

type FieldProps = {
  id: string;
  label: string;
  value: string;
  error?: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
};

const Field: FC<FieldProps> = ({ id, label, value, error, onChange }) => (
  <div className="form-field">
    <label htmlFor={id}>{label}</label>
    <input id={id} value={value} onChange={onChange} />
    {error && <span>{error}</span>}
  </div>
);

const ProfilePage: FC = () => {
  // ...
  return (
    <form>
      <Field id="name" label="Name" value={form.name} onChange={handleChange} />
      <Field id="email" label="Email" value={form.email} onChange={handleChange} />
    </form>
  );
};

export default ProfilePage;
```
❌ `Field` has its own props type and rendering responsibility — it is a distinct component that belongs in its own file. If it's generic (label + input + error), it belongs in the design system. If it's feature-specific, it goes in a separate file within the feature's `view/`.

---

## 🧵 Notes

All view components should:

- Rely on `hooks/` to access logic or internal state
- Be fully declarative
- Remain easy to test and reason about

This separation ensures that the UI layer stays simple and focused on **what** to render, not **how** the data is handled or transformed.

---

## 🔗 What to Read Next

If you want to understand where the logic lives and how the view connects with the rest of the architecture, we recommend reading:

- [`hooks/`](./hooks.md) – Learn how views access UI behavior through feature-scoped custom hooks.


> Each of these layers plays a critical role in keeping the `view/` declarative, simple, and testable.


