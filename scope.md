# 🚫 What ReactMesh Does *Not* Cover

ReactMesh is a **feature-layer architecture**. It provides a structured approach to organizing everything that lives **inside** the `features/` directory of your project.

Anything outside of `features/` is **explicitly out of scope**.

---

## 🔒 Outside ReactMesh Scope

ReactMesh does **not** define conventions, responsibilities, or rules for:

* `App.tsx` or `App.jsx`
* Application entry points
* Global routing (`router/` or `routes/`)
* Global configuration (`config/`, environment settings)
* UI libraries setup (e.g., theme providers, component libraries)
* External service initializers (e.g., Firebase, Sentry)
* Shared or standalone UI components not tied to a feature
* Any file or concern not inside the `features/` tree

---

## ✅ Where ReactMesh *Does* Apply

ReactMesh only governs files and logic located inside the `features/yourFeature/` folder. This includes:

* `view/` — UI components
* `domain/` — Business logic
* `store/` — Async orchestration, state, and the API-data-mapping serializer
* `presentation/` — Display transformations
* `hooks/` — Feature-scoped UI logic

By keeping boundaries clear, ReactMesh allows teams to adopt it gradually or alongside other patterns outside `features/`.

---

## 🧠 Why This Separation Matters

By scoping ReactMesh strictly to the feature layer:

* You can use **any routing, theming, or entry setup** without being locked into an opinionated framework.
* It keeps the architecture **portable** across teams and projects.
* It lets you evolve global structure independently from feature implementation.

ReactMesh is modular on purpose. It's not a framework — it's a system to bring order to the business logic and UI structure of your app *where it matters most*.
