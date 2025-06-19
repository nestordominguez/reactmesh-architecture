# 📏 Conventions

Consistent conventions eliminate guesswork. In a shared architecture like ReactMesh, predictable file and naming standards ensure that any developer — whether joining from another team or project — can navigate, understand, and contribute without ramp-up time.

By enforcing a common structure across teams and codebases, conventions help:

* Reduce onboarding time
* Minimize mental overhead
* Enable tooling and automation
* Scale architecture across features and teams

This document outlines the naming, file organization, and structural conventions followed in ReactMesh. These rules help keep the codebase predictable, scalable, and easy to navigate across all features.

---

## 🔤 Naming

### 📁 Folders

* Use `camelCase` for all folder names.

  * ✅ `doctorProfile/`
  * ❌ `DoctorProfile/`, `doctor_profile/`

### 📄 Files

* Use `camelCase` for file names.
* The name should reflect the **function** or **type** of the file.

  * ✅ `doctorFacade.ts`, `shiftModel.ts`
  * ❌ `DoctorFacade.ts`, `Shift_model.ts`

---

## 📦 File Placement

Each file should live in the correct layer based on its responsibility:

| Responsibility              | Location                      |
| --------------------------- | ----------------------------- |
| Rendering                   | `view/`                       |
| Form or UI event handlers   | `hooks/`                      |
| Business logic / validation | `domain/` (`model`, `facade`) |
| Data shaping                | `domain/struct/`              |
| API payload handling        | `serializer/`                 |
| Async orchestration         | `store/`                      |
| Display helpers / format    | `presentation/`               |

---

## 🔁 File Naming by Role

| File Type       | Suffix       | Example                    |
| --------------- | ------------ | -------------------------- |
| Model (domain)  | `Model`      | `shiftModel.ts`            |
| Facade (domain) | `Facade`     | `officeFacade.ts`          |
| Struct factory  | no suffix    | `buildOffice.ts`           |
| Redux slice     | `Slice`      | `appointmentSlice.ts`      |
| Saga            | `Saga`       | `shiftSaga.ts`             |
| Serializer      | `Serializer` | `appointmentSerializer.ts` |
| Hook            | `useX`       | `useAssignTurno.ts`        |

---

## 📚 Feature Isolation

All logic should live within its corresponding `features/yourFeature/` directory. Do **not** share domain, store, or serializers across features unless it’s a truly global utility.

For global concerns, prefer placing logic under `shared/`, `api/`, or `hooks/` at the root level.

---

## ✅ Do

* Follow naming conventions consistently
* Group files by feature, not type
* Place logic as close as possible to where it's used
* Split logic into the appropriate layer (`view`, `domain`, `store`, etc.)
* Favor readability and testability

---

## 🚫 Don’t

* Use PascalCase or snake\_case for file or folder names
* Mix rendering with logic in the same file
* Place business rules in components or hooks
* Trigger async operations from domain or presentation

---

## 🔗 Related

* [🏗️ `architecture.md`](./architecture.md) — Overview of the folder and layer responsibilities
* [🧱 `domain/`](./domain.md) — Domain logic conventions
* [📦 `store/`](./store.md) — Async and state conventions
* [🎨 `presentation/`](./presentation.md) — UI formatting guidelines
