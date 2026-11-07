# Requirements

## Functional Requirements

### 1. Task Management

| ID | Requirement | Source |
|---|---|---|
| FR-TM-01 | The app shall allow users to create a task with a non-empty title and an optional description. A saved task shall appear in the task list. | UC-01 |
| FR-TM-02 | The app shall display the task’s current values when editing. Saving valid changes shall update the task; cancelling shall leave it unchanged. | UC-02 |
| FR-TM-03 | The app shall request confirmation before deleting a task. If the user confirms, the app shall remove the task from the task list and daily schedule. If the user cancels the deletion in the confirmation prompt, the app shall abort the deletion and leave the task unchanged. | UC-03 |
| FR-TM-04 | The app shall allow users to add, change, or remove a deadline and an importance level independently when creating or editing a task. Neither field shall be required to create or schedule a task. | UC-04 |
| FR-TM-05 | The app shall allow users to save an estimated task duration in minutes and shall reject zero or negative durations. | UC-05 |

### 2. Daily Planning

| ID | Requirement | Source |
|---|---|---|
| FR-DP-01 | The app shall allow users to save one or more availability windows for a selected day. Each window shall have a start and end time, and the end time must be later than the start time. | UC-06 |
| FR-DP-02 | The app shall automatically calculate priority for unfinished tasks using the deadline and importance level when provided. It shall apply defined defaults for missing values. | UC-04, UC-07 |
| FR-DP-03 | The app shall recalculate a task’s priority when its deadline or importance level is added, changed, or removed. The resulting priority shall be used when generating the daily schedule. | UC-04, UC-07 |
| FR-DP-04 | The app shall allow users to generate a daily schedule. It shall consider unfinished tasks in priority order and use their estimated durations to allocate time. | UC-07 |
| FR-DP-05 | Each scheduled task shall have a start and end time within a saved availability window. Scheduled tasks shall not overlap. Tasks that do not fit shall remain visible as unscheduled. | UC-06, UC-07 |

### 3. Daily Review

| ID | Requirement | Source |
|---|---|---|
| FR-DR-01 | The app shall display scheduled tasks in chronological order with their title, start time, and end time. It shall display unscheduled tasks separately and show a clear message when the selected day has no scheduled tasks. | UC-08 |
| FR-DR-02 | The app shall show how many scheduled tasks are completed and how many remain for the selected day. Marking a scheduled task as completed shall update these totals. If no tasks are scheduled, both totals shall be zero. | UC-10 |
| FR-DR-03 | The app shall allow users to mark an unfinished task as completed. The completed status shall be saved, the task details shall remain accessible, and the task shall be excluded from future schedule generation. | UC-09 |

## Nonfunctional Requirements

These are proposed quality criteria for review.

| ID | Quality | Proposed Requirement |
|---|---|---|
| NFR-01 | Usability | A user shall be able to identify the first unfinished scheduled task and its planned start time from the daily plan without opening individual task details. |
| NFR-02 | Accessibility | Scheduled, unscheduled, and completed states shall be distinguishable without relying on colour alone. |
| NFR-03 | Data reliability | After reopening the app, successfully saved task details, completion states, and availability windows shall still be available. |
| NFR-04 | Data integrity | If saving a change fails, the app shall inform the user and preserve the last successfully saved version. It shall not present the failed change as saved. |

**Open decisions:** Define the importance levels, how deadlines affect priority, the defaults for missing values, and how ties are resolved. Decide whether a task needs a duration before it can be scheduled and whether a deadline is a hard scheduling constraint or only an input to priority.

## UC-01 — Create a Task

**User Story:** As a user I want to create a task because I need to record responsibilities before planning my day.

**Short Description:** Users capture personal, academic, or professional tasks in one place.

**Acceptance Criteria:**

- A task can be created with a non-empty title.
- A description can be added optionally.
- The saved task appears in the task list.

## UC-02 — Edit a Task

**User Story:** As a user I want to edit a task because its details or requirements may change.

**Short Description:** Users update existing tasks to keep their planning information accurate.

**Acceptance Criteria:**

- The current task details are displayed when editing.
- Valid changes are saved and displayed.
- Cancelling leaves the task unchanged.

## UC-03 — Delete a Task

**User Story:** As a user I want to delete a task because responsibilities that no longer apply should not affect my planning.

**Short Description:** Users remove unnecessary tasks from their task list and daily schedule.

**Acceptance Criteria:**

- Deletion requires confirmation.
- A deleted task disappears from the task list and schedule.
- Cancelling the confirmation preserves the task.

## UC-04 — Set Optional Prioritization Fields

**User Story:** As a user I want to optionally set a deadline and an importance level because these details help the app determine a task's priority.

**Short Description:** Users can optionally provide a deadline and an importance level. The algorithm uses these fields to calculate task priority. Tasks without these details can still be created and scheduled.

**Acceptance Criteria:**

- Deadline and importance are optional when creating or editing a task.
- Both fields can be added, changed, or removed independently.
- The algorithm considers provided values when calculating priority.
- Missing values are handled using defined defaults.
- Changing either field triggers an updated priority calculation.

## UC-05 — Estimate Task Duration

**User Story:** As a user I want to estimate a task's duration because the daily schedule should reflect the time required to complete it.

**Short Description:** Users provide an estimated effort that supports realistic allocation of tasks throughout the day.

**Acceptance Criteria:**

- A positive duration in minutes can be saved.
- Zero and negative values are rejected.
- Scheduling uses the saved estimate when allocating time.

## UC-06 — Define Available Time

**User Story:** As a busy individual I want to specify my available time because task planning must fit around my existing responsibilities.

**Short Description:** Users define the time windows in which the app may schedule tasks.

**Acceptance Criteria:**

- Users can save one or more availability windows for a day.
- Each window must end after it starts.
- Tasks are scheduled only within the saved windows.

## UC-07 — Generate a Daily Schedule

**User Story:** As a user I want to generate a daily schedule because a structured plan helps me turn intentions into action.

**Short Description:** The app distributes unfinished tasks across available time, considering their priority and estimated duration.

**Acceptance Criteria:**

- Scheduled tasks have a start and end time.
- Scheduled tasks do not overlap.
- Higher-priority tasks are allocated first when they fit.
- Tasks that do not fit remain visible as unscheduled.

## UC-08 — View the Daily Plan

**User Story:** As a user I want to view my daily plan because I need a clear overview of my planned activities.

**Short Description:** A chronological view presents scheduled tasks and their allocated time.

**Acceptance Criteria:**

- Tasks are displayed in chronological order.
- Each entry shows its title, start time, and end time.
- Unscheduled tasks are displayed separately.
- A day without scheduled tasks shows a clear empty state.

## UC-09 — Complete a Task

**User Story:** As a user I want to mark a task as completed because my plan should reflect the work I have finished.

**Short Description:** Users record task completion and keep their active workload up to date.

**Acceptance Criteria:**

- An unfinished task can be marked as completed.
- Its completed status is saved.
- It is excluded from future automatic scheduling.
- Its previous details remain accessible.

## UC-10 — Review Daily Progress

**User Story:** As a user I want to review my daily progress because seeing completed work helps me recognize progress toward my goals.

**Short Description:** A daily overview summarizes completed and remaining planned tasks.

**Acceptance Criteria:**

- The overview shows the number of completed and remaining tasks for the selected day.
- Completing a task updates the displayed counts.
- A day without planned tasks displays zero counts correctly.
