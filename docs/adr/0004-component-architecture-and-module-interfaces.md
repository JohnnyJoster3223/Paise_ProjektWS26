---
status: "accepted"
date: 2026-11-20
last-updated: 2026-11-21
decision-makers: "Jakob Sapototzky"
---

# Adopt Modular Client Monolith Architecture with Four-Tier Layering and Async Repository Seam

## Context and Problem Statement

Following the adoption of a Pure Client-Side Single-Page Application ([ADR-0001](0001-architecture-variant-selection.md)), an IndexedDB persistence engine ([ADR-0002](0002-client-side-persistence-strategy-and-schema.md)), and a pure deterministic scheduling core ([ADR-0003](0003-deterministic-priority-and-greedy-scheduling-engine.md)), we must establish the overall component architecture and public module interfaces for the Smart Task Manager.

We must define:

1. **Architectural Layering:** The structural boundaries between Presentation (UI views and design system), Application (React Contexts and custom hooks), Domain Core (pure algorithms), and Persistence (Repository Seam).
2. **Public TypeScript Contracts:** Clean, cohesive interfaces across system seams following idiomatic TypeScript practices without Hungarian naming prefixes (`TaskRepository`, not `ITaskRepository`).
3. **Application State & Reactivity:** The synchronization mechanism connecting asynchronous persistence to reactive UI views while strictly satisfying NFR-04 (Data Integrity).
4. **Viewer-Friendly Visualizations:** C4 Level 2 Container and C4 Level 3 Component diagrams clearly communicating boundaries and interactions.

## Decision Drivers

* **High Architectural Depth & Clean Seams:** Modules must present minimal, highly cohesive public interfaces while encapsulating substantial algorithmic and storage complexity.
* **Strict Seam Decoupling:** The Presentation and Domain layers must never directly import Dexie.js or touch browser IndexedDB; all storage interactions must cross the asynchronous **Repository Seam**.
* **Transactional State Integrity (NFR-04):** State mutations must use pessimistic confirmation (awaiting storage commit before updating React state) to prevent false-positive saved states or UI desynchronization.
* **Zero-Dependency Domain Core:** Scheduling and priority calculations must remain 100% pure functions accepting plain records and injected timestamps (`now: Date`), completely isolated from React or browser DOM APIs.
* **Simplicity & Minimal Bundle Size:** Leverage idiomatic React Context and custom hooks rather than heavyweight external state libraries (Redux, Zustand, TanStack Query), keeping bundle weight small and maintenance straightforward.
* **Viewer-Friendly Visual Clarity:** Architecture diagrams must be instantly comprehensible to reviewers and evaluators.

## Considered Options

* Option 1: Four-Tier Modular Client Monolith with React Context and Async Repository Seam
* Option 2: Feature-Sliced Design (FSD) with Redux Toolkit and RTK Query
* Option 3: Two-Tier UI-Coupled Architecture (Direct Dexie Calls from React Components)

## Decision Outcome

Chosen option: "Option 1: Four-Tier Modular Client Monolith with React Context and Async Repository Seam", because it establishes clean, defensible boundaries between Presentation, Application, Domain, and Persistence layers, guarantees NFR-04 data integrity through pessimistic state updates, enables isolated unit testing of domain and repository logic, and avoids the unnecessary boilerplate of external state managers.

### Consequences

* Good, because the Repository Seam (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`) strictly decouples data storage from UI components, enabling a frictionless future transition to remote REST APIs (Variant B).
* Good, because the Domain Core remains pure TypeScript with zero runtime dependencies, allowing exhaustive unit testing in Vitest without React hooks or DOM environments.
* Good, because pessimistic hook updates ensure React state accurately reflects durable storage, fulfilling NFR-04.
* Good, because using standard React Context and custom hooks keeps bundle size minimal and avoids external framework lock-in.
* Bad, because global React Context updates trigger component re-evaluations (mitigated by partitioning into three focused domain contexts: `TasksContext`, `ScheduleContext`, and `AvailabilityContext`).
* Neutral, because developer discipline is required to maintain architectural boundaries (enforced via ESLint import restrictions and code reviews).

### Confirmation

Compliance with this decision is confirmed through:

1. **Architectural Boundary Linting:** ESLint rules enforce import boundaries: `src/components/` and `src/domain/` may not import from `src/repositories/dexie/` or `dexie`.
2. **Seam Mock Testing:** Application hooks in `src/application/hooks/` are tested with in-memory repository doubles (`InMemoryTaskRepository`) without IndexedDB or DOM overhead.
3. **Interface Compliance:** Strict TypeScript compilation with `noImplicitAny` and `strictNullChecks` validates that all repository implementations conform to their contracts without Hungarian prefixes.

## Pros and Cons of the Options

### Option 1: Four-Tier Modular Client Monolith with React Context and Async Repository Seam

A classic layered architecture partitioned into Presentation, Application, Domain Core, and Persistence, connected via asynchronous interfaces and React Contexts.

* Good, because boundaries are clear, intuitive, and widely understood by software engineers.
* Good, because domain logic and repository adapters are trivially testable in isolation.
* Good, because it requires zero third-party state management libraries, keeping the bundle lean.
* Good, because the asynchronous Repository Seam preserves an evolutionary path to remote backends.
* Neutral, because context splitting is needed to avoid unnecessary re-renders.
* Bad, because simple CRUD operations require touching multiple layers (interface, repository adapter, hook, view).

### Option 2: Feature-Sliced Design (FSD) with Redux Toolkit and RTK Query

A strict slice-based architectural framework organizing code into layers (app, pages, widgets, features, entities, shared) backed by Redux Toolkit and RTK Query.

* Good, because it enforces strict standardized conventions across large teams.
* Good, because Redux Toolkit offers granular selector-based re-render optimization.
* Bad, because it introduces massive boilerplate, action creators, reducers, and slice definitions for a single-developer or small-team MVP.
* Bad, because Redux Toolkit and RTK Query add ~40–60 KB of unnecessary bundle weight.
* Bad, because FSD conventions are overly rigid for a client-side utility tool, violating the "Keep it Simple" requirement of Phase 2.

### Option 3: Two-Tier UI-Coupled Architecture (Direct Dexie Calls from React Components)

React components directly query and mutate Dexie database tables via Dexie React hooks (`useLiveQuery`).

* Good, because it requires the fewest files and boilerplate for initial rapid prototyping.
* Bad, because business logic and storage calls become deeply entangled in UI components, making automated unit testing difficult.
* Bad, because it completely violates architectural layering and eliminates the Repository Seam, making any future backend migration require a total UI rewrite.
* Bad, because it fails the software architecture evaluation criteria for clean separation of concerns.

## More Information

The modular client monolith is decomposed across four layers:

1. **Presentation Layer:** React 18 UI components, accessible design tokens (Tailwind CSS, shadcn/ui), and views.
2. **Application Layer:** State coordination via custom React hooks (`useTasks`, `useDailySchedule`, `useAvailabilityWindows`) and focused React Contexts using pessimistic persistence confirmation for NFR-04 data integrity.
3. **Domain Layer:** Pure TypeScript business logic (`calculatePriorityBreakdown`, `generateDailySchedule`, minute arithmetic).
4. **Persistence Layer:** Asynchronous Repository Seam (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`) implemented via Dexie.js for IndexedDB.

The unified `TaskRepository` interface exposes five asynchronous methods:

```typescript
export interface TaskRepository {
  getAll(): Promise<TaskRecord[]>;
  getById(id: string): Promise<TaskRecord | null>;
  save(task: TaskRecord): Promise<void>;
  saveBulk(tasks: TaskRecord[]): Promise<void>;
  delete(id: string): Promise<void>;
}
```

For complete C4 architecture diagrams, directory structure, and module interface specifications, see:

* C4 Container & Component Visualizations: [docs/architecture.md](../architecture.md) §5
* Component & Layer Specifications: [docs/architecture.md](../architecture.md) §6
* Client Persistence Engine: [ADR-0002](0002-client-side-persistence-strategy-and-schema.md)
* Algorithmic Core: [ADR-0003](0003-deterministic-priority-and-greedy-scheduling-engine.md)
