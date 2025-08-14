# ReactMesh – A Scalable Frontend Architecture for React

**ReactMesh** is a frontend architecture designed to bring order, scalability, and maintainability to React applications **of any size**.  
It introduces a mesh-based structure that activates only the logic needed for each context, making your codebase clean, modular, and resilient to growth.

---

## 🚀 Start Simple, Scale Naturally

**New to ReactMesh?** [→ See how ReactMesh scales with your features](./scaling.md)

ReactMesh adapts to your feature's actual complexity. **If something doesn't need it, don't add it.**

### 🔹 Simple Feature (just types)

```text
features/socialSecurity/
└── domain/
    └── socialSecurityTypes.ts
```

Perfect for basic data structures. No ceremony, no boilerplate.

### 🔹 Medium Feature (adds business logic)

```text
features/appointments/
├── domain/
│   ├── appointmentTypes.ts
│   └── appointmentFacade.ts
└── hooks/
    └── useAppointments.ts
```

When you need validation or business rules, add what's necessary.

### 🔹 Complex Feature (full separation)

```text
features/medicalRecords/
├── view/              # UI components
├── domain/            # Business logic & validation
├── store/             # State & async operations
├── presentation/      # Display formatting
└── hooks/             # UI event handlers
```

For complex domains that need complete architectural separation.

---

## 🔄 Add to Existing Apps - Zero Rewrites Required

**Have an existing React app?** [→ See how to add ReactMesh without breaking anything](./migration.md)

ReactMesh **grows alongside your current code** - no migration necessary.

### Start Today (5 minutes):

1. Create `src/features/` folder
2. Build your next feature with ReactMesh
3. Keep all existing code exactly as-is
4. **Done!** ReactMesh and legacy code coexist perfectly

### Long-term coexistence:

```text
your-app/
├── src/
│   ├── components/           # ← Legacy code (works forever)
│   ├── pages/               # ← Legacy code (works forever)
│   └── features/            # ← New ReactMesh features
│       ├── notifications/   # ← Better architecture
│       └── userProfile/     # ← Cleaner testing
```

**Key insight:** Legacy code can stay legacy indefinitely. ReactMesh is for new features only.

---

## 🤔 Why ReactMesh Was Created

Most frontend architectures force you to create unnecessary layers even for simple features.  
They're adaptations of backend paradigms (MVC, MVVM, Hexagonal) that weren't designed for modern React needs.

**ReactMesh** was born from real-world frustration: _"Why do I need 5 files for a simple feature?"_

It's a **frontend-native architecture** that grows organically with your actual requirements — without borrowing more than necessary.

---

## 🧱 Core Principles

ReactMesh is built around **4 optional layers** that activate only when needed:

| Layer            | When to Use                                                              |
| ---------------- | ------------------------------------------------------------------------ |
| **View**         | Always (for rendering components)                                        |
| **Domain**       | When you have business logic, validation, or complex data transformation |
| **Store**        | When you need global state or async operations                           |
| **Presentation** | When you need UI formatting separate from business logic                 |

**🔑 Key insight:** Each layer is **optional**. Add them only when your feature actually needs the separation.

---

## ✨ Real-World Example

Here's how a feature naturally evolves in ReactMesh:

### Step 1: Start with just a component

```tsx
// features/userProfile/view/UserProfile.tsx
const UserProfile = ({ user }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.email}</p>
  </div>
);
```

### Step 2: Add business logic when needed

```text
features/userProfile/
├── view/UserProfile.tsx
└── domain/
    ├── userTypes.ts
    └── userFacade.ts     # Added validation
```

### Step 3: Add state when it becomes global

```text
features/userProfile/
├── view/UserProfile.tsx
├── domain/
├── store/               # Added global user state
└── hooks/               # Added UI logic separation
```

**No premature abstraction. No unnecessary ceremony.**

---

## 📘 Documentation

### 🚀 Getting Started

- **[Start Simple, Scale Naturally](./scaling.md)** - How ReactMesh grows with your features
- **[Adding to Existing Apps](./migration.md)** - Gradual adoption without rewrites

### 🧱 Architecture Guides

- [Philosophy](./philosophy.md) - Why ReactMesh exists
- [Architecture Overview](./architecture.md) - Complete structure guide
- [Architecture Scope](./scope.md) - What ReactMesh covers (and doesn't)

### 📚 Layer Details

- [Domain Layer](./domain.md) - Business logic & validation
- [Store Layer](./store.md) - State & async operations
- [View Layer](./view.md) - UI rendering components
- [Presentation Layer](./presentation.md) - Display formatting

### ⚙️ Practical Guides

- [Hooks Structure](./hooks.md) - UI event handlers
- [Testing Philosophy](./testing.md) - How to test each layer
- [Conventions & Rules](./conventions.md) - Naming & organization

---

## 🎯 Who is this for?

- **Developers tired of over-engineered architectures** that force ceremony
- **Teams scaling React apps** without losing maintainability
- **Senior developers** who want pragmatic, battle-tested solutions
- **Anyone with existing React apps** who wants better structure for new features
- **Anyone who believes** in "complexity when needed, simplicity by default"

---

## 🧠 Battle-Tested Origins

ReactMesh was created by **Nestor Oscar Domínguez Díaz** while building a complex medical application solo.  
Born from real-world necessity, not academic theory.

- **12+ years development experience**
- **Senior Fullstack Developer & Software Architect**
- **Creator of Diagnosisfy** (Private healthcare SaaS)
- **Rails + React expert** with deep architectural knowledge
- **Proven in production** for 1+ years on large-scale applications

---

## 📣 Community & Contributions

ReactMesh is a **personal architecture** proven in production, but community input is welcome.

- 💬 **Questions?** Open an issue for architectural guidance
- 🐛 **Found problems?** Submit issues with documentation improvements
- 💡 **Ideas?** Share your experience adapting ReactMesh to your projects

**Remember:** ReactMesh works best when you resist the urge to add layers "just in case."  
Start simple. Add complexity only when your features actually demand it.

---

## 🚀 Quick Start

### For New Projects:

1. **Clone or reference** this repository for the architectural guidelines
2. **Start with a simple feature** using just `view/` and basic components
3. **Add domain logic** when you need validation or business rules
4. **Introduce store** when state becomes global or async operations are needed
5. **Scale naturally** — let your features guide the architecture, not the other way around

### For Existing Projects:

1. **Create `src/features/` folder** in your existing app
2. **Build your next new feature** using ReactMesh structure
3. **Keep all existing code unchanged** - zero migration required
4. **See benefits immediately** - better structure, easier testing, cleaner code

**The goal:** Build features that are exactly as complex as they need to be. Nothing more, nothing less.
