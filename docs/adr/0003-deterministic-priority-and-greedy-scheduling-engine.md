---
status: "accepted"
date: 2026-11-20
decision-makers: "Development Team (Jakob Sapototzky / jakspt)"
consulted: "Course Instructors, AI Architectural Advisory"
informed: "Project Stakeholders"
---

# Use Deterministic Priority Scoring and Greedy First-Fit Daily Scheduling Engine

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project ([phase_2_architecture.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/ai_docs/phases/phase_2_architecture.md)) and in alignment with our client-side architecture ([ADR-0001](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0001-architecture-variant-selection.md), [ADR-0002](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0002-client-side-persistence-strategy-and-schema.md)), we must design the algorithmic core of the system:
1. **Deterministic Priority Calculation (FR-DP-02, FR-DP-03, FR-DR-01):** Transparently and deterministically rank uncompleted tasks based on deadlines and importance ratings, providing sensible neutral fallbacks for missing attributes.
2. **Automated Daily Schedule Generation (FR-DP-04, FR-DP-05, FR-DR-02):** Automatically assign uncompleted tasks into user-defined, non-overlapping availability windows for a given date without splitting tasks across windows or days.
3. **Domain Purity & Testability:** Ensure algorithms execute as pure, zero-dependency TypeScript modules capable of 100% deterministic test coverage with zero external solver bloat.

We must define the mathematical formulas, time normalization rules, tie-breaking heuristics, and algorithmic allocation strategy.

## Decision Drivers

* **Anti-Procrastination & Cognitive Explainability (NFR-01):** Task priority scoring and schedule generation must be transparent, deterministic, and intuitive. Users must readily comprehend why a task is placed in a given slot, avoiding the skepticism and resistance caused by opaque "black-box" optimization.
* **Strict Non-Overlapping & Atomic Placement (FR-DP-05):** Tasks must be allocated as complete, contiguous blocks without splitting across windows. Overlapping task slots are strictly prohibited.
* **Robust Edge-Case Resilience:** Deterministic, predictable handling of overdue tasks, missing deadlines, missing importance ratings, overlapping user availability windows, and day capacity overflows.
* **Sub-Millisecond Execution & Zero Worker Overhead:** Algorithms must run synchronously on the browser main thread in less than 5 milliseconds for typical backlogs ($N \le 100$), avoiding Web Worker communication complexity.
* **Domain Purity & Zero Dependencies:** Core logic must be isolated from browser DOM, React APIs, or third-party libraries (e.g., date helpers or constraint solvers). Testability is achieved through explicit dependency injection of the current time reference (`now: Date`).

## Considered Options

* Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring
* Option 2: Optimal 0/1 Knapsack / Exact Dynamic Programming Packing
* Option 3: External Constraint Satisfaction Problem (CSP) Solver / Backtracking Search Library

## Decision Outcome

Chosen option: "Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring", because it guarantees predictable, explainable scheduling where high-priority tasks are always placed first (avoiding the counter-intuitive "knapsack packing anomaly"), runs in $O(N \log N)$ time with sub-millisecond latency, and requires zero external solver dependencies.

### Consequences

* Good, because users can immediately verify and trust the schedule order, directly lowering cognitive fatigue and anti-procrastination friction (NFR-01).
* Good, because the composite scoring formula (Urgency 10–60 + Importance 10–40 $\to$ Score 20–100) transparently elevates approaching and overdue tasks while respecting user-assigned importance.
* Good, because a strict 5-tier deterministic tie-breaker eliminates sorting instability and non-deterministic schedule jitter.
* Good, because the domain logic is written in pure TypeScript with zero external dependencies, allowing 100% branch and edge-case test coverage with deterministic fixtures.
* Bad, because greedy first-fit may leave small residual gaps in windows if the next priority task is longer than the remaining gap, even if a lower-priority shorter task exists (accepted intentionally to maintain strict priority precedence).
* Neutral, because multi-day horizon optimization and subtask dependency graph resolution are deferred to post-MVP releases.

### Confirmation

Compliance with this decision is confirmed through:
1. **Automated Unit Test Coverage:** The Vitest test suite achieves 100% statement and branch coverage on `calculatePriorityBreakdown`, `compareTasksByPriority`, and `generateDailySchedule`.
2. **Boundary Condition Tests:** Explicit test assertions verify exact priority scores across all urgency thresholds ($\Delta t = -1\text{ms}$ overdue, $\Delta t = 0\text{h}$, $\Delta t = 24\text{h}$, $\Delta t = 48\text{h}$, $\Delta t = 7\text{d}$, and undefined deadline).
3. **Schedule Invariant Assertions:** Property tests verify that no generated schedule ever contains overlapping task allocations, tasks exceeding window boundaries, or tasks split across disjoint windows.

## Pros and Cons of the Options

### Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring

Sort candidate tasks deterministically by priority score and tie-breakers. Normalize availability windows, then place each task sequentially into the earliest window with sufficient contiguous remaining capacity.

* Good, because it is fully deterministic, highly intuitive, and transparently aligns with user expectations (most critical tasks receive earliest slots).
* Good, because time complexity is $O(N \log N)$ for sorting plus $O(N \times W)$ for placement, executing in $< 1\text{ ms}$ on modern browsers.
* Good, because it requires zero external dependencies, keeping the bundle lean.
* Neutral, because it does not optimize for 100% theoretical bin packing density, which is desirable in task scheduling to prevent priority inversion.
* Bad, because small window fragments may remain unused if priority tasks have long durations.

### Option 2: Optimal 0/1 Knapsack / Exact Dynamic Programming Packing

Formulate schedule generation as a multi-capacity knapsack optimization problem, maximizing total scheduled priority points or total scheduled minutes.

* Good, because it achieves mathematically optimal utilization of available window minutes.
* Neutral, because standard 0/1 knapsack dynamic programming can be implemented without large libraries.
* Bad, because it causes the "Knapsack Packing Anomaly": a low-priority task is scheduled ahead of a critical urgent task simply because its duration fits into an exact remaining gap.
* Bad, because counter-intuitive task ordering destroys user trust, causing frustration and procrastination resistance.
* Bad, because dynamic programming over fine-grained minute grids ($1440 \text{ minutes}$) incurs higher memory and CPU overhead.

### Option 3: External Constraint Satisfaction Problem (CSP) Solver / Backtracking Search Library

Integrate an external constraint programming solver (e.g., `js-csp`, `constraint-solver`) with backtracking search to allocate tasks to slots.

* Good, because it easily accommodates complex multi-variable constraints (e.g., dependency chains, multi-day scheduling, resource conflicts).
* Bad, because it adds substantial external bundle bloat (~50–150 KB) for simple single-day scheduling.
* Bad, because backtracking search exhibits exponential worst-case runtime ($O(2^N)$), creating risk of UI thread freezing.
* Bad, because it introduces unnecessary architectural complexity, violating the "Keep it Simple" requirement of Phase 2.

## More Information

### Detailed Algorithm & Formula Specifications

#### 1. Deterministic Priority Calculation (FR-DP-02, FR-DP-03)

The priority score is a composite integer ranging from **20 to 100**, where higher numbers denote higher priority:

$$\text{PriorityScore} = \text{UrgencyScore} (0 \dots 60) + \text{ImportanceScore} (0 \dots 40)$$

##### A. Importance Score Mapping ($0 \dots 40$)
Derived from user-assigned `importance` (FR-TM-04):
* `HIGH`: **40 points**
* `MEDIUM`: **25 points** (default fallback when importance is omitted)
* `LOW`: **10 points**

##### B. Urgency Score Mapping ($0 \dots 60$)
Urgency is derived from the time delta between the task deadline and the reference timestamp:
$$\Delta t = \text{deadlineMs} - \text{nowMs}$$

The score is calculated via discrete piecewise thresholds:
* **Overdue** ($\Delta t < 0$): **60 points** (maximum urgency score; overdue tasks require immediate attention)
* **Critical** ($0 \le \Delta t \le 24 \text{ hours}$): **50 points**
* **Urgent** ($24\text{ hours} < \Delta t \le 48 \text{ hours}$): **40 points**
* **Upcoming** ($2\text{ days} < \Delta t \le 7 \text{ days}$): **25 points**
* **Distant** ($\Delta t > 7 \text{ days}$): **10 points**
* **No Deadline** (Omitted / Undefined): **10 points** (neutral baseline matching distant deadlines; allows importance to drive ranking)

##### C. Deterministic 5-Tier Tie-Breaker
When multiple tasks share identical composite scores, ties are resolved deterministically using the following strict hierarchy:
1. **Priority Score** (Descending: higher score first)
2. **Earliest Deadline** (Ascending: tasks with deadlines precede tasks without; earlier timestamp wins)
3. **Shortest Duration** (Ascending: Shortest Job First heuristic reduces backlog count)
4. **Creation Timestamp** (`createdAt` Ascending: oldest tasks first to prevent starvation)
5. **Lexicographical Task ID** (Ascending: guarantees absolute, stable ordering)

---

#### 2. Window Normalization & Time Arithmetic

To prevent date/timezone parsing bugs and floating-point errors:
1. **Integer Minutes Representation:** Times of day are stored and processed as integer minutes from midnight ($0 \le \text{minutes} \le 1439$):
   $$\text{toMinutes}(HH, mm) = HH \times 60 + mm$$
   $$\text{toTimeString}(mins) = \text{padStart}(\lfloor mins / 60 \rfloor, 2, '0') + ':' + \text{padStart}(mins \% 60, 2, '0')$$
2. **Window Validation & Normalization Pipeline:**
   Before running slot allocation, raw `AvailabilityWindowRecord[]` entries are processed:
   * **Validation:** Drop any window where $\text{endTimeMinutes} \le \text{startTimeMinutes}$.
   * **Sorting:** Sort remaining windows by `startTimeMinutes` ascending.
   * **Merging Overlaps:** Merge any windows that overlap or abut ($\text{window}_{i}.\text{startTime} \le \text{window}_{i-1}.\text{endTime}$).
   * **Result:** An ordered list of disjoint, contiguous availability intervals.

---

#### 3. TypeScript Domain Contracts (`src/domain/`)

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

### Scheduling Algorithm Comparison Matrix

| Evaluation Criterion | Option 1: Greedy First-Fit (Selected) | Option 2: Knapsack Optimization | Option 3: CSP Solver Library |
|---|---|---|---|
| **Predictability & Explainability** | **High** (Strict priority order) | **Low** (Knapsack packing anomaly) | **Medium** (Black-box backtracking) |
| **Execution Latency** | **< 1 ms** ($O(N \log N)$) | **~5–20 ms** ($O(N \times W)$) | **Variable** (Worst-case $O(2^N)$) |
| **External Dependencies** | **Zero** (Pure TypeScript) | **Zero** (Custom DP) | **Heavy** (~50–150 KB solver library) |
| **Unit Test Purity** | **100%** (Zero mocks required) | **100%** | **Medium** (Solver harness needed) |
| **Anti-Procrastination Alignment** | **Optimal** (Reduces cognitive friction) | **Poor** (Frustrates user trust) | **Overcomplicated** |

### References & Links

* Wayfinding Ticket: [#10 Design domain core and scheduling engine algorithms](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/10)
* Parent Architecture Variant: [0001-architecture-variant-selection.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0001-architecture-variant-selection.md)
* Component & Seam Architecture: [0004-component-architecture-and-module-interfaces.md](file:///c:/Users/Jakob/Documents/Master%20SE/Paise_ProjektWS26/docs/adr/0004-component-architecture-and-module-interfaces.md)
