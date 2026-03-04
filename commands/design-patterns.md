---
description: Analyze codebase architecture and apply professional design patterns (GoF, SOLID, Clean Architecture, DDD, CQRS, Microservices)
---

# /design-patterns Workflow

## Overview

Scan existing code → Identify architectural debt → Propose optimal patterns → Generate production-ready refactored code.

**Core process:** Detect → Diagnose → Design → Deliver

## Steps

### 1. Receive Target

The user specifies a module, file, or feature to analyze:
```
/design-patterns src/api/users
/design-patterns "the order processing flow"
/design-patterns (entire project)
```

### 2. DETECT — Scan & Map (MANDATORY)

Load the skill from `skills/design-patterns/SKILL.md` and perform:
- Recursive file tree scan
- Import graph mapping
- Function signature analysis
- File size analysis
- Generate "Heat Map" of architectural debt

### 3. DIAGNOSE — Identify Anti-Patterns

For each hotspot, classify using the anti-pattern table in SKILL.md:
- God Object, Spaghetti Code, Logic Leakage, Shotgun Surgery, Feature Envy, Primitive Obsession, Tight Coupling

### 4. DESIGN — Select Pattern

Use the 3-scale Pattern Selection Matrix:
- **Scale 1 (Code):** Refer to `references/code-patterns.md`
- **Scale 2 (Architecture):** Refer to `references/architecture-patterns.md`
- **Scale 3 (System):** Refer to `references/system-patterns.md`
- **Framework-specific:** Refer to `references/framework-catalog.md`

### 5. DELIVER — Generate Code

Produce:
- New file structure
- Production-ready boilerplate code (framework-specific)
- Step-by-step migration instructions
- Test templates
- Rollback plan

### 6. Present Results

Output using the standard format from SKILL.md:
- Architecture Health Score (/20)
- Detected Anti-Patterns table
- Recommended Patterns table
- Implementation Plan
- Generated Code
- Breaking Changes
- Rollback Plan
