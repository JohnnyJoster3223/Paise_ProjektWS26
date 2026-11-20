---
status: "accepted"
date: 2026-11-20
decision-makers: "Development Team (Jakob Sapototzky / jakspt)"
consulted: "Course Instructors, AI Architectural Advisory"
informed: "Project Stakeholders"
---

# Select IndexedDB via Dexie.js for Client-Side Persistence and Schema Management

## Context and Problem Statement

Following the adoption of a Pure Client-Side Single-Page Application architecture ([ADR-0001](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0001-architecture-variant-selection.md)), we must select a concrete client-side persistence technology and design the underlying storage schema.

The persistence architecture must strictly satisfy two non-functional requirements from [requirements.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/requirements.md):
1. **NFR-03 (Data Persistence):** After reopening the application or restarting the browser, all successfully saved task details, completion states, availability windows, and generated schedules must remain intact.
2. **NFR-04 (Data Integrity):** If saving a change fails (e.g., storage quota exceeded or I/O error), the application must inform the user and preserve the last successfully saved version, never presenting failed changes as saved.

Additionally, the storage mechanism must implement the asynchronous Repository Seam (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`), ensuring the UI presentation and domain core remain cleanly decoupled from storage mechanics and preserving an evolutionary path to remote backend synchronization.

We must determine the storage engine, database schema, primary key generation strategy, index configuration, and failure-handling mechanisms.

## Decision Drivers

* **Transactional Data Integrity (NFR-04):** Ability to execute atomic multi-entity mutations (such as updating task completion alongside daily schedule recalculations) with automatic rollback on error.
* **Persistence Reliability & Capacity (NFR-03):** Reliable, durable client-side storage without restrictive byte quotas or main-thread synchronous serialization locks.
* **Asynchronous Non-Blocking Execution:** Completely asynchronous, Promise-based storage operations to prevent main-thread UI jank while matching the asynchronous Repository Seam.
* **Structured Schema & Indexed Queries:** Efficient indexing for task filtering (`isCompleted`, `importance`, `deadline`) and date-based schedule lookups.
* **Declarative Schema Evolution & Migrations:** Automated version upgrade mechanisms allowing non-destructive schema migrations across application updates.
* **Decoupled Seam for Future Evolution:** Seamless replacement of local storage with remote HTTP/REST adapters without modifying presentation components or domain models.

## Considered Options

* Option 1: IndexedDB via Dexie.js
* Option 2: Browser LocalStorage (Synchronous Key-Value API)
* Option 3: In-Memory State with Periodic LocalStorage Snapshots

## Decision Outcome

Chosen option: "Option 1: IndexedDB via Dexie.js", because it provides native ACID transactions with automatic rollback fulfilling NFR-04, delivers native asynchronous Promise-based APIs aligning directly with the Repository Seam, offers declarative schema migrations, and avoids the 5 MB quota and main-thread serialization bottlenecks of LocalStorage.

### Consequences

* Good, because Dexie.js provides native transactional boundaries (`db.transaction('rw', ...)`), ensuring that any storage failure aborts the entire transaction automatically and leaves stored data in its last consistent state (fulfilling NFR-04).
* Good, because Dexie methods natively return `Promise<T>`, allowing repository classes (`DexieTaskRepository`, etc.) to implement asynchronous domain interfaces without artificial wrappers.
* Good, because declarative schema versioning (`db.version(n).stores(...).upgrade(...)`) provides safe, non-destructive forward migrations for future schema evolutions (fulfilling NFR-03).
* Good, because asynchronous, indexed B-tree queries keep UI interactions responsive even when storing hundreds of historical tasks.
* Bad, because Dexie.js introduces a small library dependency (~20 KB minified + gzipped) into the client bundle.
* Bad, because IndexedDB requires an in-memory mock environment (e.g., `fake-indexeddb`) for running unit and integration tests under Node.js / Vitest.

### Confirmation

Compliance with this decision is confirmed through:
1. **Automated Transaction Rollback Tests:** Integration tests using `fake-indexeddb` simulate storage failures and quota exceptions, verifying that aborted transactions leave existing records unmodified (confirming NFR-04).
2. **Persistence Lifecycle Tests:** Tests verify that task creation, updates, and schedule assignments persist across simulated application reload cycles (confirming NFR-03).
3. **Repository Interface Compliance:** TypeScript type checking ensures all Dexie adapters strictly fulfill the contracts defined in `src/repositories/interfaces.ts`.

## Pros and Cons of the Options

### Option 1: IndexedDB via Dexie.js

The native browser transactional object store (IndexedDB) wrapped by Dexie.js, a lightweight (~20 KB), TypeScript-first library providing ergonomic Promise-based transactions and declarative migrations.

* Good, because it provides robust ACID transactions across multiple stores with automatic rollback upon failure.
* Good, because storage capacity is virtually unlimited for task data (origin storage quotas typically allow hundreds of megabytes).
* Good, because it operates fully asynchronously without blocking the browser event loop during large read/write cycles.
* Good, because it features built-in declarative schema versioning and data migration hooks.
* Neutral, because Dexie adds ~20 KB to the bundle, which is negligible for modern web applications.
* Bad, because IndexedDB APIs are more complex than simple key-value storage and require mock libraries in headless testing environments.

### Option 2: Browser LocalStorage (Synchronous Key-Value API)

The standard Web Storage API providing synchronous key-value string storage directly accessible on `window.localStorage`.

* Good, because it is natively available in all browsers with zero bundle size overhead.
* Good, because it has an extremely simple API (`getItem`, `setItem`).
* Bad, because operations are completely synchronous and block the main JavaScript UI thread during serialization and deserialization of large JSON structures.
* Bad, because it imposes a strict 5 MB total quota per origin, risking unrecoverable write failures as historical task records accumulate.
* Bad, because it lacks transaction support; a failure halfway through updating multiple records leaves state partially written and corrupt, directly violating NFR-04.
* Bad, because it does not support secondary indexes, requiring full array deserialization and in-memory scans for simple queries.
* Bad, because schema migrations must be written entirely by hand without framework safeguards.

### Option 3: In-Memory State with Periodic LocalStorage Snapshots

State is managed entirely in-memory (e.g., via Zustand or React state) and asynchronously debounced or snapshotted into `localStorage`.

* Good, because in-memory reads and writes are instantaneous with zero storage latency.
* Bad, because unwritten state changes are permanently lost if the browser tab is closed or crashes before the debounce timer flushes (violating NFR-03).
* Bad, because snapshot failures still leave storage in an unpredictable, un-rollbackable state (violating NFR-04).
* Bad, because serializing entire state trees becomes progressively slower as task volume grows.

## More Information

### Database Schema Specification (`SmartTaskManagerDB`)

The database defines three normalized object stores managed by Dexie:

```typescript
import Dexie, { Table } from 'dexie';

export interface TaskRecord {
  id: string; // Primary key: UUID v4
  title: string;
  description?: string;
  durationMinutes: number; // Positive integer > 0 (FR-TM-05)
  deadline?: string; // ISO 8601 string (e.g. "2026-11-25T18:00:00Z")
  importance: 'LOW' | 'MEDIUM' | 'HIGH'; // FR-TM-04
  isCompleted: boolean; // FR-DR-04
  createdAt: string; // ISO 8601 UTC
  updatedAt: string; // ISO 8601 UTC
}

export interface AvailabilityWindowRecord {
  id: string; // Primary key: UUID v4
  date: string; // ISO date "YYYY-MM-DD" (FR-DP-01)
  startTime: string; // 24h format "HH:mm" (e.g. "09:00")
  endTime: string; // 24h format "HH:mm" (e.g. "12:30", endTime > startTime)
}

export interface ScheduledTaskAllocation {
  taskId: string;
  startTime: string; // "HH:mm"
  endTime: string;   // "HH:mm"
}

export interface DailyScheduleRecord {
  date: string; // Primary key: "YYYY-MM-DD"
  generatedAt: string; // ISO 8601 UTC
  scheduledTasks: ScheduledTaskAllocation[]; // Chronologically ordered, non-overlapping
}

export class SmartTaskManagerDb extends Dexie {
  tasks!: Table<TaskRecord, string>;
  availabilityWindows!: Table<AvailabilityWindowRecord, string>;
  dailySchedules!: Table<DailyScheduleRecord, string>;

  constructor() {
    super('SmartTaskManagerDB');

    // Schema Version 1
    this.version(1).stores({
      tasks: 'id, isCompleted, importance, deadline, createdAt',
      availabilityWindows: 'id, date',
      dailySchedules: 'date'
    });
  }
}

export const db = new SmartTaskManagerDb();
```

### Architectural Details

1. **Availability Window Duration as Derived Property:**
   Redundant `durationMinutes` columns are intentionally omitted from `availabilityWindows`. `startTime` and `endTime` are the single source of truth; window duration is derived on demand ($\text{toMinutes}(endTime) - \text{toMinutes}(startTime)$) to eliminate data synchronization bugs.
2. **Daily Schedule as Date Aggregate:**
   `dailySchedules` is stored as an aggregate document keyed by `date` (`"YYYY-MM-DD"`). Re-running the scheduling engine replaces the date's allocation atomically in a single write, preventing orphaned slot records.
3. **Dynamic Derivation of Unscheduled Tasks:**
   Unscheduled tasks are derived dynamically by filtering uncompleted tasks not present in `dailySchedules.get(date).scheduledTasks`. This guarantees immediate consistency when tasks are marked complete, edited, or deleted.
4. **Client-Side UUID v4 Identity Strategy:**
   Entity primary keys are generated client-side using `crypto.randomUUID()`, ensuring unique, collision-free identities before persistence and enabling seamless future export/import and cloud sync.
5. **NFR-04 Failure Handling Workflow:**
   Write operations are wrapped in Dexie transactions. If an operation rejects (e.g., storage quota exceeded), Dexie rolls back uncommitted changes. The repository catches the error and throws a typed `PersistenceError`. The application layer reverts any optimistic UI updates and presents an informative toast to the user (*"Failed to save changes. Your previous data was preserved."*).

### Client Persistence Comparison Matrix

| Evaluation Criterion | Option 1: IndexedDB via Dexie.js (Selected) | Option 2: Browser LocalStorage | Option 3: In-Memory + Snapshots |
|---|---|---|---|
| **NFR-03 (Persistence across reloads)** | **High** (Resilient, durable indexed storage) | **Medium** (Vulnerable to 5MB quota overflow) | **Low** (Risk of losing unwritten snapshots) |
| **NFR-04 (Data integrity on failure)** | **High** (Native ACID transactions & rollback) | **None** (No transactions; partial write corruption) | **Low** (Snapshot failure corrupts persistence) |
| **Repository Seam Alignment** | **Native** (Returns Promises; matches ports/adapters) | **Poor** (Synchronous; requires artificial Promise wrapper) | **Poor** (Synchronous memory coupling) |
| **Query Performance & Indexing** | **High** (Native B-Tree indexes on attributes) | **Low** (Full scan & deserialization required) | **High** (In-memory, but slow deserialization) |
| **Storage Capacity** | **> 50 MB - GBs** (Dynamic origin quota) | **~5 MB** (Hard limit) | **~5 MB** (Bounded by localStorage) |
| **Schema Migration Support** | **Built-in** (Declarative `db.version().upgrade()`) | **Manual** (Custom migration scripts required) | **Manual** (Custom state migration required) |
| **Bundle Size Impact** | **Minimal** (~20 KB minified + gzipped) | **Zero** (Native API) | **Minimal** |

### References & Links

* Wayfinding Ticket: [#9 Select client-side persistence strategy and design schema (ADR-2)](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/9)
* Parent Architecture Variant: [0001-architecture-variant-selection.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0001-architecture-variant-selection.md)
* Component & Seam Architecture: [0004-component-architecture-and-module-interfaces.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0004-component-architecture-and-module-interfaces.md)
