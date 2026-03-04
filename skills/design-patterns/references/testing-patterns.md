# Testing Patterns — How to Test Every Design Pattern

> Each pattern changes HOW you test your code.
> This reference shows the testing strategy for every pattern category.

---

## The Testing Principle

```
A good pattern IMPROVES testability.
If your refactored code is HARDER to test than the original, you chose the wrong pattern.
```

---

## Unit Testing Patterns for GoF

### Testing Factory Method

```typescript
describe("NotificationFactory", () => {
  it("creates EmailNotification for type 'email'", () => {
    const notification = createNotification("email");
    expect(notification).toBeInstanceOf(EmailNotification);
  });

  it("creates SMSNotification for type 'sms'", () => {
    const notification = createNotification("sms");
    expect(notification).toBeInstanceOf(SMSNotification);
  });

  it("throws for unknown type", () => {
    // @ts-expect-error — testing runtime rejection of an invalid type
    expect(() => createNotification("pigeon")).toThrow();
  });
});
```

### Testing Strategy

```typescript
describe("OrderService with PricingStrategy", () => {
  it("applies regular pricing", () => {
    const service = new OrderService(new RegularPricing());
    expect(service.total(100)).toBe(100);
  });

  it("applies premium discount", () => {
    const service = new OrderService(new PremiumPricing());
    expect(service.total(100)).toBe(85);
  });

  it("accepts any PricingStrategy implementation", () => {
    // Mock strategy for edge case testing
    const mockStrategy: PricingStrategy = { calculate: () => 0 };
    const service = new OrderService(mockStrategy);
    expect(service.total(999)).toBe(0);
  });
});
```

### Testing Observer (EventBus)

```typescript
describe("EventBus", () => {
  let bus: EventBus;
  beforeEach(() => { bus = new EventBus(); });

  it("calls all listeners for an event", () => {
    const fn1 = vi.fn();
    const fn2 = vi.fn();
    bus.on("test", fn1);
    bus.on("test", fn2);
    bus.emit("test", { data: 1 });
    expect(fn1).toHaveBeenCalledWith({ data: 1 });
    expect(fn2).toHaveBeenCalledWith({ data: 1 });
  });

  it("supports unsubscribe", () => {
    const fn = vi.fn();
    const unsubscribe = bus.on("test", fn);
    unsubscribe();
    bus.emit("test", {});
    expect(fn).not.toHaveBeenCalled();
  });

  it("does not cross-notify between events", () => {
    const fn = vi.fn();
    bus.on("event-a", fn);
    bus.emit("event-b", {});
    expect(fn).not.toHaveBeenCalled();
  });
});
```

### Testing Command (with Undo)

```typescript
describe("CommandHistory", () => {
  let history: CommandHistory;
  let cart: Cart;
  const laptop: CartItem = { id: "1", name: "Laptop", qty: 1 };

  beforeEach(() => {
    history = new CommandHistory();
    cart = new Cart();
  });

  it("executes command", () => {
    history.execute(new AddItemCommand(cart, laptop));
    expect(cart.items).toHaveLength(1);
  });

  it("undoes last command", () => {
    history.execute(new AddItemCommand(cart, laptop));
    history.undo();
    expect(cart.items).toHaveLength(0);
  });

  it("handles undo on empty history gracefully", () => {
    expect(() => history.undo()).not.toThrow();
  });
});
```

### Testing State Machine

```typescript
describe("Order State Machine", () => {
  let order: Order;
  beforeEach(() => { order = new Order(); });

  it("starts in Pending state", () => {
    expect(() => order.ship()).not.toThrow();
  });

  it("transitions: Pending → Shipped", () => {
    order.ship();
    expect(() => order.ship()).toThrow("Already shipped");
  });

  it("prevents: Shipped → Cancel", () => {
    order.ship();
    expect(() => order.cancel()).toThrow("Cannot cancel shipped order");
  });

  it("transitions: Pending → Cancelled", () => {
    order.cancel();
    expect(() => order.ship()).toThrow("Cannot ship cancelled order");
  });
});
```

### Testing Decorator (Middleware)

```typescript
describe("Middleware Decorators", () => {
  it("withAuth rejects missing token", async () => {
    const handler = withAuth(async () => new Response("OK"));
    const req = new Request("http://test.com");
    const res = await handler(req);
    expect(res?.status).toBe(401);
  });

  it("withAuth passes through with token", async () => {
    const inner = vi.fn(async () => new Response("OK"));
    const handler = withAuth(inner);
    const req = new Request("http://test.com", {
      headers: { Authorization: "Bearer token123" },
    });
    await handler(req);
    expect(inner).toHaveBeenCalled();
  });

  it("withRateLimit blocks after limit", async () => {
    const handler = withRateLimit(2, 60_000, async () => new Response("OK"));
    const req = new Request("http://test.com");
    await handler(req);  // 1
    await handler(req);  // 2
    const res = await handler(req);  // 3 → blocked
    expect(res?.status).toBe(429);
  });
});
```

---

## Testing Architecture Patterns

### Testing Service Layer

**The Key:** Mock the Repository, test the Service logic.

```typescript
import type { Mocked } from "vitest";

describe("UserService", () => {
  let service: UserService;
  let mockRepo: Mocked<UserRepository>;

  beforeEach(() => {
    mockRepo = {
      findByEmail: vi.fn(),
      save: vi.fn(),
    };
    service = new UserService(mockRepo);
  });

  it("creates user when email is unique", async () => {
    mockRepo.findByEmail.mockResolvedValue(null);
    mockRepo.save.mockResolvedValue({ id: "1", email: "test@test.com" });

    const result = await service.create({ email: "test@test.com", password: "123" });
    expect(result.email).toBe("test@test.com");
    expect(mockRepo.save).toHaveBeenCalled();
  });

  it("throws ConflictError when email exists", async () => {
    mockRepo.findByEmail.mockResolvedValue({ id: "1", email: "test@test.com" });
    await expect(service.create({ email: "test@test.com", password: "123" }))
      .rejects.toThrow("Email already exists");
  });
});
```

### Testing Repository

**The Key:** Use a test database (or in-memory DB), test REAL queries.

```typescript
describe("UserRepository (Integration)", () => {
  let repo: UserRepository;

  beforeAll(async () => {
    // Use test database
    await prisma.$connect();
    repo = new UserRepository(prisma);
  });

  afterEach(async () => {
    await prisma.user.deleteMany();
  });

  it("saves and retrieves user", async () => {
    const saved = await repo.save({ email: "test@test.com", name: "Test" });
    const found = await repo.findByEmail("test@test.com");
    expect(found?.id).toBe(saved.id);
  });

  it("returns null for non-existent email", async () => {
    const found = await repo.findByEmail("doesnt@exist.com");
    expect(found).toBeNull();
  });
});
```

### Testing CQRS

```typescript
// Command side — test business rules
describe("CreateOrderHandler", () => {
  let handler: CreateOrderHandler;
  const mockRepo = { save: vi.fn() };
  const mockEventBus = { publish: vi.fn() };

  beforeEach(() => {
    handler = new CreateOrderHandler(mockRepo, mockEventBus);
  });

  it("validates stock before creating order", async () => {
    await expect(handler.execute(new CreateOrderCommand("user-1", [
      { productId: "p-1", quantity: 999 }, // Exceeds stock
    ]))).rejects.toThrow("Insufficient stock");
  });

  it("publishes OrderCreated event", async () => {
    const validCommand = new CreateOrderCommand("user-1", [
      { productId: "p-1", quantity: 1 },
    ]);

    await handler.execute(validCommand);

    expect(mockEventBus.publish).toHaveBeenCalledWith(
      expect.objectContaining({ type: "OrderCreated" })
    );
  });
});

// Query side — test data retrieval
describe("GetOrderSummaryHandler", () => {
  let handler: GetOrderSummaryHandler;
  const readDb = { query: vi.fn() };

  beforeEach(() => {
    handler = new GetOrderSummaryHandler(readDb);
  });

  it("returns denormalized view", async () => {
    const result = await handler.execute(new GetOrderSummaryQuery("ord-1"));
    expect(result).toMatchObject({
      orderId: "ord-1",
      userName: expect.any(String),
      itemCount: expect.any(Number),
    });
  });
});
```

---

## Testing System Patterns

### Testing Circuit Breaker

```typescript
describe("CircuitBreaker", () => {
  let breaker: CircuitBreaker;

  beforeEach(() => {
    breaker = new CircuitBreaker(3, 1000); // 3 failures, 1s timeout
  });

  it("starts in CLOSED state", async () => {
    const result = await breaker.call(() => Promise.resolve("ok"));
    expect(result).toBe("ok");
  });

  it("opens after threshold failures", async () => {
    const failing = () => Promise.reject(new Error("fail"));
    for (let i = 0; i < 3; i++) {
      await breaker.call(failing).catch(() => {});
    }
    await expect(breaker.call(failing)).rejects.toThrow("Circuit is OPEN");
  });

  it("transitions to HALF_OPEN after timeout", async () => {
    // Force open
    const failing = () => Promise.reject(new Error("fail"));
    for (let i = 0; i < 3; i++) {
      await breaker.call(failing).catch(() => {});
    }

    // Wait for timeout
    await new Promise(r => setTimeout(r, 1100));

    // Should allow one test request
    const result = await breaker.call(() => Promise.resolve("recovered"));
    expect(result).toBe("recovered");
  });
});
```

### Testing Saga (Compensation)

```python
# Python — pytest
class TestOrderSaga:
    def test_full_saga_success(self):
        saga = OrderSaga(payment_service, inventory_service, shipping_service)
        result = saga.execute(order_data)
        assert result.status == "COMPLETED"
        assert payment_service.charged
        assert inventory_service.reserved
        assert shipping_service.scheduled

    def test_saga_compensates_on_inventory_failure(self):
        inventory_service.should_fail = True
        saga = OrderSaga(payment_service, inventory_service, shipping_service)

        with pytest.raises(SagaFailedError):
            saga.execute(order_data)

        # Compensation: payment was refunded
        assert payment_service.refunded
        # Shipping was never called
        assert not shipping_service.scheduled
```

---

## Testing Pyramid for Design Patterns

```
         ╱╲
        ╱  ╲        E2E Tests (few)
       ╱    ╲       - Full user flows through the app
      ╱──────╲
     ╱        ╲     Integration Tests (moderate)
    ╱          ╲    - Service + Repository together
   ╱────────────╲   - API route → Service → DB
  ╱              ╲
 ╱                ╲  Unit Tests (many)
╱──────────────────╲ - Each pattern in isolation
                      - Mocked dependencies
```

| Pattern | Unit Test | Integration Test | E2E Test |
|---------|:---------:|:----------------:|:--------:|
| Factory | ✅ Instance type check | — | — |
| Strategy | ✅ Each strategy | ✅ With real data | — |
| Observer | ✅ Emit + receive | ✅ Cross-module events | — |
| Command | ✅ Execute + Undo | — | — |
| State | ✅ All transitions | — | ✅ UI flows |
| Service Layer | ✅ Mock repo | ✅ Real DB | ✅ API test |
| Repository | — | ✅ Real DB | — |
| CQRS | ✅ Handlers | ✅ Command + Query | ✅ Full flow |
| Circuit Breaker | ✅ State transitions | ✅ Real HTTP | — |
| Saga | ✅ Step + compensate | ✅ Multi-service | ✅ Full flow |

---

## Test Organization by Pattern

```
tests/
├── unit/
│   ├── patterns/
│   │   ├── factory.test.ts
│   │   ├── strategy.test.ts
│   │   ├── observer.test.ts
│   │   ├── command.test.ts
│   │   └── state.test.ts
│   └── services/
│       ├── user.service.test.ts
│       └── order.service.test.ts
├── integration/
│   ├── repositories/
│   │   ├── user.repository.test.ts
│   │   └── order.repository.test.ts
│   └── sagas/
│       └── order.saga.test.ts
└── e2e/
    ├── api/
    │   ├── users.e2e.test.ts
    │   └── orders.e2e.test.ts
    └── flows/
        └── checkout.e2e.test.ts
```
