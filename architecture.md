# 🏗️ Architecture Overview

ReactMesh is structured around four main pillars — `view`, `domain`, `store`, and `presentation` — plus shared utilities and serializers that live outside the feature tree.

This document outlines how each part of the system is organized in the filesystem, and how responsibilities are distributed between layers.

---

## 📂 Folder Structure

```text
src/
├── features/
│   └── <featureName>/
│       ├── view/           # Rendering components and containers
│       ├── domain/         # Business logic, validation, facades, models, structs
│       ├── store/          # Async orchestration and state management
│       ├── presentation/   # Feature-specific display helpers
│       ├── serializer/     # API ↔ domain mapping (private to the feature)
│       └── hooks/          # UI event handlers
├── shared/
│   └── presentation/       # Primitive-only display helpers shared across features
└── ...
```

---

## 🔹 `view/`

Holds only components used for rendering.
There is no business logic or state orchestration here.

In React + Redux applications, `view/` is split into two files per component:

- **`<componentName>.tsx`** — pure presentational component; receives everything via props
- **`<componentName>Container.ts`** — connects to Redux via `connect()`; provides `mapStateToProps` and `mapDispatchToProps`

The container is the only place where action creators are imported outside `store/`. Hooks receive callbacks as parameters injected by the container — they never call `useDispatch` or `useSelector` directly.

---

## 🔹 `domain/`

Centralized business logic layer. Contains:

- **`model/`** – Validates input and enforces business rules.
- **`struct/`** – Defines reusable data shapes via factories.
- **`facade/`** – Orchestrates calls to model and struct. The only file other features may import from this layer.
- **`*DataTypes.ts`** – Domain interfaces and types. Always camelCase. No logic.

This is the core of feature-level business decisions. It exposes what needs to be done — not how it will be rendered or stored.

---

## 🔹 `store/`

Handles application state and asynchronous orchestration **only when needed**.

- Can use **Redux**, **Zustand**, **Jotai**, or any store solution — ReactMesh is store-agnostic.
- May contain **sagas**, **thunks**, or simple state stores, depending on the feature's complexity.
- If a request is made to fetch data that will be stored globally, the orchestration **must happen inside `store/`**.
- Sagas must not call UI libraries (toast, alerts) directly — dispatch an action instead and let a dedicated feature handle the side-effect.

---

## 🔹 `presentation/`

Feature-specific formatting and display helpers.

- Private to the feature. Other features must not import from here.
- One function per file, named after what it does.
- **The domain of the input data determines where the helper lives:**
  - Operates on primitives only → `src/shared/presentation/`
  - Receives a domain type → `features/<owner>/presentation/`

---

## 🔹 `shared/presentation/`

Pure helpers that operate only on primitives (`string`, `number`, `Date`, `Dayjs`) and are needed by multiple features.

`shared/presentation/` is not a miscellaneous drawer. The moment a helper receives a domain type, it belongs in the owning feature's `presentation/`.

---

## 🔹 `serializer/`

Transforms raw API data into domain models and vice versa.

- **Private to the feature** — never imported by other features.
- Always called in the saga before dispatching success actions.
- snake_case → camelCase mapping is done manually, field by field. Automatic converters are forbidden.
- Defines a local `Api*` type that mirrors the raw API response; the output is the domain model.

---

## 🔹 `hooks/`

Custom hooks that expose UI handlers to `view/` components.

- One responsibility per hook file.
- Never use `useDispatch` or `useSelector` directly — state and callbacks come as parameters.
- Handlers extracted from views always go here.

---

## 🔁 Data & Logic Flow

```text
User → view/ → hooks/ → domain/ (rules) → struct/ → store/ (async) → serializer/ → presentation/ → view/
```

- **`view/`** triggers interactions and renders results
- **`hooks/`** expose handlers; receive state and callbacks from the container
- **`domain/`** validates input or resolves business logic
- **`struct/`** builds shaped data
- **`store/`** handles async work and side-effects
- **`serializer/`** is used by `store/` for backend communication
- **`presentation/`** formats data for display
- The cycle returns to `view/` with final data to render

---

## 🔗 What to Read Next

- [`view/`](./view.md) – See how the UI is rendered and how it interacts with logic.
- [`domain/`](./domain.md) – Understand how business rules and validations are structured.
- [`store/`](./store.md) – Learn where and how side-effects and async operations are handled.
- [`presentation/`](./presentation.md) – Review how formatting and display logic stay out of view components.
- [`hooks/`](./hooks.md) – How hooks connect the view to the rest of the architecture.
- [`conventions/`](./conventions.md) – Naming and placement rules.
