# Smart Task Manager Domain

Anti-procrastination single-day task planning application focused on deterministic prioritization and automated schedule generation.

## Language

### Planning & Tasks

**Task**:
A distinct unit of personal or professional work to be planned, defined by a title, optional description, duration, deadline, and importance level.
_Avoid_: Todo, item, job, ticket

**Importance Level**:
A user-assigned discrete rating (Low, Medium, High) indicating how critical a task is to complete.
_Avoid_: Urgency, priority score, rank

**Deterministic Priority Score**:
A normalized numerical score calculated transparently from a task's deadline and importance level used to rank tasks objectively.
_Avoid_: Dynamic score, AI priority, weight

**Availability Window**:
A contiguous interval of time on a specific calendar date during which the user is available to work on tasks.
_Avoid_: Time slot, free time, work block

**Daily Schedule**:
An ordered, non-overlapping sequence of tasks assigned to specific time ranges within a day's availability windows.
_Avoid_: Calendar, agenda, timetable

**Atomic Slot Allocation**:
The scheduling strategy that assigns a task to an availability window as a single, contiguous time block without splitting across multiple windows or days.
_Avoid_: Task slicing, partial scheduling, chunking

**Unscheduled Task**:
An unfinished task that could not fit into the available windows of a target date or has not yet been allocated.
_Avoid_: Backlog item, pending task, overflow

### Architecture & System Seams

**Repository Seam**:
The asynchronous abstraction boundary separating domain logic and state hooks from concrete storage implementations.
_Avoid_: Database layer, storage adapter, API service
