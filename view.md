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
- Delegating state, business logic, and UI formatting to `store/`, `domain/`, and `presentation/`
- Keeping all components **stateless and declarative**

---

## ✅ Do

- Use only props or `hooks/` to get handlers and state
- Split visual responsibilities into reusable components
- Keep render logic clean and focused

## 🚫 Don’t

- Call business logic or validations directly
- Perform async operations
- Format or transform data inside components

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

## 🔧 Agregado a OfficesPage

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
❌ Esta vista maneja lógica asincrónica (fetch) y muta estado interno directamente (useState), lo cual rompe con los principios de ReactMesh.

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

## 🔧 Agregado a ShiftForm

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
❌ Esta implementación mezcla validaciones, formatting y llamadas a la API directamente en la vista.

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

## 🔧 Agregado a AssignAppointmentModal

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
❌ Esta vista hace múltiples fetch, guarda estado y orquesta operaciones que deberían ser manejadas por store/, hooks/ y domain/.

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


