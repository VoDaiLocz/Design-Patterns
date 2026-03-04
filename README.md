# Design Patterns ⚡

> An agentic AI skill that scans your codebase, detects architectural debt, and generates production-ready refactored code using professional design patterns.

## How It Works

When you invoke `/design-patterns`, it doesn't lecture you about theory. It runs a **4-phase process** on your actual code:

```
DETECT → DIAGNOSE → DESIGN → DELIVER
```

1. **Detect** — Scans your file tree, maps imports, scores complexity, generates a "Heat Map" of architectural debt.
2. **Diagnose** — Classifies anti-patterns (God Object, Logic Leakage, Tight Coupling) with file:line evidence.
3. **Design** — Selects the optimal pattern from a 3-scale matrix (Code → Architecture → System).
4. **Deliver** — Generates production-ready boilerplate code specific to your framework.

Every recommendation comes with evidence. No pattern without pain.

## The Iron Laws

```
1. NO PATTERN WITHOUT PAIN — Don't apply Factory just because textbooks say so.
2. EVIDENCE BEFORE ELEGANCE — If current code works and is testable: LEAVE IT ALONE.
3. INCREMENTAL OVER REVOLUTIONARY — One pattern, one module, one PR.
```

## Requirements

This skill works with any of these AI coding tools:

| Platform | Minimum Version |
|----------|----------------|
| Claude Code (Anthropic) | Any version supporting plugins |
| Cursor | Any version supporting `.cursor` rules |
| Codex (OpenAI) | Any version supporting instruction URLs |
| OpenCode | Any version supporting `.opencode` |

No dependencies to install. This is a pure prompt/instructions skill.

## Installation

> Install once. Works across Claude Code, Cursor, Codex, and OpenCode.

### Claude Code

```bash
/plugin marketplace add VoDaiLocz/design-patterns
/plugin install design-patterns@design-patterns
```

### Cursor

Copy the skill file into your project or global rules directory:

```bash
# Option A: Project-level (applies only to this project)
cp -r skills/design-patterns/ .cursor/rules/design-patterns/

# Option B: Clone and reference in Cursor settings → Rules → Add rule file
git clone https://github.com/VoDaiLocz/Design-Patterns.git
# Then add: skills/design-patterns/SKILL.md as a rule
```

### Codex

```
Fetch and follow instructions from https://raw.githubusercontent.com/VoDaiLocz/Design-Patterns/main/.codex/INSTALL.md
```

### OpenCode

```
Fetch and follow instructions from https://raw.githubusercontent.com/VoDaiLocz/Design-Patterns/main/.opencode/INSTALL.md
```

### Manual (Any AI tool with file context)

```bash
# Copy the skill folder anywhere your AI tool reads context from
cp -r skills/design-patterns/ ~/.gemini/antigravity/skills/design-patterns/
# Or attach SKILL.md directly as a context file in your AI tool
```

## Usage

```
/design-patterns <target>
```

**Examples:**
```
/design-patterns src/api/users         → Analyze specific module
/design-patterns "order processing"    → Analyze a feature
/design-patterns                       → Analyze entire project
```

## Example Output

**Input:** `/design-patterns src/api/routes.ts`

```markdown
## 📊 Architecture Health Score: 7/20

## 🔍 Detected Anti-Patterns
| # | Anti-Pattern      | File:Line           | Severity | Evidence                              |
|---|-------------------|---------------------|----------|---------------------------------------|
| 1 | Logic Leakage     | src/api/routes.ts:45| 🔴 High  | DB query inside route handler         |
| 2 | God Object        | src/api/routes.ts   | 🔴 High  | 450 lines, 12 functions, mixed logic  |
| 3 | Tight Coupling    | src/api/routes.ts:12| 🟡 Med   | Direct PrismaClient import            |

## 💡 Recommended Patterns
| Anti-Pattern   | → Pattern        | Scale        | Impact    |
|---------------|------------------|--------------|-----------|
| Logic Leakage | Service Layer    | Architecture | 3 files   |
| God Object    | SRP + Strategy   | Code         | 4 files   |
| Tight Coupling| Repository       | Architecture | 2 files   |

## 🏗️ Implementation Plan
Step 1: Create UserService, extract business logic from routes.ts
Step 2: Create UserRepository, move all Prisma calls
Step 3: Create OrderService + OrderRepository
Step 4: Thin routes.ts to HTTP-only controller

## 📝 Generated Code
[Framework-specific boilerplate for Next.js/FastAPI/NestJS...]
```

## What's Inside

### 3-Scale Pattern Coverage

| Scale | Patterns | Reference |
|-------|----------|-----------|
| 🔬 **Code-Level** | 23 GoF Patterns + 5 SOLID Principles | `references/code-patterns.md` |
| 🏗️ **Architecture** | Clean, Hexagonal, DDD, CQRS, Service Layer, Repository | `references/architecture-patterns.md` |
| 🌐 **System** | Microservices, Event-Driven, Circuit Breaker, Saga, API Gateway | `references/system-patterns.md` |

### Additional Capabilities (v1.1.0)

| Capability | Reference |
|------------|----------|
| 🧪 **Testing Patterns** — How to test every pattern | `references/testing-patterns.md` |
| 🚫 **Anti-Patterns Catalog** — 15 code smells with detection & cures | `references/anti-patterns.md` |
| 🔄 **Migration Playbook** — 5 step-by-step refactoring guides | `references/migration-playbook.md` |
| 📄 **Output Formats** — ADR, Health Check, Comparison Matrix | `references/output-format.md` |

### Framework-Specific Implementations

| Framework | Architecture | Key Patterns |
|-----------|-------------|-------------|
| **Next.js** | App Router + Service Layer + Repository | Server Components, Server Actions, Middleware Decorator |
| **FastAPI** | Router + Service + Repository via `Depends()` | DI, Strategy, Background Tasks Observer |
| **NestJS** | Modules + DDD-ready structure | Built-in DI, CQRS, Guards/Interceptors |
| **Django** | Apps + Service Layer (beyond MTV) | Fat Services over Fat Models |
| **Express** | Routes + Controllers + Services + Repositories | Middleware Chain of Responsibility |
| **Go** | cmd/internal structure + Interface-based DI | Repository interfaces, Handler pattern |

### Complexity Scoring

Every module is rated on 4 dimensions (max 20):

| Score | Status | Action |
|-------|--------|--------|
| 4-8 | ✅ Healthy | No action needed |
| 9-13 | 🟡 Moderate | Plan refactor next sprint |
| 14-17 | 🔴 High debt | Refactor before adding features |
| 18-20 | 🚨 Critical | Stop feature work, refactor NOW |

## Philosophy

- **No Pattern Without Pain** — Show the exact line where architecture fails
- **Evidence Before Elegance** — Measure complexity, don't guess
- **Incremental Over Revolutionary** — One pattern, one module, one PR
- **Testability First** — If the new pattern is harder to test, it's a BAD pattern
- **Scale Matters** — Clean Architecture for a TODO app is a war crime

## File Structure

```
.claude-plugin/
├── plugin.json              # Claude Code plugin manifest
└── marketplace.json         # Claude Code marketplace config

.cursor-plugin/
└── plugin.json              # Cursor plugin manifest

.codex/
└── INSTALL.md               # Codex installation guide

.opencode/
└── INSTALL.md               # OpenCode installation guide

commands/
└── design-patterns.md       # /design-patterns slash command

skills/design-patterns/
├── SKILL.md                 # Main skill — 4D Process, Iron Laws, scoring
└── references/
    ├── code-patterns.md          # 23 GoF + 5 SOLID (all with code examples)
    ├── architecture-patterns.md  # Clean, Hexagonal, DDD, CQRS
    ├── system-patterns.md        # Microservices, Event-Driven, Cloud
    ├── framework-catalog.md      # Next.js, FastAPI, NestJS, Django, Express, Go
    ├── testing-patterns.md       # ✨ NEW — Unit/Integration/E2E test code
    ├── anti-patterns.md          # ✨ NEW — 15 anti-patterns with detection
    ├── migration-playbook.md     # ✨ NEW — 5 migration guides
    └── output-format.md          # ✨ NEW — ADR, Reports, Health Check

RELEASE-NOTES.md
```

## Contributing

1. Fork this repository
2. Create a branch
3. Follow the skill format in `skills/design-patterns/SKILL.md`
4. Submit a PR

## License

MIT
