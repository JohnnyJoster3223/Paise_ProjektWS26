# ADR-0001: Architecture Variant Selection (Pure Client-Side SPA vs Fullstack C#)

> [!IMPORTANT]
> **Deliverable Translation Notice**:
> This document represents the internal, organic Architecture Decision Record drafted during Phase 2 wayfinding.
> Prior to final project submission, this decision **must be translated into the official deliverable format** using `docs/adr-template.md` and saved to `docs/adr/0001-architecture-variant-selection.md`.

- **Status:** Accepted (for MVP)
- **Date:** 2026-11-20
- **Decision Makers:** Development Team (`jakspt`)
- **Ticket Reference:** [[#8] Evaluate and justify architecture variants (Client-Side SPA vs Fullstack C#)](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/8)

---

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project (`ai_docs/phases/phase_2_architecture.md`), we must evaluate at least two distinct architectural approaches against the functional and non-functional requirements specified in `docs/requirements.md` and `docs/mvp.md`.

The core goal of the application is an anti-procrastination planning tool that reduces cognitive fatigue through deterministic task prioritization and automated single-day schedule generation. The requirements explicitly scope out multi-user collaboration, server-side data sync, subtask hierarchies, and external calendar integrations for the MVP.

The team has strong proficiency in TypeScript, React, and modern web tooling, as well as backend experience in C#. We must decide whether to build a pure client-side Single-Page Application (Variant A) or a client-server fullstack application featuring a C# ASP.NET Core Minimal API with a relational database (Variant B), while ensuring that selecting one does not prevent future architectural evolution.

---

## Decision Drivers

1. **Implementation Velocity & Simplicity:** Maximize feature completion within the semester timeframe by avoiding unnecessary distributed systems complexity, dual deployment pipelines, and cross-tier boilerplate.
2. **Zero Operational Overhead & Hosting Costs:** Utilize static edge hosting (`docs/hosting.md`, Issue #6) without database server maintenance, cold starts, cloud infrastructure bills, or operational uptime monitoring.
3. **Interactive Latency & Instant Scheduling (NFR-01):** Provide sub-millisecond schedule recalculations and task status updates locally in the browser to eliminate cognitive friction and procrastination.
4. **Data Privacy & Local Ownership (NFR-03, NFR-04):** Keep user schedule data local to the user's device, eliminating GDPR/privacy compliance concerns and backend authentication vectors.
5. **Architectural Extensibility & Clean Seams:** Maintain a clean boundary that permits an evolutionary migration to a backend (Variant B) in post-MVP phases without rewrites of UI components or domain algorithms.

---

## Considered Options

### Variant A: Pure Client-Side Single-Page Application (SPA)
- **Frontend Stack:** React, TypeScript, Vite, Tailwind CSS + shadcn/ui.
- **Architecture Style:** Modular Client Monolith (Clean/Layered: Domain Core, Persistence/Repository, Application Hooks, Presentation).
- **Persistence:** Browser-local storage (IndexedDB via Dexie.js or localStorage).
- **Execution:** 100% in-browser computation of deterministic priority rankings and atomic schedule allocations.

### Variant B: Client-Server Fullstack Architecture
- **Frontend Stack:** React, TypeScript, Vite (SPA).
- **Backend Stack:** C# .NET 8 / 9 ASP.NET Core Minimal API.
- **Database:** SQLite (local/embedded) or PostgreSQL (hosted).
- **Communication:** REST / JSON over HTTP.
- **Execution:** Server-side persistence and API endpoints for CRUD and schedule generation.

---

## Decision Outcome

**Chosen Option:** **Variant A (Pure Client-Side SPA)** for the MVP release.

### Justification
1. **Scope Alignment:** `docs/mvp.md` explicitly defines a single-user tool with zero external backend dependencies. Introducing a C# ASP.NET Core API and database server introduces speculative infrastructure for capabilities (multi-user auth, multi-device sync) that are explicitly out-of-scope non-goals.
2. **Zero Operational Cost & Maintenance:** Variant A can be hosted for free on Cloudflare Pages or GitHub Pages as static assets, with zero server compute or database hosting costs.
3. **Instant Responsiveness:** Zero network latency for daily plan generation, directly serving the core anti-procrastination UX goal.

---

## Architectural Consequences & Accepted Trade-offs

### Positive Consequences
- **Rapid Prototyping & Delivery:** Team focuses entirely on a single TypeScript codebase.
- **Trivially Testable Domain Core:** Priority scoring and schedule generation algorithms are implemented as pure, side-effect-free TypeScript functions testable in Vitest without network mocks or database fixtures.
- **Robust Offline Capability:** The application functions fully offline without network connectivity.

### Acknowledged Limitations & Risks (Accepted for MVP)
- **Device Lock-in:** User data is bound to a single browser profile on a specific device; no automatic sync between phone and desktop.
- **Browser Eviction / Data Fragility:** If a user clears their browser cache or uses incognito mode, local data can be lost (mitigated by adding JSON import/export in future releases).
- **Single-Client Concurrency:** No multi-user sharing or real-time concurrent edits.

---

## Evolutionary Path to Variant B (Preserving the Backend Migration)

To ensure that Variant A is not a technical dead end, the system introduces an explicit **Repository Seam** (`codebase-design`):

1. **Async Repository Interface (Ports & Adapters):**
   Persistence operations are defined behind asynchronous interfaces:
   ```typescript
   export interface TaskRepository {
     getAll(): Promise<Task[]>;
     getById(id: string): Promise<Task | null>;
     save(task: Task): Promise<void>;
     delete(id: string): Promise<void>;
   }
   ```
   For the MVP, an `IndexedDbTaskRepository` implements this interface.
2. **Painless Backend Migration:**
   When evolving to Variant B in a future release:
   - An `HttpTaskRepository` adapter is implemented that issues fetch requests to ASP.NET Core endpoints.
   - The React presentation components, custom hooks, and domain logic remain completely unchanged.
3. **Local-First / Client-Heavy Synergy:**
   Because the Domain Core is kept free of DOM/browser dependencies, scheduling logic can either continue running client-side (local-first with C# acting as sync storage) or be migrated/mirrored in C# if server-authoritative scheduling is needed.

---

## Comparison Matrix

| Evaluation Criterion | Variant A: Pure Client-Side SPA (Selected) | Variant B: Fullstack C# ASP.NET Core API |
|---|---|---|
| **Alignment with MVP Scope** | **High** (Matches single-user, local persistence goals) | **Low** (Over-engineered for single-user offline MVP) |
| **Operational & Hosting Cost** | **Zero** (Static hosting on Cloudflare/GitHub Pages) | **Medium/High** (Container hosting, DB tier) |
| **Interactive Latency** | **Sub-millisecond** (Local execution, zero network calls) | **Variable** (Dependent on HTTP latency & cold starts) |
| **Offline Support** | **Native** (Runs 100% locally) | **Requires complex sync logic** (PWA/Cache/Outbox) |
| **Multi-Device Synchronization** | **None** (Bound to local browser storage) | **Native** (Centralized database) |
| **Multi-User Collaboration** | **None** (Out of MVP scope) | **Native** (Central authentication & RBAC) |
| **Implementation Effort** | **Low-Medium** (Single language/runtime) | **High** (Two languages, EF Core migrations, API contracts) |
