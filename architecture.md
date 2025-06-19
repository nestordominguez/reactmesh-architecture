# 🏗️ Architecture Overview

ReactMesh is structured around four main pillars — `view`, `domain`, `store`, and `presentation` — with an additional set of shared utilities like `serializer` that live outside the core flow.

This document outlines how each part of the system is organized in the filesystem, and how responsibilities are distributed between layers.

---

## 📂 Folder Structure

```text
src/
├── view/           # Rendering components only (no logic)
├── domain/         # Business logic, validation, facades, and models
│   ├── facade/
│   ├── model/
│   └── struct/     # Return structures built via factories
├── store/          # Async orchestration and state management
├── presentation/   # UI logic: helpers, display transformers, scoped hooks
├── serializer/     # Shared data transformers (BE ↔ FE)
├── hooks/          # Shared UI event handlers (used by view)
└── ...
```
## 🔹 `view/`

Holds only components used for rendering.  
There is no business logic or state orchestration here.  
If a component needs to trigger logic or access state, it delegates to the `facade`, `store`, or custom hooks.

---

## 🔹 `domain/`

Centralized business logic layer. Contains:

- **`model/`** – Performs validations and encapsulates business rules.
- **`struct/`** – Defines reusable data shapes via factories.
- **`facade/`** – Orchestrates operations. It calls validations or builds structures as needed, but contains no logic itself.

This is the core of feature-level business decisions. It exposes what needs to be done — not how it will be rendered or stored.

---
## 🔹 `store/`

Handles application state and asynchronous orchestration **only when needed**.

- Can use **Redux**, **Zustand**, **Jotai**, or any store solution — ReactMesh is store-agnostic.
- May contain **sagas**, **thunks**, or simple state stores, depending on the feature’s complexity.
- Not all actions are asynchronous — some update the store directly without going through any async middleware.
- If a request is made to fetch data that will be stored globally, the orchestration **must happen inside `store/`**, regardless of the library being used.

This ensures that **global state is never coupled to view or domain logic**, and that all side-effects are handled in a centralized, predictable place.

---

## 🔹 `presentation/`

UI-focused formatting and transformation logic.

- Formatting helpers (e.g., `formatDate()`, `capitalizeName()`)
- Display transformers (e.g., transform color codes, labels, visual mappings)

Keeps `view/` declarative by offloading visual data transformation and formatting.


---

## 🔹 `serializer/`

A shared, feature-agnostic module responsible for transforming API data to and from the frontend format.

- Used primarily in `store/` (whether using sagas, thunks, async functions, or any async orchestration tool) to ensure consistent communication with the backend.
- Keeps transformation logic out of `domain/`, `view/`, and components.
- Promotes reuse, clarity, and testability.

### 🔁 One-way or Two-way?

ReactMesh recommends:

- Always implement **FE → BE** serialization to ensure the frontend sends consistent and structured payloads to the backend.
- **BE → FE** serialization is optional — only needed if the backend doesn't already return frontend-ready data.
- When possible, handle **serialization on both sides** to keep responsibilities decoupled and well-defined.

---
## 🔹 `hooks/`

The `hooks/` directory contains custom hooks that expose UI handlers to the `view/` components.

They are designed to be shared across views and features when needed, while still allowing feature-level organization when appropriate.

These hooks expose UI behavior (e.g., form submission, toggles, button logic) without containing any rendering logic.

They may internally interact with `domain/`, `store/`, or `presentation/`, but always return simple handlers and state to the `view/`.

This ensures that:

- Views remain declarative and clean.
- Handlers can be tested independently.
- Logic can be reused without duplication.

---

## 🔁 Data & Logic Flow (End-to-End)
```text
User → view/ → domain/ (rules) → struct/ → view/ → store/ (async) → presentation/ → view
                                                    ↓
                                                    serializer/
```

- **`view/`** triggers interactions and renders results
- **`domain/`** validates input or resolves business logic
- **`struct/`** builds shaped data
- **`store/`** handles async work or side-effects (e.g. API calls)
- **`serializer/`** is used by `store/` for backend communication
- **`presentation/`** processes display logic and formatting
- The cycle returns to `view/` with final data to render

📛 **Naming Convention**

Folder and file names are consistently written in camelCase across the project.

---

## 🔗 What to Read Next

To go deeper into each pillar of the architecture, we recommend exploring:

- [`view/`](./view.md) – See how the UI is rendered and how it interacts with logic.
- [`domain/`](./domain.md) – Understand how business rules and validations are structured.
- [`store/`](./store.md) – Learn where and how side-effects and async operations are handled.
- [`presentation/`](./presentation.md) – Review how formatting and display logic stay out of view components.

> These modules are independent but work together to ensure a clean, modular frontend architecture.
