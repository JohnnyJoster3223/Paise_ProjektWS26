# AI Usage

---

## Project Description: Turning Bullet Points into README Content

- **Date:** October 14, 2026
- **AI tool:** ChatGPT 6: Medium
- **Purpose:** Turn the initial Smart Task Manager app notes into a project/product description of approximately 300 words or fewer.

### User Prompt (English Translation)

> I have the following task. I want to create the following app. I’ve jotted down these keywords:
>
> Smart Task Manager App  
> Task management with automatic prioritization  
> What does it do?  
> Anti-procrastination app  
> Creating custom tasks  
> Automated prioritization based on algorithms  
> Task weighting and automatic scheduling throughout the day
>
> Target Audience:  
> Combating procrastination, daily planning, time management, self-help
>
> Write me a particularly well-crafted project/product description in which you summarize the following points in a beautifully phrased summary of approximately 300 words or fewer. Focus solely on this summary; I’m not interested in any other information or questions at this time. Concentrate strictly on this first task and work with precision. Don’t get carried away!

### AI Contribution

The assistant turned the supplied app notes into a project/product description covering custom tasks, automated prioritization, daily scheduling, procrastination, and the intended audience. The description was subsequently formatted in Markdown and translated into English for the README.

## GitHub Feature Test: Branch Creation and AI Usage Documentation

- **Date:** October 14, 2026
- **AI tool:** ChatGPT (Codex), using the GitHub plugin
- **Purpose:** Test the assistant's GitHub capabilities by creating a separate branch and documenting this specific interaction in English.

### User Prompt (English Translation)

> Create a new branch. I want to test your features. In the new branch, update the AI usage documentation to reflect this exact case. Please work in English on GitHub.

### AI Contribution

The assistant inspected the repository and its existing AI usage documentation, created the branch `docs/ai-usage-github-feature-test` from `main`, and added this entry to `docs/ai-usage.md`. The documentation and commit message for this test are written in English.

## Requirements: Selected Use Cases

- **Date:** October 14, 2026
- **AI tool:** ChatGPT (Codex), using the GitHub plugin
- **Purpose:** Document the user-selected use cases in the requirements document and record the change in AI Usage.

### User Prompt (English Translation)

> Add 1–3, the new 4, 5, 8, 9, 10, 12, and 15 to the requirements document. Renumber them accordingly and record the commit in AI Usage again.

### AI Contribution

The assistant drafted 15 use cases based on the README, each with a user story, a short description under 50 words, and acceptance criteria. At the user's request, deadline and importance were combined into optional prioritization fields. The assistant transferred the ten selected use cases to `docs/requirements.md`, renumbered them UC-01 through UC-10, and added this entry to `docs/ai-usage.md`.

Both documents are included in the same commit on `docs/ai-usage-github-feature-test` with the message: `docs: add ten selected use cases and record AI contribution`.

## Requirements: Functional and Nonfunctional Specification

- **Date:** November 7, 2026
- **AI tool:** ChatGPT (Codex), using the GitHub plugin
- **Purpose:** Specify the functional and nonfunctional requirements for the Smart Task Manager and document the reviewed changes.

### User Prompt (English Translation)

> Commit these requirements to the ai-usage-github-feature-test branch. Add them to docs/requirements.md above the user stories. Make sure the user stories remain unchanged. Update docs/ai-usage.md with this contribution using the established format. Work precisely, avoid mistakes, and do not invent details.

### AI Contribution

The assistant used the README and the ten existing use cases to draft requirements, distinguishing functional requirements from proposed nonfunctional criteria and keeping unresolved decisions explicit. Based on the user's review, the assistant grouped the functional requirements under Task Management, Daily Planning, and Daily Review, aligned their IDs with those groups, and refined the requirements for deletion, priority calculation, task completion, and daily progress. The assistant inserted the reviewed requirements above the existing user stories without changing them.

Both documents are included in the same commit on `docs/ai-usage-github-feature-test` with the message: `docs: add reviewed requirements and record AI contribution`.

## Requirements: Prioritization and Daily Review

- **Date:** November 7, 2026
- **AI tool:** ChatGPT (Codex), using the GitHub plugin
- **Purpose:** Apply the reviewed Must, Should, and Could priorities and distinguish the ranked task list from the daily plan.

### User Prompt (English Translation)

> Now add all of that to `docs/requirements.md`. Remove the Open Decisions paragraph and add a heading for each of the two sections. Document this request in `docs/ai-usage.md` using the established format, and translate my prompt into English as well.

### AI Contribution

The assistant reviewed all functional and nonfunctional requirements and proposed Must, Should, or Could priorities. The user set FR-TM-05, FR-DP-04, and FR-DP-05 to Should and requested separate requirements for the ranked list of unfinished tasks (Must) and the selected day's plan (Should). The assistant applied the reviewed priorities, included importance and deadlines in the ranking requirement, renumbered the Daily Review requirements, and renamed the quality label for NFR-03 to Data-Persistence. The assistant removed the Open Decisions paragraph and grouped the document under Requirements Specification and User Stories, preserving the user story text.

Both documents are included in the same commit on `docs/ai-usage-github-feature-test` with the message: `docs: prioritize reviewed requirements and record AI contribution`.

## MVP Scope Definition

- **Date:** November 7, 2026
- **AI tool:** Antigravity (Google DeepMind)
- **Purpose:** Interactively elicit, define, and document the MVP scope for the Smart Task Manager based on existing requirements and user stories.

### User Prompt (English Translation)

> I have requirements and user stories for a task management app. Now the next step is to define an mvp scope; can you suggest one? The final scope will go into mvp.md

### AI Contribution

The assistant conducted an interactive interview to systematically resolve the key architectural, functional, and behavioral boundaries for the MVP. Decisions included establishing the Core Planning Loop (FR-TM-01–05, FR-DP-01–05, FR-DR-01, 02, 04), selecting a client-side web application with local persistence, implementing a deterministic priority formula, adopting atomic non-splitting schedule allocation, enabling date-by-date forward planning while keeping multi-day batch scheduling out of scope, and defining explicit non-goals and acceptance criteria. The assistant documented the agreed scope in `docs/mvp.md` and added this entry to `docs/ai-usage.md`.

Both documents are included in the same commit on `docs/ai-usage-github-feature-test` with the message: `docs: define MVP scope and record AI contribution`.


