---
name: design-patterns
description: >
  Full-spectrum Design Patterns skill: Detect anti-patterns in existing codebases,
  diagnose architectural debt, propose optimal patterns (GoF, SOLID, DDD, CQRS,
  Microservices, Cloud), and generate production-ready boilerplate code.
  Optimized for Next.js, FastAPI, NestJS, Django, Express, and Go.
---

# Design Patterns — Detect · Diagnose · Design · Deliver

> The complete design patterns skill. From function-level GoF to system-level
> Microservices. Not just theory — it scans your code, finds the pain, and
> generates the fix.

**Announce at start:** "I'm using the design-patterns skill to analyze your codebase architecture and apply professional design patterns."

---

## The Iron Laws

```
1. NO PATTERN WITHOUT PAIN
   — Don't apply Factory just because textbooks say so.
   — Show the EXACT line where current code breaks.

2. EVIDENCE BEFORE ELEGANCE
   — Measure complexity before proposing solutions.
   — If current code works, is testable, and is readable: LEAVE IT ALONE.

3. INCREMENTAL OVER REVOLUTIONARY
   — Never refactor 20 files in one PR.
   — One pattern, one module, one PR.
```

<HARD-GATE>
Before proposing ANY pattern:
1. You MUST have scanned the target module's file tree
2. You MUST have identified at least ONE concrete anti-pattern with file:line evidence
3. You MUST have estimated the breaking impact (how many files change)
If ANY of these is missing → STOP. Go back to scanning.
</HARD-GATE>

---

## When to Trigger

| Signal in Codebase | Severity | Pattern Category |
|--------------------|----------|-----------------|
| Function > 100 lines | 🟡 Medium | Code-Level (GoF) |
| Class doing 5+ unrelated things | 🔴 High | SOLID violation |
| Business logic inside API route/controller | 🔴 High | Architecture (Service Layer) |
| Database queries scattered across files | 🟡 Medium | Architecture (Repository) |
| Duplicated logic in 3+ places | 🟡 Medium | Code-Level (Strategy/Template) |
| Circular imports between modules | 🔴 High | Architecture (Dependency Inversion) |
| Monolith > 50k LOC with mixed concerns | 🔴 Critical | System (Microservices/DDD) |
| No error recovery or retry logic | 🟡 Medium | System (Circuit Breaker) |
| Hardcoded config values | 🟢 Low | Code-Level (Factory/Builder) |

---

## The 4D Process: Detect → Diagnose → Design → Deliver

### Phase 1: DETECT (Scan & Map)

```dot
digraph detect {
    rankdir=LR;
    scan [label="Scan\nFile Tree", shape=box, style=filled, fillcolor="#e6ffe6"];
    imports [label="Map\nImport Graph", shape=box];
    complexity [label="Score\nComplexity", shape=box];
    hotspots [label="Identify\nHotspots", shape=box, style=filled, fillcolor="#ffe6e6"];
    scan -> imports -> complexity -> hotspots;
}
```

**What to scan:**
1. **File tree** — `list_dir` recursively, understand module boundaries
2. **Import graph** — Who imports whom? Find coupling clusters
3. **Function signatures** — Count parameters (>5 = code smell)
4. **File sizes** — Lines per file (>300 = needs splitting)
5. **Naming conventions** — Are they consistent? (camelCase vs snake_case mix = chaos)

**Output:** A "Heat Map" of architectural debt:
```
🔴 HIGH DEBT:  src/api/routes.ts (450 lines, 12 functions, mixed DB + logic)
🟡 MED DEBT:   src/utils/helpers.ts (280 lines, "God module")
🟢 LOW DEBT:   src/models/user.ts (45 lines, clean data model)
```

### Phase 2: DIAGNOSE (Identify Anti-Patterns)

For each hotspot, classify the anti-pattern:

| Anti-Pattern | Description | Evidence Required |
|-------------|-------------|-------------------|
| **God Object** | One class/module does everything | File > 300 LOC, > 8 methods |
| **Spaghetti Code** | No clear flow, tangled logic | Cyclomatic complexity > 15 |
| **Logic Leakage** | Business rules in wrong layer | DB queries in route handlers |
| **Shotgun Surgery** | One change requires editing 5+ files | Grep shows same logic in multiple files |
| **Feature Envy** | Module uses another module's data more than its own | Import analysis |
| **Primitive Obsession** | Using strings/numbers instead of domain objects | No type definitions for core concepts |
| **Tight Coupling** | Modules directly depend on implementation details | No interfaces/abstractions |

**Output:** Diagnosis table with file:line evidence.

### Phase 3: DESIGN (Select Pattern)

Use the **Pattern Selection Matrix** to choose the right fix:

#### Scale 1: Code-Level (GoF + SOLID)
See: `references/code-patterns.md`

| Problem | Pattern | When to Use |
|---------|---------|-------------|
| Multiple object creation paths | **Factory Method** | 3+ constructors or conditional `new` |
| Complex object construction | **Builder** | Object with 5+ optional params |
| One global instance needed | **Singleton** | Config, DB connection pool |
| Algorithm varies at runtime | **Strategy** | `if/else` chains selecting behavior |
| React to state changes | **Observer** | Event systems, pub/sub |
| Add behavior without modifying | **Decorator** | Middleware, logging, caching |
| Simplify complex subsystem | **Facade** | External API wrapper |
| Undo/redo operations | **Command** | Action history, job queues |
| Process in steps | **Template Method** | ETL pipelines, test fixtures |
| Tree-like structures | **Composite** | UI components, file systems |

#### Scale 2: Architecture-Level
See: `references/architecture-patterns.md`

| Problem | Pattern | When to Use |
|---------|---------|-------------|
| Logic mixed with infrastructure | **Clean Architecture** | New project or major refactor |
| External dependencies hard to swap | **Hexagonal (Ports & Adapters)** | Multiple data sources |
| Complex domain with many rules | **DDD (Domain-Driven Design)** | Enterprise, fintech, healthcare |
| Business logic in controllers | **Service Layer** | Any app with > 10 endpoints |
| DB access scattered | **Repository Pattern** | Any app with database |
| Read/Write have different needs | **CQRS** | High-traffic, analytics-heavy |

#### Scale 3: System-Level
See: `references/system-patterns.md`

| Problem | Pattern | When to Use |
|---------|---------|-------------|
| Monolith too large to deploy | **Microservices** | Team > 5, independent deploy needed |
| Services need loose coupling | **Event-Driven Architecture** | Async workflows, notifications |
| Cascading failures | **Circuit Breaker** | External API calls |
| Distributed transactions | **Saga Pattern** | Multi-service data consistency |
| Traffic spikes | **Bulkhead + Rate Limiter** | Public APIs |
| Config management across services | **Sidecar / Ambassador** | Kubernetes / containerized |

### Phase 4: DELIVER (Generate Code)

**This is what makes this skill different.** After selecting the pattern, the skill
generates production-ready boilerplate code specific to your framework.

**Output includes:**
1. **New file structure** — Where each file goes
2. **Boilerplate code** — Ready to paste, framework-specific
3. **Migration steps** — How to move old code into new structure
4. **Test template** — Test file for the new pattern
5. **Rollback plan** — How to undo if something breaks

---

## Complexity Scoring Formula

Rate each module on 4 dimensions (1-5 each, max 20):

| Dimension | 1 (Clean) | 5 (Critical) |
|-----------|-----------|---------------|
| **Size** | < 100 LOC | > 500 LOC |
| **Coupling** | 0-2 imports | > 10 imports |
| **Cohesion** | Single responsibility | 5+ unrelated functions |
| **Testability** | Unit testable | Requires full app to test |

**Score interpretation:**
- **4-8:** ✅ Healthy — No action needed
- **9-13:** 🟡 Moderate debt — Plan refactor in next sprint
- **14-17:** 🔴 High debt — Refactor before adding features
- **18-20:** 🚨 Critical — Stop feature work, refactor NOW

---

## Red Flags — STOP and Rethink

- You're adding an interface for a class used in only 1 place → Over-engineering
- You're splitting a 50-line file into 3 files → Premature abstraction
- You're using DDD for a CRUD app → Wrong scale
- You're applying Microservices to a 2-person team → Organizational mismatch
- The refactored code is harder to read than the original → Failed refactor
- You can't explain the pattern in one sentence → You don't understand it

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "We need Clean Architecture for everything" | Clean Architecture for a TODO app is a war crime |
| "Singleton is always bad" | DB connection pool is a legitimate Singleton |
| "We should use microservices" | If you can't build a monolith, you can't build microservices |
| "Interfaces everywhere for testability" | Mock the boundary, not every function |
| "This pattern is industry standard" | Industry standard for YOUR scale? YOUR team? |

---

## Output Format

Every design-patterns analysis MUST follow:

```markdown
## 📊 Architecture Health Score: [X/20]

## 🔍 Detected Anti-Patterns
| # | Anti-Pattern | File:Line | Severity | Evidence |
|---|-------------|-----------|----------|----------|
| 1 | [name]      | [path:L]  | 🔴/🟡/🟢 | [description] |

## 💡 Recommended Patterns
| Anti-Pattern | → Pattern | Scale | Impact |
|-------------|-----------|-------|--------|
| [current]   | [proposed]| Code/Arch/System | [files affected] |

## 🏗️ Implementation Plan
### Step 1: [action]
- Files to create: [list]
- Files to modify: [list]
- Tests to add: [list]

## 📝 Generated Code
[Framework-specific boilerplate]

## ⚠️ Breaking Changes
[List of files/tests that will break]

## ↩️ Rollback Plan
[How to undo safely]
```

---

## References

- `references/code-patterns.md` — 23 GoF patterns + SOLID principles with code examples
- `references/architecture-patterns.md` — Clean, Hexagonal, DDD, CQRS, Service Layer, Repository
- `references/system-patterns.md` — Microservices, Event-Driven, Circuit Breaker, Saga, Cloud patterns
- `references/framework-catalog.md` — Pattern implementations for Next.js, FastAPI, NestJS, Django, Express, Go
