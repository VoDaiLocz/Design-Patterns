# Anti-Patterns Catalog — The Complete Guide to Code Smells

> Every pattern exists because of an anti-pattern.
> Know the disease before prescribing the cure.

---

## The Anti-Pattern Severity Scale

| Level | Name | Action | Timeline |
|:-----:|------|--------|----------|
| 🟢 | **Low** | Monitor, fix when convenient | Next sprint |
| 🟡 | **Medium** | Plan fix, don't add features on top | This sprint |
| 🔴 | **High** | Fix before adding any new code | This week |
| 🚨 | **Critical** | Stop all feature work, fix NOW | Today |

---

## Code-Level Anti-Patterns

### 1. God Object / God Module

**Severity:** 🔴 High

**Detection:**
```bash
# Find files with too many functions
find src -name "*.ts" -exec sh -c 'echo "$(grep -c "function\|=>\|async " "$1") $1"' _ {} \; | sort -rn | head -10
# Files with > 300 LOC
find src -name "*.ts" | xargs wc -l | sort -rn | head -10
```

**Symptoms:**
- File > 300 lines
- Class with > 8 methods
- Module imported by > 50% of other modules
- Hard to name (ends in "Manager", "Handler", "Helper", "Utils")

**Root Cause:** No clear boundaries. Features got added "where it was convenient."

**Cure:** Apply SRP → split into focused modules (one reason to change each)

**Example:**
```
🔴 BEFORE: utils.ts (500 LOC, 23 functions)
🟢 AFTER:  validation.ts (60 LOC), formatting.ts (40 LOC),
           date-utils.ts (50 LOC), string-utils.ts (35 LOC)
```

---

### 2. Spaghetti Code

**Severity:** 🔴 High

**Detection:**
- Cyclomatic complexity > 15 (use `complexity-report` or eslint)
- Deeply nested if/else/try/catch (> 4 levels)
- Functions > 50 lines
- No clear entry/exit points

**Symptoms:**
```typescript
// 🔴 This function does FIVE things and you can't tell where one ends
function processOrder(data: any) {
  if (data.type === "standard") {
    if (data.items.length > 0) {
      for (const item of data.items) {
        if (item.stock > 0) {
          if (item.price > 100) {
            // ... 40 more lines nested 5 levels deep
          }
        }
      }
    }
  }
}
```

**Cure:** Extract functions, use early returns, apply Strategy/Template Method

---

### 3. Shotgun Surgery

**Severity:** 🟡 Medium

**Detection:**
```bash
# Check how many files change together in recent commits
git log --oneline --name-only -20 | grep -v "^[a-f0-9]\{7\}" | sort | uniq -c | sort -rn | head -20
# Find duplicated logic across files
grep -rn "function validateEmail\|def validate_email\|validateEmail" src/ --include="*.ts" --include="*.py"
```

**Symptoms:**
- Adding one feature requires changing 5+ files
- Same validation logic copied in 3+ places
- Configuration spread across multiple files

**Cure:** Consolidate scattered logic into a single service/module

---

### 4. Feature Envy

**Severity:** 🟡 Medium

**Detection:** Module A uses Module B's data and methods more than its own.

**Symptoms:**
```typescript
// 🔴 OrderService is "envious" of UserService
class OrderService {
  createOrder(userId: string) {
    const user = this.userRepo.findById(userId);
    const discount = user.isPremium ? 0.1 : 0;     // User logic HERE?
    const loyaltyPoints = user.points * 0.01;       // User logic HERE?
    const credit = user.balance - user.pendingCharges; // User logic HERE?
    // ... all this should be in UserService
  }
}
```

**Cure:** Move logic to the module that owns the data

---

### 5. Primitive Obsession

**Severity:** 🟡 Medium

**Detection:** Core domain concepts represented as raw strings/numbers.

**Symptoms:**
```typescript
// 🔴 Everything is a string
function createUser(
  email: string,     // No validation
  phone: string,     // No format guarantee
  currency: string,  // "USD"? "usd"? "US Dollar"?
  amount: number,    // Dollars or cents?
) { ... }
```

**Cure:** Create Value Objects
```typescript
// 🟢 Type-safe domain concepts
class Email {
  constructor(private value: string) {
    if (!value.includes("@")) throw new Error("Invalid email");
  }
  toString() { return this.value; }
}

class Money {
  constructor(public amount: number, public currency: "USD" | "EUR" | "VND") {}
  add(other: Money): Money {
    if (this.currency !== other.currency) throw new Error("Currency mismatch");
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

---

### 6. Long Parameter List

**Severity:** 🟡 Medium

**Detection:** Function with > 5 parameters.

**Cure:** Introduce Parameter Object or Builder.

```typescript
// 🔴 BEFORE
function sendEmail(to: string, from: string, subject: string,
  body: string, cc?: string[], bcc?: string[],
  replyTo?: string, attachments?: File[]) { ... }

// 🟢 AFTER
interface EmailOptions {
  to: string;
  from: string;
  subject: string;
  body: string;
  cc?: string[];
  bcc?: string[];
  replyTo?: string;
  attachments?: File[];
}
function sendEmail(options: EmailOptions) { ... }
```

---

### 7. Dead Code

**Severity:** 🟢 Low

**Detection:**
```bash
# Find unused exports (TypeScript)
npx ts-unused-exports tsconfig.json
# Find unused functions
npx knip
```

**Cure:** Delete it. Version control has your back.

---

### 8. Magic Numbers / Magic Strings

**Severity:** 🟢 Low

**Detection:** Hardcoded values without explanation.

```typescript
// 🔴 What is 86400000? What is "active"?
if (Date.now() - user.lastLogin > 86400000) {
  user.status = "active";
}

// 🟢 Self-documenting
const ONE_DAY_MS = 24 * 60 * 60 * 1000;
const UserStatus = { ACTIVE: "active", INACTIVE: "inactive" } as const;

if (Date.now() - user.lastLogin > ONE_DAY_MS) {
  user.status = UserStatus.ACTIVE;
}
```

---

## Architecture-Level Anti-Patterns

### 9. Logic Leakage

**Severity:** 🔴 High

**Detection:** Business logic in the wrong layer.

```typescript
// 🔴 Business logic in API route (Next.js)
export async function POST(req: Request) {
  const body = await req.json();
  // Business rule INSIDE a route handler!
  if (body.amount > 10000) {
    const manager = await prisma.user.findFirst({ where: { role: "manager" } });
    await sendApprovalEmail(manager.email, body);
    return Response.json({ status: "pending_approval" });
  }
  const order = await prisma.order.create({ data: body });
  return Response.json(order, { status: 201 });
}
```

**Cure:** Service Layer pattern — move ALL business logic to services

---

### 10. Circular Dependencies

**Severity:** 🔴 High

**Detection:**
```bash
npx madge --circular src/
```

**Symptoms:**
```
A imports B, B imports C, C imports A → 💥
```

**Cure:** 
- Extract shared logic into a new module
- Use Dependency Inversion (depend on interfaces)
- Use Event Bus (Observer) for communication

---

### 11. Anemic Domain Model

**Severity:** 🟡 Medium

**Detection:** Entities that are just data containers with no behavior.

```typescript
// 🔴 Anemic — entity is just a data bag
class Order {
  id: string;
  items: OrderItem[];
  status: string;
  total: number;
}

// All logic is OUTSIDE the entity, scattered in services
class OrderService {
  calculateTotal(order: Order) { ... }
  canCancel(order: Order) { ... }
  applyDiscount(order: Order, discount: number) { ... }
}
```

**Cure (Rich Domain Model):**
```typescript
// 🟢 Rich — entity OWNS its behavior
class Order {
  private items: OrderItem[] = [];
  private status: OrderStatus = OrderStatus.PENDING;

  get total(): number {
    return this.items.reduce((sum, item) => sum + item.subtotal, 0);
  }

  addItem(item: OrderItem) {
    if (this.status !== OrderStatus.PENDING) throw new Error("Cannot modify");
    this.items.push(item);
  }

  cancel() {
    if (this.status === OrderStatus.SHIPPED) throw new Error("Cannot cancel");
    this.status = OrderStatus.CANCELLED;
  }
}
```

---

### 12. Big Ball of Mud

**Severity:** 🚨 Critical

**Detection:** No clear module boundaries. Everything depends on everything.

**Symptoms:**
- No folder structure convention
- Files in random locations
- Import paths cross every "boundary"
- Changing one thing breaks unrelated features

**Cure:** Start with Module Boundaries → Service Layer → Eventually Clean Architecture

---

## System-Level Anti-Patterns

### 13. Distributed Monolith

**Severity:** 🚨 Critical

**Detection:** Microservices that MUST deploy together.

**Symptoms:**
- Services share a database
- Service A can't start without Service B running
- Deploying one service requires deploying 3 others
- Shared libraries with coupled logic

**Cure:** Define clear bounded contexts, each service owns its data

---

### 14. Chatty Services

**Severity:** 🔴 High

**Detection:** Service A calls Service B 10+ times per request.

**Cure:** Batch API calls, use BFF (Backend for Frontend), aggregate data

---

### 15. No Timeout / No Retry

**Severity:** 🔴 High

**Detection:** External HTTP calls without timeout configuration.

```typescript
// 🔴 No timeout — hangs forever if service is down
const data = await fetch("https://external-api.com/data");

// 🟢 With timeout + retry
const data = await fetch("https://external-api.com/data", {
  signal: AbortSignal.timeout(5000),
});
```

**Cure:** Circuit Breaker + Retry with exponential backoff

---

## Anti-Pattern Detection Checklist

Run this checklist on any module before refactoring:

```markdown
## Pre-Refactor Checklist for: [module name]

### Code Level
- [ ] Any file > 300 LOC?
- [ ] Any function > 50 LOC?
- [ ] Any function with > 5 parameters?
- [ ] Any `if/else` chains > 4 levels deep?
- [ ] Any class with > 8 methods?
- [ ] Any duplicated logic in 3+ places?
- [ ] Any magic numbers or strings?
- [ ] Any dead/unreachable code?

### Architecture Level
- [ ] Business logic in controllers/routes?
- [ ] Database queries outside repositories?
- [ ] Circular dependencies?
- [ ] Anemic entities (data bags)?
- [ ] Hardcoded dependencies (no DI)?

### System Level
- [ ] External calls without timeout?
- [ ] No error recovery / retry logic?
- [ ] Shared database between services?
- [ ] Services that must deploy together?
```
