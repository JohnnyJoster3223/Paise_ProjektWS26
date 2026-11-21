# Architecture Decision Records (ADRs)

This directory contains the official deliverable Architecture Decision Records for the **Smart Task Manager** project, formatted strictly according to [docs/adr-template.md](../adr-template.md).

For the initial design discovery and deliberation records, see the organic drafts in `ai_docs/adr/`.

## Index of Decisions

| ADR | Title | Status | Date | Summary |
|---|---|---|---|---|
| [ADR-0001](0001-architecture-variant-selection.md) | Select Pure Client-Side Single-Page Application (SPA) over Fullstack C# Backend | **Accepted** | 2026-11-20 | Adopt pure client-side React/TypeScript SPA with an asynchronous Repository Seam for zero hosting costs and sub-millisecond latency. |
| [ADR-0002](0002-client-side-persistence-strategy-and-schema.md) | Select IndexedDB via Dexie.js for Client-Side Persistence and Schema Management | **Accepted** | 2026-11-20 | Select IndexedDB via Dexie.js for client storage, ensuring ACID transactions and automatic failure rollback (NFR-04). |
| [ADR-0003](0003-deterministic-priority-and-greedy-scheduling-engine.md) | Use Deterministic Priority Scoring and Greedy First-Fit Daily Scheduling Engine | **Accepted** | 2026-11-20 | Use transparent priority scoring (20–100) and greedy atomic first-fit scheduling as pure zero-dependency TypeScript modules. |
| [ADR-0004](0004-component-architecture-and-module-interfaces.md) | Adopt Modular Client Monolith Architecture with Four-Tier Layering and Async Repository Seam | **Accepted** | 2026-11-20 | Adopt 4-tier modular client monolith architecture with pessimistic React state updates and clean Repository Seams. |
