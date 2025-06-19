# ReactMesh – A Scalable Frontend Architecture for React

**ReactMesh** is a frontend architecture designed to bring order, scalability, and maintainability to large React applications.  
It introduces a mesh-based structure that activates only the logic needed for each context, making your codebase clean, modular, and resilient to growth.

---

## 🤔 Why ReactMesh Was Created

Most frontend architectures are adaptations of backend paradigms like MVC, MVVM, Hexagonal Architecture, or even Domain-Driven Design (DDD).  
While useful, they weren’t designed for the real needs of modern React apps.

**ReactMesh** was born from that gap.  
It’s a **frontend-native architecture** that brings clarity to business logic, data transformation, and state orchestration — without borrowing more than necessary.

---

## 🚀 Key Concepts

ReactMesh is structured around four core layers:

| Layer          | Responsibility |
|----------------|----------------|
| **View**         | Components responsible for UI rendering only |
| **Domain**       | Business logic and validation. Exposes a `facade` that delegates to `models`, `rules`, or structured logic |
| **Store**        | State orchestration and side-effects. Agnostic to Redux, Zustand, Jotai, etc. |
| **Presentation** | UI formatting, scoped hooks, and display-level utilities |

> Additional shared modules like `serializer/` live outside the core layers and can be reused across features (especially within sagas).

---

## 🗂️ Folder Structure

```text
reactmesh-architecture/
├── README.md
├── philosophy.md
├── architecture.md
├── domain.md
├── store.md
├── view.md
├── presentation.md
├── hooks.md
├── testing.md
├── conventions.md
```
## 📘 Documentation

### 🧱 Core Layers
- [Philosophy](./philosophy.md)
- [Architecture Overview](./architecture.md)
- [Architecture Scope](./scope.md)
- [Domain Layer](./domain.md)
- [Store Layer](./store.md)
- [View Layer](./view.md)
- [Presentation Layer](./presentation.md)

### 🔄 Shared Utilities
- [Serializer](./serializer.md)

### ⚙️ Additional Guides
- [Hooks Structure](./hooks.md)
- [UI Helpers](./helpers.md)
- [Testing Philosophy](./testing.md)
- [Conventions & Rules](./conventions.md)

---

## 📌 Who is this for?

- Senior developers scaling frontend architectures  
- Tech leads looking for modularity and clarity  
- Anyone tired of spaghetti React codebases

---

## 🧠 Designed by

**Oscar Domínguez Díaz**  
Creator of Diagnosisfy (Private SaaS project)  
[GitHub Profile](https://github.com/tu-usuario)  
Senior Fullstack Developer & Software Architect

---

## 📣 Want to contribute?

This is a personal architecture, but suggestions and ideas are welcome.  
Feel free to open issues or contact me directly.
