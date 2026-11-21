# ADR-0003: Deterministic Priority Scoring and Greedy First-Fit Scheduling Engine

> [!IMPORTANT]
> **Deliverable Translation Notice**:
> This document represents the internal, organic Architecture Decision Record drafted during Phase 2 wayfinding.
> Prior to final project submission, this decision **must be translated into the official deliverable format** using `docs/adr-template.md` and saved to `docs/adr/0003-deterministic-priority-and-greedy-scheduling-engine.md`.

- **Status:** Accepted (for MVP)
- **Date:** 2026-11-20
- **Decision Makers:** Development Team (`jakspt`)
- **Ticket Reference:** [[#10] Design domain core and scheduling engine algorithms](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/10)

---

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project (`ai_docs/phases/phase_2_architecture.md`) and in alignment with [ADR-0001](0001-architecture-variant-selection.md) (Modular Client Monolith) and [ADR-0002](0002-client-side-persistence-strategy-and-schema.md) (Client-Side Persistence & Schema), we must specify the computational core of the application:
1. **Deterministic Priority Calculation (FR-DP-02, FR-DP-03, FR-DR-01):** Transparently ranking unfinished tasks based on user-provided deadlines and importance ratings, with neutral defaults for missing fields.
2. **Automated Daily Schedule Generation (FR-DP-04, FR-DP-05, FR-DR-02):** Automatically assigning unfinished tasks into non-overlapping availability windows for a selected date without splitting tasks.
3. **Domain Purity & Testability:** Designing these algorithms as pure, zero-dependency TypeScript modules with 100% test coverage and zero side effects.

We must define the exact mathematical formulas, data structures, edge-case behaviors, and algorithmic strategies.

---

## Decision Drivers

1. **Anti-Procrastination & Cognitive Explainability (NFR-01):** The priority ranking and schedule must be deterministic, transparent, and explainable to users. Users must understand why a task is scheduled when it is, without feeling controlled by an opaque "black-box AI" algorithm.
2. **Strict Non-Overlapping & Atomic Placement (FR-DP-05):** Tasks must be scheduled as complete, contiguous blocks (no task slicing across windows or days). Overlapping tasks are strictly forbidden.
3. **Graceful Handling of Edge Cases:** Cleanly handling overdue tasks, missing deadlines, missing importance ratings, overlapping user-defined availability windows, and day capacity overflows.
4. **Computational Efficiency & Zero Main-Thread Blocking:** The scheduling engine must run synchronously in the client browser with sub-millisecond execution time for typical daily task volumes ($N \le 100$), avoiding worker overhead.
5. **Architectural Purity & Seam Cleanliness:** Pure domain logic with zero external runtime dependencies (no libraries like `moment`, `date-fns`, or solver packages). Deterministic unit testing enabled by explicit dependency injection of the current time reference (`now: Date`).

---

## Considered Options

### Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring (Selected)
- **Priority Scoring:** Transparent composite score from 0 to 100 ($Score = Urgency [0..60] + Importance [0..40]$) using discrete time-to-deadline buckets and defined fallback defaults.
- **Scheduling Algorithm:** Sort unfinished tasks by deterministic priority, normalize and merge daily availability windows, and sequentially place each task into the earliest window with sufficient contiguous remaining capacity (greedy first-fit).
- **Pros:** Completely deterministic, $O(N \log N)$ sorting and $O(N \times W)$ allocation, zero external solver dependencies, intuitive alignment with user expectations (highest priority tasks are scheduled first).
- **Cons:** Does not guarantee 100% theoretical packing efficiency (e.g. might leave a 15-minute gap if the next task is 30 minutes, even if a later lower-priority 15-minute task exists).

### Option 2: Optimal Knapsack / Exact Dynamic Programming (Rejected)
- Formulate daily schedule generation as a variant of the 0/1 Knapsack Problem or Bin Packing, maximizing total scheduled priority points or total scheduled duration.
- **Pros:** Mathematically optimal utilization of available window minutes.
- **Cons:** Unpredictable and counter-intuitive for users. A lower-priority task might be scheduled ahead of a critical urgent task simply because its duration fills an exact mathematical gap ("knapsack packing anomaly"). This directly violates user trust and increases anti-procrastination anxiety.

### Option 3: Constraint Satisfaction Solver (CSP / Backtracking Search via external JS library) (Rejected)
- Integrate an external constraint programming library (e.g., `js-csp`, `constraint-solver`) with backtracking search to allocate time slots.
- **Pros:** Handles arbitrary multi-variable constraints (e.g., future subtask dependencies, multi-day horizons).
- **Cons:** Massive overengineering for an MVP single-day planner; adds unnecessary bundle bloat (~50–150 KB); potential exponential worst-case runtime; violates the "Keep it Simple" requirement of Phase 2.

---

## Decision Outcome

**Chosen Option:** **Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring**

### Justification
1. **Psychological & Domain Fit:** In task management, strict priority adherence is more important than theoretical bin-packing density. If Task A is due in 1 hour and Task B is due next week, Task A must be scheduled first, even if scheduling Task A leaves an unfilled 15-minute gap.
2. **Predictable & Explainable:** Users can instantly verify why tasks appear in their specific slots.
3. **Pure TypeScript Implementation:** Requires zero third-party dependencies. Can be tested exhaustively with 100% deterministic unit tests.

---

## Detailed Algorithm & Formula Specifications

### 1. Deterministic Priority Calculation (FR-DP-02, FR-DP-03)

The priority score is a composite integer ranging from **20 to 100**, where higher numbers denote higher priority:

$$\text{PriorityScore} = \text{UrgencyScore} (0 \dots 60) + \text{ImportanceScore} (0 \dots 40)$$

#### A. Importance Score Mapping ($0 \dots 40$)
Users assign a discrete `importance` rating (`LOW`, `MEDIUM`, `HIGH`) per FR-TM-04:
- `HIGH`: **40 points**
- `MEDIUM`: **25 points** (also the default fallback if importance is omitted)
- `LOW`: **10 points**

#### B. Urgency Score Mapping ($0 \dots 60$)
Urgency is derived from the time delta between the task's deadline and the reference time:
$$\Delta t = \text{deadlineMs} - \text{nowMs}$$

The score is calculated via discrete piecewise thresholds:
- **Overdue** ($\Delta t < 0$): **60 points** (maximum urgency score; overdue tasks must be confronted immediately)
- **Critical** ($0 \le \Delta t \le 24 \text{ hours}$): **50 points**
- **Urgent** ($24\text{ hours} < \Delta t \le 48 \text{ hours}$): **40 points**
- **Upcoming** ($2\text{ days} < \Delta t \le 7 \text{ days}$): **25 points**
- **Distant** ($\Delta t > 7 \text{ days}$): **10 points**
- **No Deadline** (Omitted / Undefined): **10 points** (neutral baseline matching distant deadlines; allows importance to drive ranking)

#### C. Deterministic 5-Tier Tie-Breaker
When two or more tasks have identical priority scores, ties are resolved deterministically using the following strict hierarchy:
1. **Priority Score** (Descending: higher score first)
2. **Earliest Deadline** (Ascending: tasks with deadlines precede tasks without deadlines; earlier timestamp wins)
3. **Shortest Duration** (Ascending: Shortest Job First heuristic reduces task backlog count)
4. **Creation Timestamp** (`createdAt` Ascending: oldest tasks first to prevent starvation)
5. **Lexicographical Task ID** (Ascending: guarantees absolute, stable ordering)

---

### 2. Availability Window Normalization & Time Arithmetic

To prevent date/timezone parsing bugs and floating-point errors:
1. **Integer Minutes Representation:** All times of day are represented as integer minutes from midnight:
   $$\text{toMinutes}(HH, mm) = HH \times 60 + mm \quad (0 \le \text{minutes} \le 1439)$$
   Formatting back to 24h format:
   $$\text{toTimeString}(mins) = \text{padStart}(\lfloor mins / 60 \rfloor, 2, '0') + ':' + \text{padStart}(mins \% 60, 2, '0')$$
2. **Window Validation & Normalization Pipeline:**
   Before running slot allocation, input windows (`AvailabilityWindowRecord[]`) are processed:
   - **Validation:** Drop any window where $\text{endTimeMinutes} \le \text{startTimeMinutes}$.
   - **Sorting:** Sort remaining windows by `startTimeMinutes` ascending.
   - **Merging Overlaps:** Merge any windows that overlap or are directly adjacent ($\text{window}_{i}.\text{startTime} \le \text{window}_{i-1}.\text{endTime}$).
   - **Result:** An ordered list of disjoint, contiguous availability intervals with zero internal gaps.

---

### 3. Greedy Atomic First-Fit Allocation Algorithm (FR-DP-04, FR-DP-05)

#### Input:
- `tasks: TaskRecord[]` (all tasks in store)
- `windows: AvailabilityWindowRecord[]` (windows defined for the target date)
- `targetDate: string` ("YYYY-MM-DD")
- `now: Date` (injected current time)

#### Algorithm Steps:
1. **Filter Candidate Tasks:**
   $$\text{candidates} = \{ t \in \text{tasks} \mid \neg t.\text{isCompleted} \land t.\text{durationMinutes} > 0 \}$$
2. **Sort Candidates:**
   Sort `candidates` using the 5-tier deterministic priority comparator.
3. **Normalize Windows:**
   Normalize and merge availability windows into `normalizedWindows`. Initialize an allocation cursor for each window at `window.startTimeMinutes`.
4. **Greedy First-Fit Placement:**
   For each task $T$ in sorted `candidates`:
   - Search `normalizedWindows` chronologically for the first window $W$ where:
     $$(W.\text{endTimeMinutes} - W.\text{cursorMinutes}) \ge T.\text{durationMinutes}$$
   - **If found:**
     - Record allocation:
       $$\text{startTime} = \text{toTimeString}(W.\text{cursorMinutes})$$
       $$\text{endTime} = \text{toTimeString}(W.\text{cursorMinutes} + T.\text{durationMinutes})$$
     - Advance $W.\text{cursorMinutes} \mathrel{+}= T.\text{durationMinutes}$.
     - Add to `scheduledTasks`.
   - **If not found in any window:**
     - Add $T$ to `unscheduledTasks` with diagnostic reason:
       - If total available window capacity was 0: `'NO_WINDOWS_AVAILABLE'`
       - Otherwise: `'EXCEEDS_REMAINING_CAPACITY'`
5. **Assemble & Return `ScheduleResult`:**
   ```typescript
   export interface ScheduleResult {
     date: string;
     generatedAt: string;
     scheduledTasks: ScheduledTaskAllocation[];
     unscheduledTasks: {
       taskId: string;
       reason: 'EXCEEDS_REMAINING_CAPACITY' | 'NO_WINDOWS_AVAILABLE';
     }[];
     totalAllocatedMinutes: number;
     totalAvailableMinutes: number;
   }
   ```

---

## TypeScript Domain Module Architecture

All domain logic resides in pure modules within `src/domain/`:

```
src/
└── domain/
    ├── priority/
    │   ├── calculatePriority.ts    # Pure priority score calculation & comparator
    │   └── calculatePriority.test.ts
    └── scheduling/
        ├── timeUtils.ts            # Integer minutes conversions & window normalization
        ├── timeUtils.test.ts
        ├── engine.ts               # Pure greedy first-fit scheduling engine
        └── engine.test.ts
```

### Pure Interfaces (Zero Dependencies)

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
export function generateDailySchedule(
  tasks: TaskRecord[],
  windows: AvailabilityWindowRecord[],
  targetDate: string,
  now: Date
): ScheduleResult;
```

---

## Verification & Test Strategy

Because all modules are pure functions accepting `now: Date`, testing achieves 100% branch coverage with deterministic fixtures:
1. **Urgency Boundary Tests:** Exactly verify scores at $\Delta t = -1\text{ms}$ (overdue: 60), $\Delta t = 0$ (50), $\Delta t = 24\text{h}$ (50), $\Delta t = 24\text{h} + 1\text{ms}$ (40), $\Delta t = 48\text{h}$ (40), $\Delta t = 7\text{d}$ (25), $\Delta t > 7\text{d}$ (10), and undefined deadline (10).
2. **Tie-Breaker Tests:** Verify stability when two tasks have identical scores, verifying order by deadline, duration, `createdAt`, and UUID.
3. **Window Normalization Tests:** Test overlapping, abutting, and out-of-order windows (`09:00-11:00`, `10:30-12:00` $\to$ `09:00-12:00`), and reject inverted windows (`14:00-12:00`).
4. **Scheduling Edge Cases:**
   - Zero available windows $\to$ all tasks unscheduled (`NO_WINDOWS_AVAILABLE`).
   - Task duration exceeds remaining window gap $\to$ skips to next window or marked `EXCEEDS_REMAINING_CAPACITY`.
   - Completed tasks $\to$ ignored by scheduler.
   - Zero-duration tasks $\to$ excluded from scheduling.
