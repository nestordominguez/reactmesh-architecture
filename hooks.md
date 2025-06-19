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

## ✅ Do

- Use hooks to separate event handling from views
- Use `useCallback`, `useMemo`, or local `useState` if needed
- Keep the API of the hook focused on the view's needs

## 🚫 Don’t

- Perform rendering or return JSX
- Contain business logic (should be delegated to `domain/`)
- Manage global state directly (delegate to `store/`)

---

## ✅ Example: `useOfficeHandlers`

```tsx
// src/features/office/hooks/useOfficeHandlers.ts

export const useOfficeHandlers = () => {
  const dispatch = useAppDispatch()
  const offices = useAppSelector(selectOffices)

  const handleAddOffice = () => {
    dispatch(startCreatingOffice())
  }

  return {
    offices,
    handleAddOffice
  }
}
```
This hook connects the view to Redux logic in store/ and provides a clean interface.

## 🚫 Bad Example: `useOfficeHandlers`

```tsx
// ❌ Don’t include rendering in hooks

export const useOfficeHandlers = () => {
  return (
    <div>
      <h1>Should never return JSX!</h1>
    </div>
  )
}
```
Hooks are logic providers — they should never render UI.

## 🚫 Bad Example: Business Logic in Hooks
```tsx
// ❌ Don’t put validations or core rules in hooks

export const useFormValidation = () => {
  const validateForm = (data: FormData) => {
    if (!data.name) throw new Error("Missing name") // 🚫
  }

  return { validateForm }
}
```
Business rules like validations belong in domain/model/, not in hooks.

## 🧵 Notes
Hooks are essential to keeping views clean and declarative.
They serve as the bridge between the UI and the rest of the architecture, but do not contain logic themselves.

## 🔗 Related

📦 view/ — Where hooks are consumed

🏗️ architecture.md — Overview of layer responsibilities

🧱 domain/ — Where validation and business logic should live