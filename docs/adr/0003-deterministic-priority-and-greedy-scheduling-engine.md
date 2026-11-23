---
status: "accepted"
date: 2026-11-20
last-updated: 2026-11-21
decision-makers: "Jakob Sapototzky"
---

# Use Deterministic Priority Scoring and Greedy First-Fit Daily Scheduling Engine

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project ([phase_2_architecture.md](../../ai_docs/phases/phase_2_architecture.md)) and in alignment with our client-side architecture ([ADR-0001](0001-architecture-variant-selection.md), [ADR-0002](0002-client-side-persistence-strategy-and-schema.md)), we must design the algorithmic core of the system:

1. **Deterministic Priority Calculation (FR-DP-02, FR-DP-03, FR-DR-01):** Transparently and deterministically rank uncompleted tasks based on deadlines and importance ratings, providing sensible neutral fallbacks for missing attributes.
2. **Automated Daily Schedule Generation (FR-DP-04, FR-DP-05, FR-DR-02):** Automatically assign uncompleted tasks into user-defined, non-overlapping availability windows for a given date without splitting tasks across windows or days.
3. **Domain Purity & Testability:** Ensure algorithms execute as pure, zero-dependency TypeScript modules capable of 100% deterministic test coverage with zero external solver bloat.

We must define the mathematical formulas, time normalization rules, tie-breaking heuristics, and algorithmic allocation strategy.

## Decision Drivers

* **Anti-Procrastination & Cognitive Explainability (NFR-01):** Task priority scoring and schedule generation must be transparent, deterministic, and intuitive. Users must readily comprehend why a task is placed in a given slot, avoiding the skepticism and resistance caused by opaque "black-box" optimization.
* **Strict Non-Overlapping & Atomic Placement (FR-DP-05):** Tasks must be allocated as complete, contiguous blocks without splitting across windows. Overlapping task slots are strictly prohibited.
* **Robust Edge-Case Resilience:** Deterministic, predictable handling of overdue tasks, missing deadlines, missing importance ratings, overlapping user availability windows, and day capacity overflows.
* **Sub-Millisecond Execution & Zero Worker Overhead:** Algorithms must run synchronously on the browser main thread in less than 5 milliseconds for typical backlogs (N <= 100), avoiding Web Worker communication complexity.
* **Domain Purity & Zero Dependencies:** Core logic must be isolated from browser DOM, React APIs, or third-party libraries (e.g., date helpers or constraint solvers). Testability is achieved through explicit dependency injection of the current time reference (`now: Date`).

## Considered Options

* Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring
* Option 2: Optimal 0/1 Knapsack / Exact Dynamic Programming Packing
* Option 3: External Constraint Satisfaction Problem (CSP) Solver / Backtracking Search Library

## Decision Outcome

Chosen option: "Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring", because it guarantees predictable, explainable scheduling where high-priority tasks are always placed first (avoiding the counter-intuitive "knapsack packing anomaly"), runs in O(N log N) time with sub-millisecond latency, and requires zero external solver dependencies.

### Consequences

* Good, because users can immediately verify and trust the schedule order, directly lowering cognitive fatigue and anti-procrastination friction (NFR-01).
* Good, because the composite scoring formula (Urgency 10–60 + Importance 10–40 -> Score 20–100) transparently elevates approaching and overdue tasks while respecting user-assigned importance.
* Good, because a strict 5-tier deterministic tie-breaker eliminates sorting instability and non-deterministic schedule jitter.
* Good, because the domain logic is written in pure TypeScript with zero external dependencies, allowing 100% branch and edge-case test coverage with deterministic fixtures.
* Bad, because greedy first-fit may leave small residual gaps in windows if the next priority task is longer than the remaining gap, even if a lower-priority shorter task exists (accepted intentionally to maintain strict priority precedence).
* Neutral, because multi-day horizon optimization and subtask dependency graph resolution are deferred to post-MVP releases.

### Confirmation

Compliance with this decision is confirmed through:

1. **Automated Unit Test Coverage:** The Vitest test suite achieves 100% statement and branch coverage on `calculatePriorityBreakdown`, `compareTasksByPriority`, and `generateDailySchedule`.
2. **Boundary Condition Tests:** Explicit test assertions verify exact priority scores across all urgency thresholds (delta t = -1ms overdue, delta t = 0h, delta t = 24h, delta t = 48h, delta t = 7d, and undefined deadline).
3. **Schedule Invariant Assertions:** Property tests verify that no generated schedule ever contains overlapping task allocations, tasks exceeding window boundaries, or tasks split across disjoint windows.

## Pros and Cons of the Options

### Option 1: Greedy Atomic First-Fit Allocation with Piecewise Urgency Scoring

Sort candidate tasks deterministically by priority score and tie-breakers. Normalize availability windows, then place each task sequentially into the earliest window with sufficient contiguous remaining capacity.

* Good, because it is fully deterministic, highly intuitive, and transparently aligns with user expectations (most critical tasks receive earliest slots).
* Good, because time complexity is O(N log N) for sorting plus O(N x W) for placement, executing in < 1 ms on modern browsers.
* Good, because it requires zero external dependencies, keeping the bundle lean.
* Neutral, because it does not optimize for 100% theoretical bin packing density, which is desirable in task scheduling to prevent priority inversion.
* Bad, because small window fragments may remain unused if priority tasks have long durations.

### Option 2: Optimal 0/1 Knapsack / Exact Dynamic Programming Packing

Formulate schedule generation as a multi-capacity knapsack optimization problem, maximizing total scheduled priority points or total scheduled minutes.

* Good, because it achieves mathematically optimal utilization of available window minutes.
* Neutral, because standard 0/1 knapsack dynamic programming can be implemented without large libraries.
* Bad, because it causes the "Knapsack Packing Anomaly": a low-priority task is scheduled ahead of a critical urgent task simply because its duration fits into an exact remaining gap.
* Bad, because counter-intuitive task ordering destroys user trust, causing frustration and procrastination resistance.
* Bad, because dynamic programming over fine-grained minute grids (1440 minutes) incurs higher memory and CPU overhead.

### Option 3: External Constraint Satisfaction Problem (CSP) Solver / Backtracking Search Library

Integrate an external constraint programming solver (e.g., `js-csp`, `constraint-solver`) with backtracking search to allocate tasks to slots.

* Good, because it easily accommodates complex multi-variable constraints (e.g., dependency chains, multi-day scheduling, resource conflicts).
* Bad, because it adds substantial external bundle bloat (~50–150 KB) for simple single-day scheduling.
* Bad, because backtracking search exhibits exponential worst-case runtime (O(2^N)), creating risk of UI thread freezing.
* Bad, because it introduces unnecessary architectural complexity, violating the "Keep it Simple" requirement of Phase 2.

## More Information

### Detailed Algorithm & Formula Specifications

#### 1. Deterministic Priority Calculation (FR-DP-02, FR-DP-03)

The priority score is a composite integer ranging from **20 to 100**, where higher numbers denote higher priority:

`PriorityScore = UrgencyScore (10..60) + ImportanceScore (10..40)`

##### A. Importance Score Mapping (10 to 40)

Derived from user-assigned `importance` (FR-TM-04):

* `HIGH`: **40 points**
* `MEDIUM`: **25 points** (default fallback when importance is omitted)
* `LOW`: **10 points**

##### B. Urgency Score Mapping (10 to 60)

Urgency is derived from the time delta between the task deadline and the reference timestamp:
`delta t = deadlineMs - nowMs`

The score is calculated via discrete piecewise thresholds:

* **Overdue** (`delta t < 0`): **60 points** (maximum urgency score; overdue tasks require immediate attention)
* **Critical** (`0 <= delta t <= 24 hours`): **50 points**
* **Urgent** (`24 hours < delta t <= 48 hours`): **40 points**
* **Upcoming** (`2 days < delta t <= 7 days`): **25 points**
* **Distant** (`delta t > 7 days`): **10 points**
* **No Deadline** (Omitted / Undefined): **10 points** (neutral baseline matching distant deadlines; allows importance to drive ranking)

##### C. Deterministic 5-Tier Tie-Breaker

When multiple tasks share identical composite scores, ties are resolved deterministically using the following strict hierarchy:

1. **Priority Score** (Descending: higher score first)
2. **Earliest Deadline** (Ascending: tasks with deadlines precede tasks without; earlier timestamp wins)
3. **Shortest Duration** (Ascending: Shortest Job First heuristic reduces backlog count)
4. **Creation Timestamp** (`createdAt` Ascending: oldest tasks first to prevent starvation)
5. **Lexicographical Task ID** (Ascending: guarantees absolute, stable ordering)

#### 2. Window Normalization & Time Arithmetic

To prevent date/timezone parsing bugs and floating-point errors:

1. **Integer Minutes Representation:** Times of day are stored and processed as integer minutes from midnight (`0 <= minutes <= 1439`):
   * `toMinutes(HH, mm) = HH * 60 + mm`
   * `toTimeString(mins) = padStart(Math.floor(mins / 60), 2, '0') + ':' + padStart(mins % 60, 2, '0')`
2. **Window Validation & Normalization Pipeline:**
   Before running slot allocation, raw `AvailabilityWindowRecord[]` entries are processed:
   * **Validation:** Drop any window where `endTimeMinutes <= startTimeMinutes`.
   * **Sorting:** Sort remaining windows by `startTimeMinutes` ascending.
   * **Merging Overlaps:** Merge any windows that overlap or abut (`window[i].startTime <= window[i-1].endTime`).
   * **Result:** An ordered list of disjoint, contiguous availability intervals.

For full TypeScript function signatures, domain contracts, and schedule allocation algorithms, see:

* Algorithmic Core Specification: [docs/architecture.md](../architecture.md) §9
* Component & Seam Architecture: [0004-component-architecture-and-module-interfaces.md](0004-component-architecture-and-module-interfaces.md)
