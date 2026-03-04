# Output Formats & Templates

> Standardized output formats for every design-patterns analysis.
> Architecture Decision Records, Refactoring Reports, Health Scorecards.

---

## 1. Architecture Decision Record (ADR)

Use this template when proposing a pattern change:

```markdown
# ADR-[NUMBER]: [TITLE]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-X]

## Date
[YYYY-MM-DD]

## Context
[What is the issue? Why are we considering this change?]
- Current state: [describe the current architecture]
- Pain points: [list specific problems with file:line evidence]
- Scale: [current LOC, team size, request volume]

## Decision
We will adopt **[Pattern Name]** for **[specific module/feature]**.

### Rationale
| Criterion | Current | Proposed | Improvement |
|-----------|---------|----------|-------------|
| Testability | ❌ Requires full app | ✅ Unit testable | +80% test coverage |
| Coupling | 🔴 12 imports | 🟢 3 imports | -75% coupling |
| LOC per file | 🔴 450 LOC | 🟢 ~120 LOC avg | -73% file size |
| Time to add feature | 🔴 ~4 hours | 🟢 ~1 hour | -75% dev time |

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|-----------------|
| [Alternative 1] | [Reason] |
| [Alternative 2] | [Reason] |

## Consequences

### Positive
- [benefit 1]
- [benefit 2]

### Negative
- [tradeoff 1]
- [tradeoff 2]

### Risks
| Risk | Probability | Mitigation |
|------|:-----------:|------------|
| [risk 1] | Medium | [mitigation] |

## Implementation Plan
1. [step 1] — ETA: [time]
2. [step 2] — ETA: [time]

## Files Affected
- `path/to/file1.ts` — [what changes]
- `path/to/file2.ts` — [what changes]
```

---

## 2. Refactoring Report

Full output format for `/design-patterns` analysis:

```markdown
# 🏗️ Architecture Analysis Report

**Project:** [name]
**Date:** [YYYY-MM-DD]
**Analyzed by:** design-patterns skill v1.1.0

---

## 📊 Architecture Health Score: [X/20]

| Dimension | Score | Details |
|-----------|:-----:|---------|
| Size | [1-5] | Avg [X] LOC/file, Max [Y] LOC |
| Coupling | [1-5] | Avg [X] imports/file, Max [Y] |
| Cohesion | [1-5] | [X] modules with mixed concerns |
| Testability | [1-5] | [X]% unit testable |

**Verdict:** [✅ Healthy | 🟡 Moderate Debt | 🔴 High Debt | 🚨 Critical]

---

## 🗺️ Architecture Heat Map

```
🔴 HIGH:   [file] ([LOC] lines, [N] functions, [issues])
🟡 MEDIUM: [file] ([LOC] lines, [issues])
🟢 LOW:    [file] ([LOC] lines, clean)
```

---

## 🔍 Detected Anti-Patterns

| # | Anti-Pattern | File:Line | Severity | Evidence |
|---|-------------|-----------|----------|----------|
| 1 | [name] | [path:L] | 🔴/🟡/🟢 | [description] |
| 2 | [name] | [path:L] | 🔴/🟡/🟢 | [description] |

---

## 💡 Recommended Patterns

| Anti-Pattern | → Pattern | Scale | Impact | Confidence |
|-------------|-----------|-------|--------|:----------:|
| [current] | [proposed] | Code/Arch/System | [files] | [High/Med] |

---

## 🏗️ Implementation Plan

### Phase 1: [name] (ETA: [time])
**Files to create:**
- [ ] `path/to/new/file.ts`

**Files to modify:**
- [ ] `path/to/existing/file.ts` — [what changes]

**Tests to add:**
- [ ] `path/to/test/file.test.ts`

**Verification:**
```bash
npm test -- --coverage
```

### Phase 2: [name] (ETA: [time])
[...]

---

## 📝 Generated Code

### [Pattern Name] — [Module]

```typescript
// [filename]
[production-ready boilerplate]
```

---

## ⚠️ Breaking Changes

| File | What Breaks | How to Fix |
|------|-------------|------------|
| [path] | [description] | [fix] |

---

## ↩️ Rollback Plan

1. `git revert [commit-hash]`
2. Verify: `npm test`
3. Redeploy: `npm run deploy`

---

## 📚 References

- ADR: [link to ADR if created]
- Pattern docs: [link to relevant reference file]
```

---

## 3. Pattern Comparison Matrix

Use when the user needs to choose between 2+ patterns:

```markdown
## Pattern Comparison: [Pattern A] vs [Pattern B]

| Criterion | [Pattern A] | [Pattern B] |
|-----------|:-----------:|:-----------:|
| Complexity to implement | ⭐⭐ | ⭐⭐⭐⭐ |
| Learning curve | Low | High |
| Testability improvement | +50% | +90% |
| Files affected | 3 | 8 |
| Risk of regression | Low | Medium |
| Future extensibility | Medium | High |
| Team familiarity | ✅ Known | ❌ New concept |

**Recommendation:** [Pattern A/B] because [specific reason for THIS project].
```

---

## 4. Quick Health Check

Compact format for rapid assessments:

```markdown
## ⚡ Quick Health Check: [module]

| Metric | Value | Status |
|--------|-------|:------:|
| Total LOC | [N] | [✅/🟡/🔴] |
| Files | [N] | [✅/🟡/🔴] |
| Avg LOC/file | [N] | [✅/🟡/🔴] |
| Max LOC file | [path] ([N]) | [✅/🟡/🔴] |
| Coupling (avg imports) | [N] | [✅/🟡/🔴] |
| Functions > 50 LOC | [N] | [✅/🟡/🔴] |
| Files > 300 LOC | [N] | [✅/🟡/🔴] |
| Circular dependencies | [N] | [✅/🟡/🔴] |

**Top 3 Issues:**
1. [issue + file:line]
2. [issue + file:line]
3. [issue + file:line]

**Recommended Next Step:** [specific action]
```
