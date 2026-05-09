# 🏗️ Architecture Overview

ReactMesh is structured around four main pillars — `view`, `domain`, `store`, and `presentation` — plus shared utilities and serializers that live outside the feature tree.

This document outlines how each part of the system is organized in the filesystem, and how responsibilities are distributed between layers.

---

## 📂 Folder Structure

```text
src/
├── features/
│   └── <featureName>/
│       ├── view/           # Rendering components (and optional containers if using Redux + connect())
│       ├── domain/         # Business logic, validation, facades, models, structs
│       │   ├── *DataTypes.ts
│       │   ├── *Model.ts
│       │   ├── *Facade.ts  # Internal — only used by this feature's hooks
│       │   ├── *Portal.ts  # OPTIONAL — only when other features need to import from this one
│       │   └── struct/     # builders, mutators, selectors
│       ├── store/          # Async orchestration, state management, and serializer
│       │   ├── *Slice.ts / *Api.ts / *Store.ts   # depending on library (Redux Toolkit / RTK Query / Zustand)
│       │   ├── *Saga.ts                          # if using Saga
│       │   ├── *StoreType.ts
│       │   ├── *Serializer.ts                    # API ↔ domain mapping (lives inside store/)
│       │   └── *SerializerType.ts
│       ├── presentation/   # Feature-specific display helpers
│       └── hooks/          # UI event handlers
├── shared/
│   └── presentation/       # Primitive-only display helpers shared across features
└── ...
```

> **Note:** `serializer/` as a sibling of `store/` was the previous layout. It now lives inside `store/`. Existing features still using `features/*/serializer/` are valid until a separate migration task runs.

---

## 🔹 `view/`

Holds only components used for rendering.
There is no business logic or state orchestration here.

The component itself is always pure: it receives data and callbacks via props (or via hooks) and renders. The wiring to the store is **store-agnostic**:

- **With Redux + `connect()`** (optional pattern): `view/` is split into `<componentName>.tsx` (pure) and `<componentName>Container.ts` (connects to Redux via `connect()`).
- **With hooks-direct** (Redux Toolkit, RTK Query, Zustand, TanStack Query): the component or its hook calls `useDispatch`/`useSelector`/`useQuery`/`useStore` directly — no Container needed.

Both patterns are valid. The Container pattern is no longer required; pick the one that matches your library and team preference. What is non-negotiable is **Rule 1 from [`hooks.md`](./hooks.md): BE calls live in `store/`** — the hook or container only triggers them, never contains them.

---

## 🔹 `domain/`

Centralized business logic layer. Contains:

- **`*Model.ts`** – Validates input and enforces business rules. Private to the feature.
- **`struct/`** – Defines pure data-structure helpers. Private to the feature.
  - `builders.ts` creates new structures from defaults or partial input.
  - `mutators.ts` reshapes existing structures into another structure or payload.
  - `selectors.ts` extracts or derives values from existing structures without changing their shape.
- **`*Facade.ts`** – Orchestrates calls to model and struct. **Internal to the feature** — only used by the feature's own hooks.
- **`*Portal.ts`** – **OPTIONAL.** Curated public API; appears only when another feature needs to import from this one. When it exists, it is the only file other features may import from this layer. See [`domain.md`](./domain.md) for the full rule.
- **`*DataTypes.ts`** – Domain interfaces and types. Always camelCase. No logic.

This is the core of feature-level business decisions. It exposes what needs to be done — not how it will be rendered or stored.

---

## 🔹 `store/`

Handles application state and asynchronous orchestration **only when needed**.

- Can use **Redux**, **Zustand**, **Jotai**, or any store solution — ReactMesh is store-agnostic.
- May contain **sagas**, **thunks**, or simple state stores, depending on the feature's complexity.
- If a request is made to fetch data that will be stored globally, the orchestration **must happen inside `store/`**.
- Async orchestration code (sagas, RTK Query callbacks, Zustand actions, thunks) must not call UI libraries (toast, alerts, navigation) directly — dispatch an action instead and let a dedicated feature handle the side-effect.

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

## 🔹 Serializer (lives inside `store/`)

Transforms raw API data into domain models and vice versa. **The serializer is a sub-layer of `store/`, not a sibling layer.** Files: `store/<feature>Serializer.ts` + `store/<feature>SerializerType.ts`.

> **Migration callout:** existing codebases may still have `features/*/serializer/` folders. Decision landed; migration is a separate task — both locations are valid until the migration runs.

- **Private to the feature** — never imported by other features.
- Always called from the orchestration code (saga, `transformResponse`, async action) before the data leaves `store/`.
- snake_case → camelCase mapping is done manually, field by field. Automatic converters are forbidden.
- Defines a local `Api*` type that mirrors the raw API response; the output is the domain model.

See [`store.md`](./store.md) for full rules and per-library examples.

---

## 🔹 `hooks/`

Custom hooks that expose UI handlers to `view/` components.

- One responsibility per hook file.
- May access the store directly (`useDispatch`, `useSelector`, `useQuery`, `useStore`) — the **two non-negotiable rules** are: (1) BE calls live in `store/` (hooks trigger, never define), (2) hooks don't hide domain logic (validation/transformation/decisions live in `domain/facade`).
- Handlers extracted from views always go here.

See [`hooks.md`](./hooks.md) for the full rules and examples.

---

## 🔁 Data & Logic Flow

```text
User → view/ → hooks/ → domain/ (rules) → struct/ → store/ (async + serializer) → presentation/ → view/
```

- **`view/`** triggers interactions and renders results
- **`hooks/`** expose handlers; access the store directly (or receive state/callbacks from a Container if using the optional Redux + `connect()` pattern)
- **`domain/`** validates input or resolves business logic
- **`struct/`** builds, reshapes, or derives structured domain data
- **`store/`** handles async work, side-effects, and the API ↔ domain serializer (the serializer is a sub-layer of `store/`)
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
