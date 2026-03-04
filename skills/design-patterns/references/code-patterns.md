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

**Before (violation):**
```typescript
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  area() { return this.width * this.height; }
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // Violates LSP!
  setHeight(h: number) { this.width = this.height = h; } // Callers of Rectangle are surprised
}
```

**After (compliant):**
```typescript
interface Shape {
  area(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side * this.side; }
}

// Both are safely substitutable for Shape
function printArea(shape: Shape) {
  console.log(shape.area()); // Works for any Shape
}
```

### I — Interface Segregation Principle

**Rule:** No client should be forced to depend on methods it does not use.

**Red Flag:** Interface with 10+ methods where most implementors only use 3.

**Before (violation):**
```typescript
// 🔴 Fat interface — Worker must implement irrelevant methods
interface Worker {
  work(): void;
  eat(): void;    // Robots don't eat!
  sleep(): void;  // Robots don't sleep!
}

class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error("Robots don't eat"); }  // Forced to implement
  sleep() { throw new Error("Robots don't sleep"); } // Forced to implement
}
```

**After (compliant):**
```typescript
// 🟢 Segregated interfaces — each client gets only what it needs
interface Workable { work(): void; }
interface Eatable  { eat(): void; }
interface Sleepable { sleep(): void; }

class Human implements Workable, Eatable, Sleepable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Workable {
  work() { /* ... */ }
  // No forced empty methods
}
```

### D — Dependency Inversion Principle

**Rule:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Red Flag:** `import { PrismaClient } from '@prisma/client'` directly in a service file.

**Before (violation):**
```typescript
// 🔴 UserService is tightly coupled to Prisma
import { PrismaClient } from "@prisma/client";

class UserService {
  private prisma = new PrismaClient(); // High-level depends on low-level

  async findById(id: string) {
    return this.prisma.user.findUnique({ where: { id } });
  }
}
```

**After (compliant):**
```typescript
// 🟢 Depend on abstraction, not implementation
interface IUserRepository {
  findById(id: string): Promise<User | null>;
}

class UserService {
  constructor(private repo: IUserRepository) {} // Depends on interface

  async findById(id: string) {
    return this.repo.findById(id); // No Prisma knowledge here
  }
}

// Swap out for tests:
class InMemoryUserRepository implements IUserRepository {
  private users = new Map<string, User>();
  async findById(id: string) { return this.users.get(id) ?? null; }
}
```

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

**TypeScript Example:**
```typescript
// Abstract products
interface Button { render(): string; }
interface Input  { render(): string; }

// Concrete family: Light theme
class LightButton implements Button { render() { return "<button class='light'>"; } }
class LightInput  implements Input  { render() { return "<input class='light'>"; } }

// Concrete family: Dark theme
class DarkButton implements Button { render() { return "<button class='dark'>"; } }
class DarkInput  implements Input  { render() { return "<input class='dark'>"; } }

// Abstract factory
interface UIFactory {
  createButton(): Button;
  createInput(): Input;
}

class LightThemeFactory implements UIFactory {
  createButton() { return new LightButton(); }
  createInput()  { return new LightInput(); }
}

class DarkThemeFactory implements UIFactory {
  createButton() { return new DarkButton(); }
  createInput()  { return new DarkInput(); }
}

// Usage — swap entire theme without changing rendering logic
function renderForm(factory: UIFactory) {
  const button = factory.createButton();
  const input  = factory.createInput();
  return `${input.render()} ${button.render()}`;
}

renderForm(new LightThemeFactory()); // Light UI
renderForm(new DarkThemeFactory());  // Dark UI — zero code change
```

**Red Flag:** Abstract Factory for a single product family is overkill. Use Factory Method instead.

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

**TypeScript Example:**
```typescript
// lib/prisma.ts — Legitimate Singleton (DB connection)
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

**Python Example:**
```python
# config.py — Thread-safe Singleton
import threading

class AppConfig:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._load()
        return cls._instance

    def _load(self):
        self.db_url = os.getenv("DATABASE_URL")
        self.secret = os.getenv("SECRET_KEY")
```

### 5. Prototype

**Problem:** Creating objects by cloning existing ones.

**When to Use:** Deep copying complex objects, template-based creation.

**TypeScript Example:**
```typescript
interface Cloneable<T> {
  clone(): T;
}

class DashboardWidget implements Cloneable<DashboardWidget> {
  constructor(
    public title: string,
    public config: Record<string, unknown>,
    public filters: string[],
  ) {}

  clone(): DashboardWidget {
    return new DashboardWidget(
      this.title,
      { ...this.config },         // Shallow copy config
      [...this.filters],          // Copy array
    );
  }
}

// Usage: Create variants from a template
const template = new DashboardWidget("Sales", { period: "30d" }, ["region"]);
const clone = template.clone();
clone.title = "Sales — APAC";    // Original unchanged
```

---

## Structural Patterns (7)

### 6. Adapter

**Problem:** Incompatible interfaces between two existing systems.

**When to Use:** Wrapping third-party APIs, migrating from one library to another.

**TypeScript Example:**
```typescript
// Old payment system
class StripeGateway {
  charge(cents: number, token: string) { /* Stripe API */ }
}

// New payment interface your app expects
interface PaymentProcessor {
  pay(amount: number, currency: string, method: string): Promise<Receipt>;
}

// Adapter: makes Stripe fit your interface
class StripeAdapter implements PaymentProcessor {
  constructor(private stripe: StripeGateway) {}

  async pay(amount: number, currency: string, method: string): Promise<Receipt> {
    const cents = Math.round(amount * 100); // Stripe uses cents
    await this.stripe.charge(cents, method);
    return { amount, currency, status: "paid" };
  }
}

// Now swap Stripe for PayPal without changing app code:
// class PayPalAdapter implements PaymentProcessor { ... }
```

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

function withRateLimit(limit: number, windowMs: number, next: Middleware): Middleware {
  const windows = new Map<string, { count: number; resetAt: number }>();
  return async (req) => {
    const ip = req.headers.get("x-forwarded-for") ?? "unknown";
    const now = Date.now();
    const w = windows.get(ip);
    if (!w || w.resetAt <= now) {
      windows.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (w.count >= limit) {
      return new Response("Too Many Requests", { status: 429 });
    } else {
      w.count++;
    }
    return next(req);
  };
}
```

### 8. Facade

**Problem:** Complex subsystem with many classes/functions.

**When to Use:** Simplifying external API integrations, wrapping complex libraries.

**TypeScript Example:**
```typescript
// Without Facade: Client must know 4 subsystems
// const user = await prisma.user.create(...);
// await stripe.createCustomer(user.email);
// await sendgrid.send({ to: user.email, ... });
// await analytics.track("signup", user.id);

// With Facade: One call does everything
class OnboardingFacade {
  constructor(
    private db: UserRepository,
    private billing: BillingService,
    private email: EmailService,
    private analytics: AnalyticsService,
  ) {}

  async registerUser(data: RegisterDTO): Promise<User> {
    const user = await this.db.create(data);
    await this.billing.createCustomer(user);
    await this.email.sendWelcome(user);
    this.analytics.track("signup", { userId: user.id });
    return user;
  }
}
```

### 9. Proxy

**Problem:** Need to control access to an object.

**When to Use:** Lazy loading, access control, caching, logging.

**TypeScript Example (Caching Proxy):**
```typescript
class CachingUserRepository implements UserRepository {
  private cache = new Map<string, { user: User; expiry: number }>();

  constructor(private real: UserRepository, private ttlMs = 60_000) {}

  async findById(id: string): Promise<User | null> {
    const cached = this.cache.get(id);
    if (cached && cached.expiry > Date.now()) return cached.user;

    const user = await this.real.findById(id); // Delegate to real
    if (user) this.cache.set(id, { user, expiry: Date.now() + this.ttlMs });
    return user;
  }
}
```

### 10. Composite

**Problem:** Tree structures where leaves and branches are treated uniformly.

**When to Use:** UI component trees, file systems, org charts, permission trees.

**TypeScript Example:**
```typescript
interface FileSystemNode {
  name: string;
  getSize(): number;
}

class File implements FileSystemNode {
  constructor(public name: string, private size: number) {}
  getSize() { return this.size; }
}

class Directory implements FileSystemNode {
  private children: FileSystemNode[] = [];
  constructor(public name: string) {}

  add(node: FileSystemNode) { this.children.push(node); }
  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }
}

// Usage
const root = new Directory("src");
const lib = new Directory("lib");
lib.add(new File("utils.ts", 1200));
lib.add(new File("prisma.ts", 400));
root.add(lib);
root.add(new File("index.ts", 300));
console.log(root.getSize()); // 1900
```

### 11. Bridge

**Problem:** Abstraction and implementation should vary independently.

**When to Use:** Cross-platform rendering, multiple DB drivers.

**TypeScript Example:**
```typescript
// Abstraction
interface MessageSender {
  send(to: string, body: string): Promise<void>;
}

// Implementations
class SmsSender implements MessageSender {
  async send(to: string, body: string) { /* Twilio */ }
}
class EmailSender implements MessageSender {
  async send(to: string, body: string) { /* SendGrid */ }
}

// Bridge: Notification type × Delivery method
class OrderNotification {
  constructor(private sender: MessageSender) {}
  async notify(userId: string, orderId: string) {
    await this.sender.send(userId, `Order ${orderId} confirmed`);
  }
}

// Swap delivery without changing notification logic
new OrderNotification(new SmsSender());
new OrderNotification(new EmailSender());
```

### 12. Flyweight

**Problem:** Large number of similar objects consuming memory.

**When to Use:** Game entities, document character rendering, icon libraries.

**TypeScript Example:**
```typescript
class IconFlyweight {
  private static cache = new Map<string, IconFlyweight>();
  private constructor(public readonly svg: string) {}

  static get(name: string): IconFlyweight {
    if (!this.cache.has(name)) {
      const svg = loadSvgFromDisk(name); // Expensive I/O
      this.cache.set(name, new IconFlyweight(svg));
    }
    return this.cache.get(name)!;
  }
}

// 1000 buttons, but only 5 unique SVGs loaded
const icon1 = IconFlyweight.get("check"); // Loads from disk
const icon2 = IconFlyweight.get("check"); // Returns cached
console.log(icon1 === icon2); // true — same instance
```

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

**TypeScript Example:**
```typescript
type EventHandler<T = unknown> = (data: T) => void;

class EventBus {
  private listeners = new Map<string, Set<EventHandler>>();

  on<T>(event: string, handler: EventHandler<T>) {
    if (!this.listeners.has(event)) this.listeners.set(event, new Set());
    this.listeners.get(event)!.add(handler as EventHandler);
    return () => this.listeners.get(event)?.delete(handler as EventHandler); // Unsubscribe
  }

  emit<T>(event: string, data: T) {
    this.listeners.get(event)?.forEach(fn => fn(data));
  }
}

// Usage
const bus = new EventBus();
bus.on<{ orderId: string }>("order.placed", (data) => {
  console.log(`Send email for order ${data.orderId}`);
});
bus.on<{ orderId: string }>("order.placed", (data) => {
  console.log(`Update inventory for order ${data.orderId}`);
});
bus.emit("order.placed", { orderId: "ord-123" });
```

### 15. Command

**Problem:** Need to encapsulate actions as objects.

**When to Use:** Undo/redo, job queues, macro recording, transaction scripts.

**TypeScript Example:**
```typescript
interface Command {
  execute(): void;
  undo(): void;
}

class AddItemCommand implements Command {
  constructor(private cart: Cart, private item: CartItem) {}
  execute() { this.cart.add(this.item); }
  undo() { this.cart.remove(this.item.id); }
}

class CommandHistory {
  private history: Command[] = [];

  execute(cmd: Command) {
    cmd.execute();
    this.history.push(cmd);
  }

  undo() {
    const cmd = this.history.pop();
    cmd?.undo();
  }
}

// Usage
const history = new CommandHistory();
history.execute(new AddItemCommand(cart, { id: "1", name: "Laptop", qty: 1 }));
history.undo(); // Laptop removed
```

### 16. Template Method

**Problem:** Algorithm has fixed steps but some steps vary.

**When to Use:** ETL pipelines, test fixtures, report generators.

**Python Example:**
```python
from abc import ABC, abstractmethod

class DataPipeline(ABC):
    def run(self):
        data = self.extract()       # Step 1: fixed interface
        transformed = self.transform(data)  # Step 2: varies
        self.load(transformed)      # Step 3: fixed interface

    @abstractmethod
    def extract(self) -> list: ...
    @abstractmethod
    def transform(self, data: list) -> list: ...
    @abstractmethod
    def load(self, data: list) -> None: ...

class CsvToPostgresPipeline(DataPipeline):
    def extract(self): return read_csv("data.csv")
    def transform(self, data): return [clean(row) for row in data]
    def load(self, data): postgres.bulk_insert(data)

class ApiToS3Pipeline(DataPipeline):
    def extract(self): return fetch_api("/users")
    def transform(self, data): return [enrich(row) for row in data]
    def load(self, data): s3.upload_json(data)
```

### 17. Iterator

**Problem:** Traverse a collection without exposing its structure.

**When to Use:** Custom data structures, paginated API results.

**TypeScript Example:**
```typescript
class PaginatedIterator<T> {
  private page = 0;
  private done = false;

  constructor(private fetchPage: (page: number) => Promise<T[]>) {}

  async *[Symbol.asyncIterator]() {
    while (!this.done) {
      const items = await this.fetchPage(this.page++);
      if (items.length === 0) { this.done = true; break; }
      for (const item of items) yield item;
    }
  }
}

// Usage — iterate through ALL pages transparently
const users = new PaginatedIterator((p) => fetchUsers({ page: p, limit: 50 }));
for await (const user of users) {
  console.log(user.name); // auto-fetches next page when needed
}
```

### 18. State

**Problem:** Object behavior changes based on internal state (`if state == "X"`).

**When to Use:** Order status, workflow engines, UI state machines.

**TypeScript Example:**
```typescript
interface OrderState {
  cancel(order: Order): void;
  ship(order: Order): void;
}

class PendingState implements OrderState {
  cancel(order: Order) { order.setState(new CancelledState()); }
  ship(order: Order) { order.setState(new ShippedState()); }
}

class ShippedState implements OrderState {
  cancel() { throw new Error("Cannot cancel shipped order"); }
  ship() { throw new Error("Already shipped"); }
}

class CancelledState implements OrderState {
  cancel() { throw new Error("Already cancelled"); }
  ship() { throw new Error("Cannot ship cancelled order"); }
}

class Order {
  private state: OrderState = new PendingState();
  setState(s: OrderState) { this.state = s; }
  cancel() { this.state.cancel(this); }
  ship() { this.state.ship(this); }
}
```

### 19. Chain of Responsibility

**Problem:** Multiple handlers, each decides to process or pass along.

**When to Use:** Middleware pipelines, approval workflows, event bubbling.

**TypeScript Example (Express-style middleware):**
```typescript
type Handler = (req: Request, next: () => void) => void;

class Pipeline {
  private handlers: Handler[] = [];

  use(handler: Handler) { this.handlers.push(handler); return this; }

  execute(req: Request) {
    let i = 0;
    const next = () => {
      if (i < this.handlers.length) this.handlers[i++](req, next);
    };
    next();
  }
}

// Usage
new Pipeline()
  .use((req, next) => { console.log("Auth check"); next(); })
  .use((req, next) => { console.log("Rate limit"); next(); })
  .use((req, next) => { console.log("Handle request"); })
  .execute(request);
```

### 20. Mediator

**Problem:** Many objects communicate directly, creating a web of dependencies.

**When to Use:** Chat rooms, air traffic control, form validation orchestration.

**TypeScript Example:**
```typescript
class FormMediator {
  private fields = new Map<string, FormField>();

  register(field: FormField) {
    this.fields.set(field.name, field);
    field.setMediator(this);
  }

  notify(sender: string, event: string) {
    if (sender === "country" && event === "change") {
      const city = this.fields.get("city")!;
      city.reset();                  // Clear city when country changes
      city.loadOptionsFor(this.fields.get("country")!.value);
    }
    if (sender === "shipping" && event === "change") {
      const billing = this.fields.get("billingAddress")!;
      billing.syncWith(this.fields.get("shipping")!.value);
    }
  }
}
```

### 21. Memento

**Problem:** Need to save and restore object state.

**When to Use:** Undo functionality, game save states, form drafts.

**TypeScript Example:**
```typescript
interface EditorSnapshot {
  content: string;
  cursor: number;
  timestamp: number;
}

class TextEditor {
  content = "";
  cursor = 0;

  save(): EditorSnapshot {
    return { content: this.content, cursor: this.cursor, timestamp: Date.now() };
  }

  restore(snapshot: EditorSnapshot) {
    this.content = snapshot.content;
    this.cursor = snapshot.cursor;
  }
}

class History {
  private snapshots: EditorSnapshot[] = [];
  push(s: EditorSnapshot) { this.snapshots.push(s); }
  pop(): EditorSnapshot | undefined { return this.snapshots.pop(); }
}
```

### 22. Visitor

**Problem:** Add operations to objects without modifying them.

**When to Use:** AST processing, report generation across different node types.

**TypeScript Example:**
```typescript
interface ASTNode {
  accept(visitor: ASTVisitor): void;
}

class NumberNode implements ASTNode {
  constructor(public value: number) {}
  accept(visitor: ASTVisitor) { visitor.visitNumber(this); }
}

class BinaryOpNode implements ASTNode {
  constructor(public op: string, public left: ASTNode, public right: ASTNode) {}
  accept(visitor: ASTVisitor) { visitor.visitBinaryOp(this); }
}

interface ASTVisitor {
  visitNumber(node: NumberNode): void;
  visitBinaryOp(node: BinaryOpNode): void;
}

class PrintVisitor implements ASTVisitor {
  visitNumber(n: NumberNode) { process.stdout.write(String(n.value)); }
  visitBinaryOp(n: BinaryOpNode) {
    n.left.accept(this);
    process.stdout.write(` ${n.op} `);
    n.right.accept(this);
  }
}
```

### 23. Interpreter

**Problem:** Need to evaluate a language or expression.

**When to Use:** DSLs, rule engines, math expression parsers.

**Python Example:**
```python
class Expression:
    def interpret(self, context: dict) -> bool: ...

class Equals(Expression):
    def __init__(self, field: str, value):
        self.field = field
        self.value = value
    def interpret(self, ctx: dict) -> bool:
        return ctx.get(self.field) == self.value

class And(Expression):
    def __init__(self, *exprs: Expression):
        self.exprs = exprs
    def interpret(self, ctx: dict) -> bool:
        return all(e.interpret(ctx) for e in self.exprs)

# Usage: DSL for filtering
rule = And(Equals("role", "admin"), Equals("active", True))
rule.interpret({"role": "admin", "active": True})  # True
rule.interpret({"role": "user", "active": True})   # False
```
