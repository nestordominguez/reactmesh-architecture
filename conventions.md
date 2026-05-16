# 📏 Conventions

Consistent conventions eliminate guesswork. In a shared architecture like ReactMesh, predictable file and naming standards ensure that any developer — whether joining from another team or project — can navigate, understand, and contribute without ramp-up time.

By enforcing a common structure across teams and codebases, conventions help:

- Reduce onboarding time
- Minimize mental overhead
- Enable tooling and automation
- Scale architecture across features and teams

This document outlines the naming, file organization, and structural conventions followed in ReactMesh. These rules help keep the codebase predictable, scalable, and easy to navigate across all features.

---

## 🔤 Naming

### 📁 Folders

- Use `camelCase` for all folder names.

  - ✅ `doctorProfile/`
  - ❌ `DoctorProfile/`, `doctor_profile/`

### 📄 Files

- Use `camelCase` for file names.
- The name should reflect the **function** or **type** of the file.

  - ✅ `doctorFacade.ts`, `shiftModel.ts`
  - ❌ `DoctorFacade.ts`, `Shift_model.ts`

---

## 📦 File Placement

Each file must live in the correct layer based on its responsibility. This is not a suggestion — it is the contract that makes every feature predictable.

| Responsibility | Location |
|---|---|
| Rendering | `view/` |
| Form or UI event handlers | `hooks/` |
| Business logic / validation | `domain/` (`model`, `facade`) |
| Data shaping / derivation | `domain/struct/` |
| API payload mapping (serializer) | `store/` (sub-layer; lives next to the slice/saga/api) |
| Async orchestration + BE calls | `store/` |
| Feature-specific display helpers | `view/presentation/` |
| Primitive-only display helpers (shared) | `src/shared/presentation/` |

> **Migration callout:** existing features may still place serializers in a sibling `serializer/` folder. The new layout puts them inside `store/`. Both are valid until a separate migration task runs. See [`store.md`](./store.md).

### 🔑 Layer placement is non-negotiable

The layers that exist must never be violated. No file goes loose at the feature root if it belongs to a layer. A domain type always goes in `domain/`, an action or saga always goes in `store/`.

**Why this matters:** The structure is the documentation. A developer opening any feature can immediately find every piece in its expected place. A single file outside its layer forces the reader to rebuild the mental map of the entire feature from scratch — even if the code itself is correct. Predictability is not a cosmetic concern; it is what makes a large codebase readable without prior context.

---

## 🔁 File Naming by Role

| File Type | Suffix | Example |
|---|---|---|
| Model (domain) | `Model` | `shiftModel.ts` |
| Facade (domain) | `Facade` | `officeFacade.ts` |
| Portal (domain, optional) | `Portal` | `shiftPortal.ts` |
| Struct factory | no suffix | `buildOffice.ts` |
| Struct selector | no suffix | `findOpenEntry.ts` or `selectors.ts` |
| Redux Toolkit slice | `Slice` | `appointmentSlice.ts` |
| Redux Saga | `Saga` | `shiftSaga.ts` |
| RTK Query api | `Api` | `appointmentApi.ts` |
| Zustand store | `Store` | `wizardStore.ts` |
| State type | `StoreType` | `appointmentStoreType.ts` |
| Serializer (lives in `store/`) | `Serializer` | `appointmentSerializer.ts` |
| Serializer type (lives in `store/`) | `SerializerType` | `appointmentSerializerType.ts` |
| Hook | `useX` | `useAssignAppointment.ts` |
| Presentation helper | descriptive verb | `formatTime.ts`, `statusToColor.ts` |
| Container (Redux + connect, optional) | `Container` | `appointmentContainer.ts` |

---

## 📚 Feature Isolation

All logic lives within its corresponding `features/yourFeature/` directory.

**Allowed cross-feature imports:**
1. `view/` components
2. `domain/*Portal` (when it exists — Portal is optional, see [`domain.md`](./domain.md))

**Everything else is private:** view/presentation, store (slice/saga/api/serializer), hooks, domain types, model, facade.

If feature A needs to display data owned by feature B, import a `view/` component from B — never B's `view/presentation/` helper.

For genuinely shared utilities that operate only on primitives, use `src/shared/presentation/`.

---

## ✅ Do

- Follow naming conventions consistently
- Group files by feature, not type
- Place every file in its correct layer
- Favor readability and testability

---

## 🚫 Don't

- Use PascalCase or snake\_case for file or folder names
- Define multiple components in a single file — a component with its own props type is a separate unit, even if unexported
- Mix rendering with logic in the same file
- Place business rules in components or hooks
- Put files at the feature root that belong in a layer
- Import `view/presentation/` helpers from another feature
- Put domain collection queries in hooks or views when they belong in `domain/struct/selectors.ts`

---

## 🔗 Related

- [🏗️ `architecture.md`](./architecture.md) — Overview of the folder and layer responsibilities
- [🧱 `domain/`](./domain.md) — Domain logic conventions
- [📦 `store/`](./store.md) — Async and state conventions
- [🎨 `presentation/`](./presentation.md) — UI formatting guidelines
