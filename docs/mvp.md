# MVP Scope Specification — Smart Task Manager

## 1. Executive Summary & Objective

The **Smart Task Manager** MVP aims to validate the core anti-procrastination value proposition: reducing cognitive friction and decision fatigue through **automated, algorithm-based prioritization and structured single-day schedule generation**.

Instead of acting as a passive to-do list or an overly complex calendar suite, the MVP focuses on the **Core Planning Loop**: capturing tasks, assigning deterministic priority scores, defining daily availability windows, and automatically distributing tasks into non-overlapping schedule blocks.

---

## 2. Target Platform & Architecture

- **Platform:** Client-side Web Application (Single-Page App) running entirely in modern web browsers.
- **Persistence:** Local browser storage (`localStorage` / `IndexedDB`).
- **Backend / Network:** Zero external backend server dependencies required for MVP; all computations and state management run client-side.

---

## 3. Scope Boundaries

### 3.1 In-Scope Functional Requirements

| Group | Requirement ID | Name / Summary | Traceability | Priority |
|---|---|---|---|---|
| **Task Management** | **FR-TM-01** | Create task with non-empty title and optional description | UC-01 | Must |
| | **FR-TM-02** | Edit existing task details with save/cancel options | UC-02 | Must |
| | **FR-TM-03** | Delete task with confirmation prompt | UC-03 | Should |
| | **FR-TM-04** | Optional deadline and importance level (1–3 or Low/Med/High) | UC-04 | Must |
| | **FR-TM-05** | Positive task duration estimation in minutes | UC-05 | Should |
| **Daily Planning** | **FR-DP-01** | Define one or more availability windows for a selected day (start < end) | UC-06 | Must |
| | **FR-DP-02** | Automatic deterministic priority calculation using deadline and importance | UC-04, UC-07 | Must |
| | **FR-DP-03** | Automatic priority recalculation upon field update | UC-04, UC-07 | Must |
| | **FR-DP-04** | Generate daily schedule for a selected day using priority order & duration | UC-07 | Should |
| | **FR-DP-05** | Atomic slot allocation within windows (no overlaps; unfitting tasks remain unscheduled) | UC-06, UC-07 | Should |
| **Daily Review** | **FR-DR-01** | Display master unfinished task list ranked by priority | UC-04, UC-08 | Must |
| | **FR-DR-02** | Display chronological daily plan with start/end times and clear empty state | UC-08 | Should |
| | **FR-DR-04** | Mark task as completed, save state, and exclude from future schedule runs | UC-09 | Must |

### 3.2 In-Scope Nonfunctional Requirements

| ID | Quality Criterion | MVP Specification |
|---|---|---|
| **NFR-01** | **Usability** | The first unfinished scheduled task and its planned start time must be identifiable immediately from the daily view without opening task details (combating procrastination). |
| **NFR-02** | **Accessibility** | Task states (scheduled, unscheduled, completed) must be clearly distinguishable via text labels or distinct icons, not relying on color alone. |
| **NFR-03** | **Data Persistence** | All created tasks, completion states, availability windows, and generated schedules remain intact upon page reload or browser restart. |
| **NFR-04** | **Data Integrity** | If a storage operation fails, the system displays an informative error message and preserves the last valid state without corruption. |

---

## 4. Key Behavioral & Algorithmic Decisions

### 4.1 Planning Horizon & Date Handling (Compromise Solution)
- **Date-by-Date Planning Ahead:** Users can navigate to any calendar date (defaulting to today), define availability windows for that date, and generate that specific day's schedule.
- **Single-Day Task Binding:** Each scheduled task is assigned to exactly one specific date. Once scheduled on Date *X*, a task is reserved and excluded from scheduling on other dates unless unscheduled or rescheduled.

### 4.2 Scheduling Allocation Strategy
- **Atomic Slot Allocation (No Splitting):** Tasks are scheduled as complete, contiguous blocks using a greedy first-fit strategy into the earliest compatible availability window.
- **Handling Overflow:** If a task's duration exceeds the remaining capacity of available windows, it is not split; it remains visible in the Unscheduled list for that day.
- **Schedule Regeneration:** Re-running schedule generation clears and recalculates the plan for the selected day based on currently unfinished, unallocated tasks.

### 4.3 Deterministic Priority Calculation
- A transparent, rule-based formula combining:
  1. **Urgency:** Time remaining until the deadline (days/hours).
  2. **Importance:** Discrete scale (e.g., Low / Medium / High).
- **Default Fallbacks:** When deadline or importance is omitted, neutral default values are applied to allow seamless scheduling of unranked tasks.

---

## 5. Explicit Out-of-Scope (Non-Goals for MVP)

The following capabilities are explicitly deferred to post-MVP releases:

1. **Daily Progress Metrics (FR-DR-03):** Completed vs. remaining task counts, productivity analytics, streaks, and charts.
2. **Multi-Day Range Batch Scheduling:** Generating schedules across multi-day blocks (e.g., 7-day rolling window) in a single batch.
3. **Task Splitting & Chunking:** Dividing long tasks across multiple windows or days.
4. **Subtasks & Dependencies:** Hierarchical task breakdowns or prerequisite dependencies.
5. **User Authentication & Cloud Synchronization:** Multi-user accounts, login, remote database sync, and multi-device support.
6. **External Calendar Integrations:** Syncing with Google Calendar, Outlook, or Apple iCal.
7. **Reminders & Push Notifications:** Browser push notifications, sound alerts, or countdown timers.
8. **Manual Drag-and-Drop Slot Editing:** Freeform visual dragging of time slots on a canvas.
9. **AI / LLM Semantic Scoring:** Dynamic LLM evaluation of task descriptions or automatic duration guessing.

---

## 6. Definition of Done (MVP Acceptance Criteria)

The MVP is complete and ready for evaluation when:
1. A user can create, view, edit, and delete tasks with optional deadlines, importance, and estimated durations.
2. Unfinished tasks are automatically ranked according to the deterministic priority algorithm.
3. A user can select any target date, define one or more valid availability windows, and generate a non-overlapping daily schedule.
4. Tasks that cannot fit into the windows remain clearly listed as unscheduled.
5. The daily plan prominently shows the next scheduled task and its start time without requiring clicks.
6. Tasks can be marked as completed, updating their status and excluding them from future schedule runs.
7. All data persists reliably across browser refreshes and session restarts.
