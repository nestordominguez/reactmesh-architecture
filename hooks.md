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

## 🔑 Core Rule: Hooks Never Access the Store Directly

Hooks must not use `useDispatch` or `useSelector`. Doing so couples the hook to Redux, which makes it impossible to test in isolation without a full Redux Provider.

**State values** and **action callbacks** are always injected as parameters — typically by the container via `mapDispatchToProps` and `mapStateToProps`.

```ts
// ✅ Hook receives everything as params — no store access
type Params = {
  patient: Maybe<Patient>;
  loading: boolean;
  searchPatient: (dni: string) => void;
};

export const useSearchPatients = ({ patient, loading, searchPatient }: Params) => {
  const { dni } = useParams<{ dni: string }>();

  useEffect(() => {
    if (!patient && dni) searchPatient(dni);
  }, [dni, patient]);

  return { patient, loading, dni };
};
```

```ts
// ❌ Hook accesses the store directly — untestable in isolation
export const useSearchPatients = () => {
  const dispatch = useDispatch();
  const patient = useSelector((state: RootState) => state.appointment.patient);
  const loading = useSelector((state: RootState) => state.patient.loading);

  useEffect(() => {
    if (!patient) dispatch(searchPatientStart());
  }, [patient]);

  return { patient, loading };
};
```

This rule has a direct impact on testing: a hook that receives params needs no Redux Provider, no mocked store, and no `act()` wrappers around dispatch. It's a pure function of its inputs.

---

## 🔑 One Responsibility Per Hook

Never mix concerns in one hook. A hook that manages form state must not also dispatch Redux actions or sync wizard state.

```
use<Feature>Form    → local state + facade calls only
use<Feature>Sync    → only syncs derived state (e.g. wizard steps)
use<Feature>Action  → only triggers a dispatch (receives callback as param)
```

Each hook is independently testable. Form hooks need no Zustand mock. Action hooks need no Redux Provider.

---

## ✅ Example: `useAssignAppointmentForm`

```ts
// ✅ Receives dispatch callbacks as params from the container
type Params = {
  appointmentId: Maybe<string>;
  selectedOffice: Maybe<SavedOffice>;
  patient: Maybe<Patient>;
  createAppointment: (appointment: NewAppointment) => void;
  updateAppointment: (appointment: SavedAppointment) => void;
};

export const useAssignAppointmentForm = ({
  appointmentId,
  selectedOffice,
  patient,
  createAppointment,
  updateAppointment,
}: Params) => {
  const [selectedDay, setSelectedDay] = useState<Maybe<string>>(null);
  // ...
  return { selectedDay, setSelectedDay, handleConfirm };
};
```

The container injects the callbacks:

```ts
// Container
const mapDispatchToProps = {
  createAppointment: createAppointmentStart,
  updateAppointment: updateAppointmentStart,
};
```

---

## 🚫 Don't

- Use `useDispatch` or `useSelector` in hooks
- Perform rendering or return JSX
- Contain business logic (delegate to `domain/`)
- Mix form state management with Redux dispatch in the same hook

---

## 🔗 Related

- [🖼️ `view/`](./view.md) — Where hooks are consumed
- [🏗️ `architecture.md`](./architecture.md) — Overview of layer responsibilities
- [🧱 `domain/`](./domain.md) — Where validation and business logic should live
