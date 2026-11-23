# ADR-0002: Client-Side Persistence Strategy and Schema (IndexedDB via Dexie.js)

> [!IMPORTANT]
> **Deliverable Translation Notice**:
> This document represents the internal, organic Architecture Decision Record drafted during Phase 2 wayfinding.
> Prior to final project submission, this decision **must be translated into the official deliverable format** using `docs/adr-template.md` and saved to `docs/adr/0002-client-side-persistence-strategy-and-schema.md`.

- **Status:** Accepted (for MVP)
- **Date:** 2026-11-20
- **Decision Makers:** Development Team (`jakspt`)
- **Ticket Reference:** [[#9] Select client-side persistence strategy and design schema (ADR-2)](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/9)

---

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project (`ai_docs/phases/phase_2_architecture.md`) and in alignment with [ADR-0001](0001-architecture-variant-selection.md) (Pure Client-Side SPA), we must establish a concrete client-side persistence architecture.

The persistence solution must strictly satisfy two critical Non-Functional Requirements:
1. **NFR-03 (Data Persistence):** After reopening the app or restarting the browser, all successfully saved task details, completion states, availability windows, and generated schedules must remain intact.
2. **NFR-04 (Data Integrity):** If saving a change fails (e.g. storage quota exceeded, I/O error), the application must inform the user and preserve the last successfully saved version, never presenting failed changes as saved.

Furthermore, the persistence architecture must implement the asynchronous **Repository Seam** defined in ADR-0001 (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`), ensuring the presentation layer and domain logic remain decoupled from the concrete storage engine and preserving an evolutionary path to remote backend synchronization.

We must determine the client-side storage engine, the exact database schema, primary keys, indexes, and failure handling strategy.

---

## Decision Drivers

1. **Transactional Data Integrity (NFR-04):** Ability to execute atomic multi-entity mutations (e.g. updating task completion alongside daily schedule recalculation) with automatic rollback on error.
2. **Persistence Reliability & Capacity (NFR-03):** Reliable long-term client-side storage without restrictive storage quotas or forced synchronous serialization.
3. **Asynchronous Non-Blocking Execution:** Completely asynchronous storage operations preventing main-thread UI jank, perfectly aligning with the async Repository Seam.
4. **Structured Schema & Fast Indexed Queries:** Efficient indexing for task filtering (`isCompleted`, `importance`, `deadline`) and date-based schedule lookups.
5. **Schema Evolution & Migration Safety:** Declarative, automated version upgrade mechanisms allowing non-destructive data schema changes across application releases.
6. **Decoupled Seam for Future Evolution:** Seamless replacement of local storage with remote HTTP/REST adapters (Variant B) without touching UI components.

---

## Considered Options

### Option 1: IndexedDB via Dexie.js (Selected)
- Native browser transactional database (IndexedDB) wrapped with **Dexie.js**, a lightweight (~20 KB), highly performant, TypeScript-first wrapper.
- Fully asynchronous API returning native Promises.
- Full ACID transaction support across multiple object stores with automatic rollback on failure.
- Declarative schema versioning (`db.version(1).stores(...)`) with automated schema migrations.
- Generous browser storage quotas (typically gigabytes, min. 50MB before prompting).

### Option 2: Browser `localStorage`
- Standard Web Storage API providing synchronous key-value string storage.
- Synchronous API that blocks the JavaScript event loop during large serialization/deserialization passes.
- Hard 5 MB total quota per origin.
- Stores strings only, requiring continuous `JSON.stringify` and `JSON.parse` cycles for every read and write.
- No transaction support: partial failures leave state corrupted or half-saved, violating NFR-04.
- No built-in schema migration or indexing mechanisms.

### Option 3: In-Memory State with Periodic `localStorage` Snapshotting
- Keep all state in an in-memory store (e.g. Zustand or Redux) and debounce-dump snapshots into `localStorage`.
- High risk of data loss on sudden browser crash/tab closure before snapshot writes flush (violating NFR-03).
- Lacks ACID guarantees and granular entity updates.

---

## Decision Outcome

**Chosen Option:** **Option 1: IndexedDB via Dexie.js**

### Justification
1. **Fulfills NFR-04 via Native ACID Transactions:** Dexie provides native transactional boundaries (`db.transaction('rw', ...)`). If any operation within a transaction throws (e.g. disk full, quota exceeded), Dexie aborts the entire transaction automatically, leaving the database in its last consistent state.
2. **Aligns with Asynchronous Repository Seam:** Dexie methods naturally return `Promise<T>`, allowing repository implementations (`IndexedDbTaskRepository`, etc.) to implement asynchronous domain interfaces without synthetic `async` shims.
3. **Robust Schema Versioning:** Dexie’s declarative versioning and `upgrade()` callbacks allow future feature expansions to migrate existing client databases safely without data loss.
4. **Performance & Scale:** Asynchronous, non-blocking indexed queries prevent UI frame drops even with hundreds of historical tasks.

---

## Concrete Database Schema & Entity Specification

### 1. Object Stores and Indexes

The database (`SmartTaskManagerDB`) defines three normalized object stores in Dexie:

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

---

## Detailed Architectural Decisions

### 1. Availability Window Duration: Derived Domain Property
- **Decision:** Do not persist a redundant `durationMinutes` column in `availabilityWindows`.
- **Rationale:** `startTime` and `endTime` are the single source of truth as required by FR-DP-01 (`startTime < endTime`). Window duration is a pure derivation:
  $$\text{durationMinutes} = \text{toMinutes}(endTime) - \text{toMinutes}(startTime)$$
- Storing a redundant column would introduce the risk of data drift or synchronization anomalies during updates.

### 2. Daily Schedule Representation: Date Aggregate Document
- **Decision:** Persist `dailySchedules` as a single aggregate record per date (`date` as primary key).
- **Rationale:** According to `docs/mvp.md` (§4.1, §4.2), scheduling operates on a date-by-date basis. Schedule generation or regeneration for Date *X* clears and replaces the entire day's allocation. Storing the schedule as a date-keyed document guarantees atomic replacement in a single write, avoids orphaned slot rows, and eliminates relational join complexity in IndexedDB.

### 3. Dynamic Derivation of Unscheduled Tasks
- **Decision:** Unscheduled tasks are dynamically derived by the application layer/hooks, not stored as an explicit snapshot of IDs.
- **Rationale:** An unscheduled task for Date *X* is any task where `isCompleted === false` and `id` is not present in `dailySchedules.get(date).scheduledTasks`. Deriving this dynamically ensures immediate referential consistency: when a task is edited, completed, or deleted, the unscheduled view updates automatically without requiring cascading updates across historical schedule records.

### 4. Identity Strategy: Client-Side UUID v4
- **Decision:** Primary keys for `tasks` and `availabilityWindows` are generated client-side using `crypto.randomUUID()`.
- **Rationale:** Entities obtain stable, collision-free identities in memory prior to persistence. This simplifies optimistic UI updates, eliminates reliance on auto-increment IDs, and guarantees that local records can be exported, imported, or synced to a backend (Variant B) without primary key collisions.

### 5. NFR-04 Data Integrity & Failure Notification
- **Decision:** All write operations in repository classes are wrapped in Dexie transactions and structured try/catch blocks.
- **Workflow on Failure:**
  1. If storage operations fail (quota exceeded, corrupt disk, aborted transaction), Dexie rolls back all database modifications to the last committed state.
  2. The Repository Seam catches the rejection and throws a typed `PersistenceError`.
  3. The Application layer / React Hook reverts any optimistic UI state and dispatches an informative error toast: *"Failed to save changes. Your previous data was preserved."*
  4. Fulfills NFR-04: The app preserves the last valid state and never presents failed changes as saved.

### 6. Schema Evolution & Versioning Pipeline
- **Decision:** All schema changes must be declared using Dexie's declarative versioning (`db.version(n).stores(...).upgrade(...)`).
- **Rationale:** Guarantees forward compatibility and non-destructive data migrations across application updates, satisfying NFR-03.

---

## Comparison Matrix

| Evaluation Criterion | Option 1: IndexedDB via Dexie.js (Selected) | Option 2: Browser LocalStorage | Option 3: In-Memory + Snapshots |
|---|---|---|---|
| **NFR-03 (Persistence across reloads)** | **High** (Resilient, durable indexed storage) | **Medium** (Vulnerable to 5MB quota overflow) | **Low** (Risk of losing unwritten snapshots) |
| **NFR-04 (Data integrity on failure)** | **High** (Native ACID transactions & rollback) | **None** (No transactions; partial write corruption) | **Low** (Snapshot failure corrupts persistence) |
| **Repository Seam Alignment** | **Native** (Returns Promises; matches ports/adapters) | **Poor** (Synchronous; requires artificial Promise wrapper) | **Poor** (Synchronous memory coupling) |
| **Query Performance & Indexing** | **High** (Native B-Tree indexes on attributes) | **Low** (Full scan & deserialization required) | **High** (In-memory, but slow deserialization) |
| **Storage Capacity** | **> 50 MB - GBs** (Dynamic origin quota) | **~5 MB** (Hard limit) | **~5 MB** (Bounded by localStorage) |
| **Schema Migration Support** | **Built-in** (Declarative `db.version().upgrade()`) | **Manual** (Custom migration scripts required) | **Manual** (Custom state migration required) |
| **Bundle Size Impact** | **Minimal** (~20 KB minified + gzipped) | **Zero** (Native API) | **Minimal** |
