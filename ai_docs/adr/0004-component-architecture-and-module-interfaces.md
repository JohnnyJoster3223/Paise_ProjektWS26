# ADR-0004: Modular Client Monolith Architecture, Component Design, and Public Module Interfaces

> [!IMPORTANT]
> **Deliverable Translation Notice**:
> This document represents the internal, organic Architecture Decision Record drafted during Phase 2 wayfinding.
> Prior to final project submission, this decision **must be translated into the official deliverable format** using `docs/adr-template.md` and saved to `docs/adr/0004-component-architecture-and-module-interfaces.md`.

- **Status:** Accepted (for MVP)
- **Date:** 2026-11-20
- **Decision Makers:** Development Team (`jakspt`)
- **Ticket Reference:** [[#11] Design component architecture and public module interfaces](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/11)

---

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project (`ai_docs/phases/phase_2_architecture.md`) and in alignment with [ADR-0001](0001-architecture-variant-selection.md) (Modular Client Monolith in React/TypeScript), [ADR-0002](0002-client-side-persistence-strategy-and-schema.md) (Client-Side Persistence via Dexie.js), and [ADR-0003](0003-deterministic-priority-and-greedy-scheduling-engine.md) (Pure Priority Scoring & Greedy Scheduling Engine), we must specify the complete component architecture and public module interfaces.

We must define:
1. **Architectural Layering:** The structural boundaries between Presentation (UI), Application (State/Hooks), Domain Core (Logic), and Persistence (Repository Seam).
2. **Public TypeScript Interfaces:** Concise, high-leverage TypeScript contracts across all system seams without Hungarian prefixes (`TaskRepository`, not `ITaskRepository`).
3. **Application State & Reactivity:** The coordination mechanism connecting async persistence to reactive UI views while strictly satisfying NFR-04 (Data Integrity).
4. **Viewer-Friendly C4 Architecture Visualizations:** C4 Level 2 Container and C4 Level 3 Component diagrams clearly illustrating system boundaries and module interactions.

---

## Decision Drivers

1. **Deep Modules & Clean Seams (`codebase-design`):** Modules must present minimal, highly cohesive interfaces while hiding substantial implementation complexity.
2. **Strict Seam Decoupling:** The Presentation and Domain layers must never directly touch Dexie.js or IndexedDB; all storage interactions must cross the asynchronous **Repository Seam**.
3. **Transactional Integrity & Error Resilience (NFR-04):** State mutations must use pessimistic confirmation (awaiting storage commit before updating React state) to prevent false-positive saved states or UI desynchronization.
4. **Zero-Dependency Domain Core:** Scheduling and priority calculations must remain 100% pure functions accepting plain records and injected timestamps (`now: Date`), completely isolated from React or browser DOM APIs.
5. **Simplicity & Minimal Bundle Size:** Leverage idiomatic React Context and custom hooks rather than heavyweight external state libraries (Redux, Zustand, TanStack Query), keeping bundle weight small and maintenance straightforward.
6. **Viewer-Friendly Visual Clarity:** Architecture diagrams must be instantly comprehensible to reviewers, students, and evaluators.

---

## Decision Outcome

**Chosen Architecture:** **Modular Client Monolith with 4 Layered Tiers**

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. Presentation Layer (React 18 + Vite + Tailwind CSS + shadcn/ui)     │
│    - DashboardView, TaskListView, DailyScheduleView, WindowManagerView │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Application Layer (React Context & Custom Hooks)                    │
│    - TasksContext / useTasks, ScheduleContext / useDailySchedule       │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Domain Core (Pure TypeScript, Zero Dependencies)                    │
│    - calculatePriorityBreakdown, compareTasksByPriority                │
│    - generateDailySchedule, timeUtils (Integer minutes arithmetic)     │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Persistence Layer (Repository Seam & Dexie.js IndexedDB Adapters)   │
│    - TaskRepository, AvailabilityWindowRepository, ScheduleRepository  │
│    - DexieTaskRepository, DexieWindowRepository, DexieScheduleRepo     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Detailed Specifications

### 1. Public TypeScript Module Interfaces

#### A. Persistence Layer: The Repository Seam (`src/repositories/interfaces.ts`)

Following TypeScript conventions, interface names omit the `I` prefix. All methods return native Promises:

```typescript
import {
  TaskRecord,
  AvailabilityWindowRecord,
  DailyScheduleRecord,
} from '../domain/models';

/**
 * Repository Seam contract for Task persistence.
 */
export interface TaskRepository {
  getAll(): Promise<TaskRecord[]>;
  getById(id: string): Promise<TaskRecord | null>;
  save(task: TaskRecord): Promise<void>;
  saveBulk(tasks: TaskRecord[]): Promise<void>;
  delete(id: string): Promise<void>;
}

/**
 * Repository Seam contract for Availability Window persistence.
 */
export interface AvailabilityWindowRepository {
  getByDate(date: string): Promise<AvailabilityWindowRecord[]>;
  save(window: AvailabilityWindowRecord): Promise<void>;
  saveBulk(windows: AvailabilityWindowRecord[]): Promise<void>;
  delete(id: string): Promise<void>;
}

/**
 * Repository Seam contract for Daily Schedule persistence.
 */
export interface DailyScheduleRepository {
  getByDate(date: string): Promise<DailyScheduleRecord | null>;
  save(schedule: DailyScheduleRecord): Promise<void>;
  delete(date: string): Promise<void>;
}
```

#### B. Domain Core Contracts (`src/domain/`)

Pure functions and immutable calculation outputs (specified in ADR-0003):

```typescript
// src/domain/priority/calculatePriority.ts
export interface PriorityBreakdown {
  score: number; // 20 - 100
  urgencyScore: number; // 10 - 60
  importanceScore: number; // 10 - 40
}

export function calculatePriorityBreakdown(
  task: { deadline?: string; importance?: 'LOW' | 'MEDIUM' | 'HIGH' },
  now: Date
): PriorityBreakdown;

export function compareTasksByPriority(
  a: TaskRecord,
  b: TaskRecord,
  now: Date
): number;

// src/domain/scheduling/engine.ts
export interface ScheduledTaskAllocation {
  taskId: string;
  startTime: string; // "HH:mm"
  endTime: string;   // "HH:mm"
  durationMinutes: number;
}

export interface UnscheduledTaskReport {
  taskId: string;
  reason: 'EXCEEDS_REMAINING_CAPACITY' | 'NO_WINDOWS_AVAILABLE';
}

export interface ScheduleResult {
  date: string;
  generatedAt: string;
  scheduledTasks: ScheduledTaskAllocation[];
  unscheduledTasks: UnscheduledTaskReport[];
  totalAllocatedMinutes: number;
  totalAvailableMinutes: number;
}

export function generateDailySchedule(
  tasks: TaskRecord[],
  windows: AvailabilityWindowRecord[],
  targetDate: string,
  now: Date
): ScheduleResult;
```

#### C. Application Layer: Custom Hooks & State Contracts (`src/application/`)

```typescript
// src/application/hooks/useTasks.ts
export interface UseTasksResult {
  tasks: TaskRecord[];
  isLoading: boolean;
  error: Error | null;
  createTask: (data: Omit<TaskRecord, 'id' | 'createdAt' | 'updatedAt'>) => Promise<string>;
  updateTask: (task: TaskRecord) => Promise<void>;
  toggleTaskCompletion: (id: string) => Promise<void>;
  deleteTask: (id: string) => Promise<void>;
  refreshTasks: () => Promise<void>;
}

// src/application/hooks/useDailySchedule.ts
export interface UseDailyScheduleResult {
  date: string;
  schedule: DailyScheduleRecord | null;
  isLoading: boolean;
  isGenerating: boolean;
  error: Error | null;
  generateSchedule: () => Promise<void>;
  clearSchedule: () => Promise<void>;
}
```

---

### 2. State Mutation & Integrity Strategy (NFR-04 Compliance)

1. **Pessimistic State Updates:**
   Custom hooks await successful IndexedDB commits (`await repository.save(entity)`) before updating React state. Because Dexie operations execute locally within ~1–2 ms, there is no noticeable user lag, and false-positive state displays are completely prevented.
2. **Explicit Error Handling & Notifications:**
   If a repository write rejects (e.g., browser quota exceeded), the catch block captures the error, sets `error`, and invokes a non-intrusive toast notification (via shadcn/ui Toast/Sonner), preserving the previously saved in-memory state.

---

### 3. C4 Architecture Visualizations (Viewer-Friendly)

#### A. C4 Level 2: Container Diagram

```mermaid
flowchart TD
    classDef person fill:#08427B,stroke:#073B6F,color:#fff;
    classDef spa fill:#1168BD,stroke:#0B4884,color:#fff;
    classDef storage fill:#1168BD,stroke:#0B4884,color:#fff;
    classDef cdn fill:#666666,stroke:#444444,color:#fff;

    User("User<br/>[Student / Professional]<br/>Manages daily tasks and schedule"):::person

    subgraph BrowserEnvironment ["Client Browser Environment"]
        SPA["Single-Page Application (SPA)<br/>[React 18, TypeScript, Vite]<br/>Executes UI, deterministic prioritization,<br/>and greedy schedule generation"]:::spa
        IndexedDB["Browser IndexedDB<br/>[Dexie.js Client Storage]<br/>Stores tasks, availability windows,<br/>and daily schedules locally"]:::storage
    end

    subgraph StaticHosting ["Static Cloud Edge"]
        Cloudflare["Static CDN Host<br/>[Cloudflare Pages / GitHub Pages]<br/>Serves pre-compiled HTML, JS, CSS"]:::cdn
    end

    User -->|"1. Opens app / Interacts with UI"| SPA
    Cloudflare -.->|"Serves bundle over HTTPS"| SPA
    SPA -->|"2. Reads & writes transactionally<br/>(Async Repository Seam)"| IndexedDB
```

#### B. C4 Level 3: Component Diagram (Modular Client Monolith)

```mermaid
flowchart TD
    classDef ui fill:#2A6F97,stroke:#014F86,color:#fff;
    classDef app fill:#014F86,stroke:#012A4A,color:#fff;
    classDef domain fill:#2C7DA0,stroke:#014F86,color:#fff;
    classDef repo fill:#468FAF,stroke:#014F86,color:#fff;
    classDef db fill:#012A4A,stroke:#012A4A,color:#fff;

    subgraph PresentationLayer ["1. Presentation Layer (UI Views & Controls)"]
        DashboardView["Dashboard View<br/>[React View]"]:::ui
        TaskListView["Task Management View<br/>[React View]"]:::ui
        ScheduleView["Daily Schedule View<br/>[React View]"]:::ui
        UIComponents["UI Primitives<br/>[shadcn/ui + Tailwind CSS]"]:::ui
    end

    subgraph ApplicationLayer ["2. Application Layer (State & Hook Seam)"]
        TasksHook["useTasks / TasksContext<br/>[Custom Hook & Context]"]:::app
        ScheduleHook["useDailySchedule / ScheduleContext<br/>[Custom Hook & Context]"]:::app
        WindowsHook["useAvailabilityWindows<br/>[Custom Hook & Context]"]:::app
    end

    subgraph DomainCore ["3. Domain Core (Pure TypeScript Engine)"]
        PriorityCalc["Priority Calculator<br/>[calculatePriorityBreakdown]<br/>Piecewise urgency & importance scoring"]:::domain
        SchedulerEngine["Scheduling Engine<br/>[generateDailySchedule]<br/>Greedy atomic first-fit slot allocation"]:::domain
        TimeUtils["Time Arithmetic<br/>[timeUtils]<br/>Integer minutes normalization & merging"]:::domain
    end

    subgraph PersistenceLayer ["4. Persistence Layer (Repository Seam)"]
        TaskRepo["TaskRepository<br/>[Interface Seam]"]:::repo
        WindowRepo["AvailabilityWindowRepository<br/>[Interface Seam]"]:::repo
        ScheduleRepo["DailyScheduleRepository<br/>[Interface Seam]"]:::repo
        DexieAdapters["Dexie DB Adapters<br/>[SmartTaskManagerDB]<br/>ACID transactional storage"]:::repo
    end

    subgraph ClientStorage ["Browser Local Storage"]
        IDB[("IndexedDB<br/>Object Stores:<br/>tasks | windows | schedules")]:::db
    end

    %% UI to Application
    DashboardView --> TasksHook
    DashboardView --> ScheduleHook
    TaskListView --> TasksHook
    ScheduleView --> ScheduleHook
    ScheduleView --> WindowsHook
    PresentationLayer -.-> UIComponents

    %% Application to Domain Core
    TasksHook -->|"Sorts tasks using comparator"| PriorityCalc
    ScheduleHook -->|"Invokes pure schedule generation"| SchedulerEngine
    SchedulerEngine --> PriorityCalc
    SchedulerEngine --> TimeUtils

    %% Application to Persistence Layer
    TasksHook -->|"CRUD via async seam"| TaskRepo
    WindowsHook -->|"CRUD via async seam"| WindowRepo
    ScheduleHook -->|"Persists schedule"| ScheduleRepo

    %% Persistence to Database
    TaskRepo --> DexieAdapters
    WindowRepo --> DexieAdapters
    ScheduleRepo --> DexieAdapters
    DexieAdapters -->|"Async read/write"| IDB
```

---

### 4. Codebase Directory Organization

To enforce architectural depth and prevent circular dependencies:

```
src/
├── domain/                  # Tier 3: Pure domain algorithms (zero dependencies)
│   ├── models/              # TypeScript domain types & records
│   ├── priority/            # Priority scoring & comparator
│   └── scheduling/          # Greedy first-fit engine & time arithmetic
├── repositories/            # Tier 4: Public Repository Seam & Dexie adapters
│   ├── interfaces.ts        # TaskRepository, AvailabilityWindowRepository, etc.
│   └── dexie/               # Dexie.js database instance & adapter implementations
├── application/             # Tier 2: Application state, React Contexts, custom hooks
│   ├── context/             # TasksContext, ScheduleContext, AvailabilityContext
│   └── hooks/               # useTasks, useDailySchedule, useAvailabilityWindows
└── components/              # Tier 1: Presentation layer
    ├── views/               # DashboardView, TaskListView, ScheduleView
    ├── layout/              # Navbar, Sidebar, AppLayout
    └── ui/                  # shadcn/ui primitives (Button, Card, Dialog, Toast)
```

---

## Architectural Consequences & Accepted Trade-offs

### Positive Consequences
- **High Architectural Depth:** Domain Core and Repositories expose concise, minimal surfaces while encapsulating robust algorithmic and database logic.
- **Trivial Seam Testing:**
  - Domain algorithms are tested via simple pure input/output unit tests.
  - Custom hooks are tested with in-memory mock repository adapters (`InMemoryTaskRepository`) without IndexedDB overhead.
- **Painless Evolutionary Path:** To evolve to a client-server architecture (Variant B), only the adapters under `src/repositories/` need to be swapped with HTTP fetch implementations; the Domain Core, Application Hooks, and Presentation components remain untouched.

### Accepted Trade-offs
- **Context Re-renders:** Global Context updates cause component tree evaluations; mitigated by splitting into three focused contexts (`TasksContext`, `ScheduleContext`, `AvailabilityContext`) rather than a single massive state blob.
