---
status: "accepted"
date: 2026-11-20
last-updated: 2026-11-21
decision-makers: "Jakob Sapototzky"
---

# Select Pure Client-Side Single-Page Application (SPA) over Fullstack C# Backend

## Context and Problem Statement

For Phase 2 of the Smart Task Manager project ([phase_2_architecture.md](../../ai_docs/phases/phase_2_architecture.md)), we must evaluate at least two distinct architectural variants against the functional and non-functional requirements specified in [requirements.md](../requirements.md) and [mvp.md](../mvp.md).

The primary goal of the application is to serve as an anti-procrastination planning tool that minimizes cognitive fatigue through transparent, deterministic task prioritization and automated single-day schedule generation. The project requirements explicitly scope out multi-user collaboration, server-side data synchronization, subtask hierarchies, and external calendar integrations for the MVP release.

The development team possesses strong engineering proficiency in TypeScript, React, and modern web build tooling (Vite), as well as backend expertise in C# (.NET). We must determine whether the system should be implemented as a Pure Client-Side Single-Page Application (Variant A) or a Client-Server Fullstack architecture featuring a C# ASP.NET Core Minimal API with a relational database (Variant B), while ensuring that selecting one variant does not foreclose future architectural evolution.

## Decision Drivers

* **Implementation Velocity & Simplicity:** Maximize feature completion within the semester timeframe by eliminating unnecessary distributed systems complexity, dual deployment pipelines, and cross-tier network boilerplate.
* **Zero Operational Overhead & Hosting Costs:** Utilize static edge hosting ([hosting.md](../hosting.md)) via GitHub Pages without database server maintenance, cold starts, cloud infrastructure bills, or operational uptime monitoring.
* **Interactive Latency & Instant Scheduling (NFR-01):** Provide sub-millisecond schedule recalculations and task status updates locally in the browser to eliminate cognitive friction and procrastination anxiety.
* **Data Privacy & Local Ownership (NFR-03, NFR-04):** Keep user task and schedule data strictly on the user's local device, eliminating GDPR/privacy compliance concerns and backend authentication attack vectors.
* **Architectural Extensibility & Clean Seams:** Maintain a clean boundary (Ports and Adapters / Repository Seam) that permits an evolutionary migration to a backend (Variant B) in post-MVP phases without rewrites of UI components or domain algorithms.

## Considered Options

* Variant A: Pure Client-Side Single-Page Application (SPA) in React and TypeScript
* Variant B: Client-Server Fullstack Architecture with C# ASP.NET Core Minimal API and Relational Database

## Decision Outcome

Chosen option: "Variant A: Pure Client-Side Single-Page Application (SPA) in React and TypeScript", because it directly aligns with the single-user, local-persistence MVP requirements, eliminates all operational and hosting expenses, delivers sub-millisecond local scheduling latency without network round-trips, and maintains a lean single-language codebase while establishing an asynchronous Repository Seam for future backend migration.

### Consequences

* Good, because the entire team operates within a unified TypeScript and React toolchain, increasing feature delivery velocity and reducing context switching.
* Good, because priority scoring and schedule generation algorithms are implemented as pure, zero-dependency TypeScript modules testable in Vitest with 100% deterministic coverage and zero network mocks.
* Good, because the application can be hosted completely free of charge as static assets on GitHub Pages with global CDN distribution.
* Good, because user data remains 100% private and offline-capable without authentication hurdles, server maintenance, or outage risks.
* Bad, because user data is initially bound to a single browser profile on a specific device, lacking automatic multi-device cloud synchronization.
* Bad, because clearing browser storage or utilizing private browsing mode could result in data loss if the user does not export their data.
* Neutral, because multi-user sharing and concurrent collaborative editing are unsupported, though explicitly defined as out-of-scope non-goals for the MVP.

### Confirmation

Compliance with this decision is confirmed through:

1. **Architectural Seam Verification:** Code reviews and boundary rules verify that presentation components and application hooks interact with data exclusively through asynchronous repository interfaces (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`), preventing tight coupling to browser storage.
2. **Static Asset Verification:** CI/CD build scripts verify that `npm run build` generates purely static client assets (`index.html`, JavaScript, CSS) deployable to static edge CDNs without server runtimes.
3. **Automated Unit Testing:** The test suite in Vitest validates that domain logic executes purely in-memory with zero network or database dependencies.

## Pros and Cons of the Options

### Variant A: Pure Client-Side Single-Page Application (SPA) in React and TypeScript

The application is bundled into static client assets running entirely within the user's browser. State is managed locally via IndexedDB (Dexie.js), and scheduling algorithms execute synchronously on the client thread.

* Good, because it requires zero backend infrastructure, zero cloud server hosting fees, and zero operational maintenance.
* Good, because client-side execution guarantees instant, sub-millisecond schedule recalculations and offline functionality.
* Good, because domain logic and UI components share a single, strongly-typed TypeScript codebase.
* Neutral, because IndexedDB requires asynchronous repository abstractions, which adds minor initial setup but enforces clean architecture.
* Bad, because data is confined to the local browser profile unless manually exported.
* Bad, because browser storage quota policies or aggressive cache clearing could evict data if not properly handled.

### Variant B: Client-Server Fullstack Architecture with C# ASP.NET Core Minimal API and Relational Database

The frontend SPA communicates via REST/JSON APIs with a backend service developed in C# (.NET 8/9) with Entity Framework Core and SQLite or PostgreSQL.

* Good, because it provides centralized, multi-device data synchronization and effortless backup.
* Good, because it readily accommodates future multi-user collaboration, user accounts, and server-side integrations (e.g., Google Calendar sync).
* Good, because the team has prior C# backend engineering proficiency.
* Neutral, because C# Minimal APIs offer high execution performance.
* Bad, because developing, maintaining, and deploying two separate codebases (TypeScript + C#) significantly slows delivery within the semester timeline.
* Bad, because hosting a .NET runtime and managed database incurs continuous operational maintenance and cloud hosting costs.
* Bad, because every task mutation and schedule generation requires network round-trips, introducing latency and failure modes that hinder the anti-procrastination user experience.
* Bad, because building multi-user authentication and database migrations introduces speculative complexity for features explicitly excluded from the MVP scope.

## More Information

To prevent architectural lock-in, the client-side SPA introduces an asynchronous **Repository Seam** based on the Ports and Adapters (Hexagonal) pattern. Persistence interactions are abstracted behind asynchronous TypeScript interfaces (`TaskRepository`, `AvailabilityWindowRepository`, `DailyScheduleRepository`), allowing a future migration to an HTTP REST backend without modifying UI components or domain algorithms.

For complete architectural specifications, module contracts, and comparison matrices, see:

* Overall Component Architecture and Seams: [docs/architecture.md](../architecture.md) §3 and §6
* Client-Side Persistence Engine: [ADR-0002](0002-client-side-persistence-strategy-and-schema.md)
* Component Architecture & Interfaces: [ADR-0004](0004-component-architecture-and-module-interfaces.md)
* Static Hosting Evaluation: [docs/hosting.md](../hosting.md)
