# Architecture Decision Records (ADRs)

This directory contains the official deliverable Architecture Decision Records for the **Smart Task Manager** project, formatted strictly according to [docs/adr-template.md](../adr-template.md).

For the initial design discovery and deliberation records, see the organic drafts in `ai_docs/adr/`.

## Index of Decisions

| ADR | Title | Status | Date | Summary |
|---|---|---|---|---|
| [ADR-0001](0001-architecture-variant-selection.md) | Select Pure Client-Side Single-Page Application (SPA) over Fullstack C# Backend | **Accepted** | 2026-11-20 | Selected a pure client-side SPA in React/TypeScript to maximize implementation velocity, achieve zero hosting costs, and provide sub-millisecond local scheduling latency without network round-trips, while establishing an asynchronous Repository Seam for future backend migration. |
| [ADR-0002](0002-client-side-persistence-strategy-and-schema.md) | Select IndexedDB via Dexie.js for Client-Side Persistence and Schema Management | **Accepted** | 2026-11-20 | Selected IndexedDB wrapped with Dexie.js for client storage, satisfying NFR-03 (data persistence) and NFR-04 (data integrity via native ACID transactions and automatic rollback on failure) behind an asynchronous Repository Seam. |
| [ADR-0003](0003-deterministic-priority-and-greedy-scheduling-engine.md) | Use Deterministic Priority Scoring and Greedy First-Fit Daily Scheduling Engine | **Accepted** | 2026-11-20 | Implemented deterministic priority scoring (Urgency 10–60 + Importance 10–40) and greedy atomic first-fit time-slot scheduling as pure, zero-dependency TypeScript modules, avoiding opaque knapsack packing anomalies and supporting NFR-01 anti-procrastination explainability. |
| [ADR-0004](0004-component-architecture-and-module-interfaces.md) | Adopt Modular Client Monolith Architecture with Four-Tier Layering and Async Repository Seam | **Accepted** | 2026-11-20 | Established a 4-tier modular client monolith (Presentation, Application, Domain, Persistence) with clean public contracts, pessimistic state updates for NFR-04 integrity, and viewer-friendly C4 Container and Component architecture diagrams. |
