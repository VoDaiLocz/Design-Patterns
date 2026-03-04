# Design Patterns Release Notes

## v1.1.0 (2026-03-04)

### Added

- **Testing Patterns** (`references/testing-patterns.md`) — Unit, integration, and E2E test code for every pattern category (GoF, Architecture, System)
- **Anti-Patterns Catalog** (`references/anti-patterns.md`) — 15 anti-patterns with detection commands, severities, and cures
- **Migration Playbook** (`references/migration-playbook.md`) — 5 step-by-step migration guides with verification checkpoints and rollback plans
- **Output Formats** (`references/output-format.md`) — ADR template, Refactoring Report, Health Check, Pattern Comparison Matrix

### Fixed

- Builder pattern `build()` method now returns a copy to prevent shared-reference mutation

---

## v1.0.0 (2026-03-04)

### Added

- **4D Process: Detect → Diagnose → Design → Deliver** — systematic codebase analysis and refactoring
- **Iron Laws** — "No Pattern Without Pain", "Evidence Before Elegance", "Incremental Over Revolutionary"
- **HARD-GATE enforcement** — mandatory codebase scan with file:line evidence before any pattern proposal
- **Complexity Scoring** — 4-dimension Health Score (/20) per module (Size, Coupling, Cohesion, Testability)
- **3-Scale Pattern Selection Matrix:**
  - **Code-Level:** 23 GoF Patterns (Creational, Structural, Behavioral) + 5 SOLID Principles
  - **Architecture-Level:** Service Layer, Repository, Clean Architecture, Hexagonal, DDD, CQRS
  - **System-Level:** Microservices, Event-Driven, Circuit Breaker, Saga, API Gateway, Bulkhead, Sidecar, Strangler Fig
- **Framework-Specific Implementations:**
  - Next.js (App Router, Server Components, Server Actions)
  - FastAPI (Depends() DI, Pydantic, Background Tasks)
  - NestJS (Modules, Guards, Interceptors, built-in CQRS)
  - Django (Service Layer beyond MTV)
  - Express (Middleware Chain of Responsibility)
  - Go (Interface-based DI, cmd/internal structure)
- **Anti-Pattern Detection** — God Object, Logic Leakage, Tight Coupling, Shotgun Surgery, Feature Envy, Primitive Obsession
- **Code Generation** — production-ready boilerplate per framework
- **Migration Paths** — step-by-step guides: Monolith → Service Layer → Clean Architecture → Microservices
- **Red Flags** — checklist to prevent over-engineering and pattern abuse
- **`/design-patterns` slash command** — works with Claude Code, Cursor, Codex, OpenCode
- **Plugin manifests** — Claude Code (`.claude-plugin/`), Cursor (`.cursor-plugin/`)
- **Install guides** — Codex (`.codex/INSTALL.md`), OpenCode (`.opencode/INSTALL.md`)
