# Code-Level Design Patterns — GoF + SOLID

> 23 Gang of Four patterns + 5 SOLID principles.
> Each pattern has: Problem → Solution → When to Use → Code Example → Red Flag.

---

## SOLID Principles (NON-NEGOTIABLE)

<HARD-GATE>
Before applying ANY GoF pattern, check SOLID compliance first.
A pattern on top of SOLID violations = lipstick on a pig.
</HARD-GATE>

### S — Single Responsibility Principle

**Rule:** A class/module should have one, and only one, reason to change.

**Detection:**
```
grep -c "function\|def \|func " target_file
# > 8 functions in one file = SRP violation suspect
```

**Before (violation):**
```typescript
// user.service.ts — Does EVERYTHING
class UserService {
  createUser() { /* DB insert */ }
  sendWelcomeEmail() { /* SMTP logic */ }
  generateReport() { /* PDF generation */ }
  validatePassword() { /* Regex logic */ }
}
```

**After (compliant):**
```typescript
// user.service.ts — Only user CRUD
class UserService {
  constructor(private repo: UserRepository) {}
  create(data: CreateUserDTO) { return this.repo.save(data); }
}

// email.service.ts — Only email
class EmailService {
  sendWelcome(user: User) { /* SMTP */ }
}

// report.service.ts — Only reports
class ReportService {
  generateUserReport(userId: string) { /* PDF */ }
}
```

### O — Open/Closed Principle

**Rule:** Open for extension, closed for modification.

**Red Flag:** `if/else` chain that grows every time a new type is added.

**Before:**
```python
def calculate_price(order_type: str, amount: float):
    if order_type == "standard":
        return amount
    elif order_type == "premium":
        return amount * 0.9
    elif order_type == "vip":       # Added last week
        return amount * 0.8
    elif order_type == "enterprise": # Added yesterday — WHEN DOES IT END?
        return amount * 0.7
```

**After (Strategy pattern):**
```python
class PricingStrategy(Protocol):
    def calculate(self, amount: float) -> float: ...

class StandardPricing:
    def calculate(self, amount: float) -> float:
        return amount

class PremiumPricing:
    def calculate(self, amount: float) -> float:
        return amount * 0.9

# New type? Just add a new class. ZERO changes to existing code.
```

### L — Liskov Substitution Principle

**Rule:** Subtypes must be substitutable for their base types.

**Red Flag:** Subclass throws `NotImplementedError` on a parent method.

### I — Interface Segregation Principle

**Rule:** No client should be forced to depend on methods it does not use.

**Red Flag:** Interface with 10+ methods where most implementors only use 3.

### D — Dependency Inversion Principle

**Rule:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Red Flag:** `import { PrismaClient } from '@prisma/client'` directly in a service file.

---

## Creational Patterns (5)

### 1. Factory Method

**Problem:** Code uses `new ConcreteClass()` in 3+ places with conditional logic.

**When to Use:**
- Creating objects from user input or config
- Different variants of a product (EmailNotification, SMSNotification, PushNotification)

**TypeScript Example:**
```typescript
interface Notification {
  send(message: string): Promise<void>;
}

class EmailNotification implements Notification {
  async send(message: string) { /* SMTP */ }
}

class SMSNotification implements Notification {
  async send(message: string) { /* Twilio */ }
}

// Factory
function createNotification(type: "email" | "sms"): Notification {
  const map: Record<string, new () => Notification> = {
    email: EmailNotification,
    sms: SMSNotification,
  };
  return new map[type]();
}
```

**Red Flag:** Don't use Factory for 1-2 simple classes. Direct `new` is fine.

### 2. Abstract Factory

**Problem:** Families of related objects that must be used together.

**When to Use:** UI theme systems (DarkButton + DarkInput + DarkCard).

### 3. Builder

**Problem:** Constructor with 5+ parameters, many optional.

**When to Use:**
- Query builders, config objects, complex DTOs
- Objects that need step-by-step construction

**TypeScript Example:**
```typescript
class QueryBuilder {
  private query: Partial<Query> = {};

  select(fields: string[]) { this.query.fields = fields; return this; }
  from(table: string) { this.query.table = table; return this; }
  where(condition: string) { this.query.conditions ??= []; this.query.conditions.push(condition); return this; }
  limit(n: number) { this.query.limit = n; return this; }
  build(): Query { return this.query as Query; }
}

// Usage
const query = new QueryBuilder()
  .select(["id", "name"])
  .from("users")
  .where("active = true")
  .limit(10)
  .build();
```

### 4. Singleton

**Problem:** Exactly one instance needed globally.

**When to Use:** Database connection pool, logger, app config.

**⚠️ WARNING:** Singleton is the most OVERUSED pattern. Use only when:
- Resource is expensive to create (DB connection)
- Shared state is truly global (app config)

**NEVER use for:** Services, controllers, or anything with business logic.

### 5. Prototype

**Problem:** Creating objects by cloning existing ones.

**When to Use:** Deep copying complex objects, template-based creation.

---

## Structural Patterns (7)

### 6. Adapter

**Problem:** Incompatible interfaces between two existing systems.

**When to Use:** Wrapping third-party APIs, migrating from one library to another.

### 7. Decorator

**Problem:** Add behavior to objects without modifying their class.

**When to Use:** Middleware chains, logging, caching, auth checks.

**Next.js Example (Middleware decorator):**
```typescript
// Composable middleware pattern
type Middleware = (req: Request) => Promise<Response | null>;

function withAuth(next: Middleware): Middleware {
  return async (req) => {
    const token = req.headers.get("Authorization");
    if (!token) return new Response("Unauthorized", { status: 401 });
    return next(req);
  };
}

function withRateLimit(limit: number, next: Middleware): Middleware {
  const counts = new Map<string, number>();
  return async (req) => {
    const ip = req.headers.get("x-forwarded-for") ?? "unknown";
    const current = counts.get(ip) ?? 0;
    if (current >= limit) return new Response("Too Many Requests", { status: 429 });
    counts.set(ip, current + 1);
    return next(req);
  };
}
```

### 8. Facade

**Problem:** Complex subsystem with many classes/functions.

**When to Use:** Simplifying external API integrations, wrapping complex libraries.

### 9. Proxy

**Problem:** Need to control access to an object.

**When to Use:** Lazy loading, access control, caching, logging.

### 10. Composite

**Problem:** Tree structures where leaves and branches are treated uniformly.

**When to Use:** UI component trees, file systems, org charts.

### 11. Bridge

**Problem:** Abstraction and implementation should vary independently.

**When to Use:** Cross-platform rendering, multiple DB drivers.

### 12. Flyweight

**Problem:** Large number of similar objects consuming memory.

**When to Use:** Game entities, document character rendering.

---

## Behavioral Patterns (11)

### 13. Strategy

**Problem:** `if/else` or `switch` selecting different algorithms.

**When to Use:** Payment methods, sorting algorithms, validation rules.

**FastAPI Example:**
```python
from abc import ABC, abstractmethod

class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, base_price: float) -> float: ...

class RegularPricing(PricingStrategy):
    def calculate(self, base_price: float) -> float:
        return base_price

class PremiumPricing(PricingStrategy):
    def calculate(self, base_price: float) -> float:
        return base_price * 0.85

class OrderService:
    def __init__(self, pricing: PricingStrategy):
        self.pricing = pricing

    def total(self, base_price: float) -> float:
        return self.pricing.calculate(base_price)
```

### 14. Observer

**Problem:** Objects need to react when another object changes state.

**When to Use:** Event systems, real-time updates, pub/sub.

### 15. Command

**Problem:** Need to encapsulate actions as objects.

**When to Use:** Undo/redo, job queues, macro recording, transaction scripts.

### 16. Template Method

**Problem:** Algorithm has fixed steps but some steps vary.

**When to Use:** ETL pipelines, test fixtures, report generators.

### 17. Iterator

**Problem:** Traverse a collection without exposing its structure.

**When to Use:** Custom data structures, paginated API results.

### 18. State

**Problem:** Object behavior changes based on internal state (`if state == "X"`).

**When to Use:** Order status, workflow engines, UI state machines.

### 19. Chain of Responsibility

**Problem:** Multiple handlers, each decides to process or pass along.

**When to Use:** Middleware pipelines, approval workflows, event bubbling.

### 20. Mediator

**Problem:** Many objects communicate directly, creating a web of dependencies.

**When to Use:** Chat rooms, air traffic control, form validation orchestration.

### 21. Memento

**Problem:** Need to save and restore object state.

**When to Use:** Undo functionality, game save states, form drafts.

### 22. Visitor

**Problem:** Add operations to objects without modifying them.

**When to Use:** AST processing, report generation across different node types.

### 23. Interpreter

**Problem:** Need to evaluate a language or expression.

**When to Use:** DSLs, rule engines, math expression parsers.
