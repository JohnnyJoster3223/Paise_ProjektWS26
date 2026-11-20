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

### Reflection

Overall, using AI worked really well. After adjusting the initial AI output a bit (mostly regarding vague terms), the requirements and MVP scope were to our liking. For the MVP, we used the /grill-me skill to have the AI conduct an interview with us and ask questions about the key decisions.

## Phase 2: Software Architecture & Infrastructure Specification

- **Date:** November 20, 2026
- **AI tool:** Antigravity (Google DeepMind)
- **Purpose:** Develop, evaluate, critically review, and document the comprehensive Phase 2 software architecture, hosting evaluation, Architecture Decision Records (ADRs), and C4 diagrams for the Smart Task Manager MVP under the Wayfinder issue-driven orchestration framework.

### User Prompts & Workflow (English Translation)

Key prompts and interactions throughout Phase 2 included:

1. **Wayfinder Map & Frontier Initialization (`/wayfinder`):**
   > Chart the way from the MVP scope in `docs/mvp.md` and Phase 2 requirements in `ai_docs/phases/phase_2_architecture.md` to a complete, defensible architecture specification package. Break the work down into decision tickets on the GitHub issue tracker (`wayfinder:map` #5).
2. **Research Spikes:**
   > Research static SPA hosting options for React/Vite (Cloudflare Pages, GitHub Pages, Vercel) with focus on zero costs, collaboration blockers, and HTML5 routing. Research UI component libraries (shadcn/ui, Tailwind CSS, DaisyUI, React Bootstrap) with focus on accessibility and typing.
3. **Architecture Decisions & Grilling (`/grill-me`):**
   > Interactively grill the architectural trade-offs: Client-Side SPA vs Fullstack C# ASP.NET Core; IndexedDB persistence strategy via Dexie.js vs LocalStorage/SQLite; deterministic 20–100 priority scoring and greedy atomic first-fit scheduling vs knapsack/CSP optimization; and 4-tier Modular Client Monolith component architecture with async Repository Seam.
4. **Critical Questioning of AI Proposals:**
   > Critically question and evaluate AI proposals that suggest unnecessary complexity, such as heavy Redux Toolkit/Saga state boilerplate, external constraint solvers, or distributed backend microservices.
5. **Specification Authoring & Deliverable Validation:**
   > Compile official MADR records in `docs/adr/`, comprehensive architecture specification in `docs/architecture.md`, hosting evaluation in `docs/hosting.md`, and validate all Phase 2 deliverables against the Definition of Done in `ai_docs/phases/phase_2_architecture.md` (leaving the reflection section to the team).

### AI Contribution

The assistant contributed across all stages of Phase 2:

1. **Issue-Driven Wayfinding & Planning:** Created and maintained GitHub issue `Map: Phase 2 Software Architecture` (#5) and decomposed the architectural fog of war into eleven child decision tickets (#6 through #16) with native dependency tracking, preventing premature implementation and focusing each session on resolving a single concrete architectural decision.
2. **Hosting & UI Library Research:** Conducted comprehensive comparative research documented in `docs/research/hosting-options.md` and `docs/research/ui-libraries.md`, identifying Cloudflare Pages' unmetered bandwidth and Git author neutrality as superior for student team collaboration over Vercel's Hobby commit-blocking paywall.
3. **Architecture Variant Comparison & Evaluation:** Formulated an 8-dimension comparative evaluation matrix between Variant A (Pure Client-Side SPA in React/TypeScript) and Variant B (Client-Server Fullstack with C# ASP.NET Core Minimal API). Documented the decision outcome in `docs/adr/0001-architecture-variant-selection.md`, justifying Variant A for the MVP while architecting an asynchronous Repository Seam to preserve post-MVP migration to Variant B.
4. **Persistence Strategy & Schema Design:** Designed normalized IndexedDB schemas using Dexie.js for tasks, availability windows, and daily schedules with client UUID v4 identity and declarative version migrations. Documented in `docs/adr/0002-client-side-persistence-strategy-and-schema.md`.
5. **Domain Algorithmic Core:** Developed the mathematical specifications for the deterministic composite 20–100 priority scoring formula (piecewise urgency + discrete importance + 5-tier deterministic tie-breaker) and the greedy atomic first-fit slot scheduling engine with integer-minute time normalization. Documented in `docs/adr/0003-deterministic-priority-and-greedy-scheduling-engine.md`.
6. **Component Architecture & Clean Module Interfaces:** Established the Four-Tier Modular Client Monolith (`Presentation`, `Application`, `Domain Core`, `Persistence Layer`) with clean TypeScript contracts (Repository Seam, partitioned React Contexts, pure domain functions) and viewer-friendly C4 Level 1 (System Context), C4 Level 2 (Container), and C4 Level 3 (Component) diagrams using text-only Mermaid syntax. Documented in `docs/adr/0004-component-architecture-and-module-interfaces.md` and `docs/architecture.md`.
7. **Critical Evaluation of AI Proposals:** Actively identified, challenged, and rejected three AI-suggested overengineering antipatterns: (a) global Redux Toolkit/Saga boilerplate in favor of native partitioned React Contexts; (b) external CSP/0-1 Knapsack bin-packing solvers in favor of deterministic greedy scheduling (avoiding the "Knapsack Packing Anomaly" where urgent tasks are dropped for tight fits); and (c) premature distributed micro-backends in favor of a static edge SPA with an async Repository Seam.
8. **Compilation of Official Deliverables:** Synthesized all findings into `docs/architecture.md`, `docs/hosting.md`, and official MADR records in `docs/adr/` (ADR-0001 through ADR-0004) accompanied by `docs/adr/README.md`.

### Reflection

Overall, using an AI agent through the wayfinder skill worked really well to navigate our decisions/next planning steps.
The only real downsides were that the AI output was often quite long and formulated in a complex manner, making it hard to cognitively interact with the content. We got the feeling that compared to a "human-only" approach, the process of prompting the AI, then waiting, then reading through the long output only to accept it was quite different.
