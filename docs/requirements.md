# Requirements

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
