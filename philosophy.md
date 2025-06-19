# 🧠 Philosophy – Why ReactMesh Exists

Most frontend architectures in use today are either directly inherited from the backend world or adapted from general-purpose paradigms like MVC, MVVM, Hexagonal Architecture, or even Domain-Driven Design (DDD).

While some of these ideas are valuable, they were **never created with modern frontend needs in mind** — especially in large-scale, API-driven React applications.

Applying these patterns directly to the frontend often leads to unnecessary complexity, poor separation of UI concerns, and mismatches between the real user experience and domain abstractions.

---

## ❌ The Problem

Frontend development today faces challenges that these architectures fail to address:

- Component logic tightly coupled with business rules
- Validation logic scattered across UI, API, and state layers
- ViewModel-like structures that leak rendering concerns
- Inconsistent or duplicated API serialization/transformation
- State and side-effects blended into rendering components
- Asynchronous workflows tightly coupled to UI state

Trying to force these patterns into the frontend leads to bloated components, duplicated logic, and systems that are hard to scale, test, or understand.

---

## ⚡ The Origin of ReactMesh

> "I created ReactMesh because the architectures we were using were never made for the frontend. We kept adapting backend patterns, hoping they’d fit — but they never truly met our expectations."

ReactMesh was born out of that frustration. It’s not an adaptation.  
It’s a **frontend-native architecture** designed specifically for:

- Component isolation
- Business rule encapsulation
- Data transformation boundaries
- Asynchronous orchestration
- Scoped state clarity
- Store-agnostic integration
- Long-term maintainability

---

## 🧭 Core Philosophy

ReactMesh is based on four foundational pillars:

1. **View** – Dedicated to rendering only. Stateless by design and unaware of business logic.
2. **Domain** – Owns validation and business rules. Exposes a `facade` that delegates to models or calculated logic based on the current need.
3. **Store** – Orchestrates state and side-effects. Not tied to Redux — Zustand, Jotai, or any store can be used per feature.
4. **Presentation** – Handles UI-specific logic such as formatting helpers, display transformers, and scoped hooks for visual composition.

This layered structure enforces clear responsibility boundaries, improves testability, and keeps each feature modular and self-contained.

ReactMesh also draws inspiration from frameworks like **Ruby on Rails**, particularly in its philosophy of *convention over configuration*.

Wherever possible, it encourages predictable folder structures, naming conventions, and minimal setup friction — so developers can focus on feature logic instead of boilerplate.

---

In ReactMesh, **models are not responsible for creating data structures** — they focus on validating inputs or domain rules.  
When a specific structure needs to be returned (e.g. for UI or API preparation), it is defined in the `struct/` folder.

The **facade** coordinates both tasks:
- It may run a validation by calling the model.
- It may build a return structure by calling a `struct` factory.
- Or it may do both — depending on what the feature needs.

The facade itself never contains logic — it simply knows **what to run and when**.

---

## 🔍 Not MVVM. Not Hexagonal. Not Clean.

ReactMesh is inspired by great ideas from other architectures — but it’s not one of them.

- **MVVM** was designed for desktop UIs like WPF, not for reactive web frontends.
- **Hexagonal Architecture** decouples infrastructure well, but doesn’t address frontend-state or component responsibility boundaries.
- **Clean Architecture** imposes too much ceremony for UI development and often creates friction with React’s component-based model.
- **DDD** provides powerful concepts, but many of its abstractions (like aggregates or repositories) become burdensome or misaligned in the UI layer.

ReactMesh is opinionated, pragmatic, and shaped by real-world pain — not academic theory.

---

## 🧱 A Mesh, Not a Monolith

Instead of enforcing a strict architecture across all features, ReactMesh promotes a **reactive mesh** of layers that activate only when needed.

- Lightweight features stay simple.
- Complex features scale without losing control.
- Your architecture grows with your product — not against it.

---

## 🔗 What to Read Next

Now that you understand the core philosophy behind ReactMesh, continue with:

- [`architecture.md`](./architecture.md) – Explore the actual folder structure and how responsibilities are distributed.

> Philosophy is the foundation — the following guides show how it’s applied in practice.
