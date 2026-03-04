# System-Level Design Patterns

> Microservices, Event-Driven, Circuit Breaker, Saga, Cloud Patterns.
> For when a single application is no longer enough.

---

<HARD-GATE>
System-level patterns have the HIGHEST cost of getting wrong.
Before applying ANY system pattern, verify:
1. Team size > 5 developers (microservices for 2 people = suicide)
2. Current deployment pain is REAL (not imagined)
3. You have monitoring/observability in place (no monitoring = no microservices)
</HARD-GATE>

---

## 1. Microservices Architecture

### When to Use
| ✅ Yes | ❌ No |
|--------|-------|
| Team > 5, independent release cycles | Solo dev or small team |
| Different services need different tech stacks | Uniform tech stack |
| Parts of the system need independent scaling | Uniform traffic |
| Organizational structure = separate teams per domain | One team |

### The Decomposition Strategy
```
1. Start with a well-structured Monolith (Modular Monolith)
2. Identify bounded contexts via DDD Event Storming
3. Extract the context with LEAST coupling first
4. Put API Gateway between monolith and new service
5. Extract next context, repeat
6. Kill the monolith when empty
```

### Service Communication Patterns

| Pattern | When | Example |
|---------|------|---------|
| **Synchronous (REST/gRPC)** | Need immediate response | User → Auth Service |
| **Asynchronous (Message Queue)** | Fire-and-forget, eventual consistency | Order → Payment processing |
| **Event-Driven (Pub/Sub)** | Multiple consumers need to react | OrderPlaced → [Email, Inventory, Analytics] |

### Anti-Patterns to Avoid
- **Distributed Monolith:** Services that deploy independently but can't function independently
- **Chatty Services:** Service A calls Service B 10 times per request
- **Shared Database:** Two services reading/writing the same tables

---

## 2. Event-Driven Architecture (EDA)

### Core Concept
Services communicate by publishing and consuming **events** rather than direct calls.

### Event Types

| Type | Description | Example |
|------|------------|---------|
| **Domain Event** | Something happened in business domain | `OrderPlaced`, `PaymentReceived` |
| **Integration Event** | Event shared between bounded contexts | `UserRegistered` (Auth → Billing) |
| **Command Event** | Request to do something | `ProcessPayment` |

### Implementation Stack

| Cloud | Tools |
|-------|-------|
| AWS | SNS + SQS, EventBridge, Kinesis |
| GCP | Pub/Sub, Cloud Tasks |
| Azure | Service Bus, Event Grid |
| Self-hosted | RabbitMQ, Apache Kafka, Redis Streams |

### Event Schema (MANDATORY structure)

```json
{
  "eventId": "uuid-v4",
  "eventType": "order.placed",
  "timestamp": "2026-03-04T10:00:00Z",
  "version": "1.0",
  "source": "order-service",
  "data": {
    "orderId": "ord-123",
    "userId": "usr-456",
    "total": 99.99
  },
  "metadata": {
    "correlationId": "corr-789",
    "traceId": "trace-012"
  }
}
```

---

## 3. Circuit Breaker Pattern

### Problem
Service A calls Service B. Service B is down. Service A keeps retrying, consuming resources, cascading failure.

### Solution
A "circuit breaker" that opens when failures exceed a threshold.

### States

```
     ┌──────────┐  failures > threshold  ┌──────────┐
     │  CLOSED  │ ──────────────────────→ │   OPEN   │
     │ (normal) │                         │ (reject) │
     └──────────┘                         └──────────┘
          ↑                                     │
          │              timeout                │
          │         ┌──────────────┐            │
          └──────── │  HALF-OPEN   │ ←──────────┘
                    │ (test 1 req) │
                    └──────────────┘
```

### Implementation (TypeScript)

```typescript
class CircuitBreaker {
  private failures = 0;
  private state: "CLOSED" | "OPEN" | "HALF_OPEN" = "CLOSED";
  private lastFailure = 0;

  constructor(
    private threshold: number = 5,
    private timeout: number = 30000,
  ) {}

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === "OPEN") {
      if (Date.now() - this.lastFailure > this.timeout) {
        this.state = "HALF_OPEN";
      } else {
        throw new Error("Circuit is OPEN — service unavailable");
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = "CLOSED";
  }

  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
    if (this.failures >= this.threshold) {
      this.state = "OPEN";
    }
  }
}
```

---

## 4. Saga Pattern

### Problem
Distributed transaction across multiple services. Can't use traditional DB transactions.

### Solution
A sequence of local transactions. Each step has a compensating action for rollback.

### Types

| Type | How | Best For |
|------|-----|----------|
| **Choreography** | Each service publishes events, next service reacts | Simple flows (2-3 services) |
| **Orchestration** | Central orchestrator tells each service what to do | Complex flows (4+ services) |

### Example: Order Processing Saga (Orchestration)

```
Step 1: Order Service → Create Order (status: PENDING)
  └── Compensate: Cancel Order

Step 2: Payment Service → Process Payment
  └── Compensate: Refund Payment

Step 3: Inventory Service → Reserve Items
  └── Compensate: Release Items

Step 4: Shipping Service → Schedule Delivery
  └── Compensate: Cancel Shipment

Step 5: Order Service → Update Order (status: COMPLETED)
```

If Step 3 fails:
```
→ Compensate Step 2: Refund Payment
→ Compensate Step 1: Cancel Order
→ Publish: OrderFailed event
```

---

## 5. API Gateway Pattern

### Problem
- Clients need to call multiple microservices
- Cross-cutting concerns (auth, rate limiting, logging) duplicated

### Solution
Single entry point that routes, authenticates, and rate-limits.

### Responsibilities

| Concern | Gateway Handles |
|---------|----------------|
| Authentication | JWT validation, API key check |
| Rate Limiting | Per-client throttling |
| Routing | URL → Service mapping |
| Load Balancing | Distribute across instances |
| Caching | Response caching for reads |
| Monitoring | Request logging, metrics |

---

## 6. Bulkhead Pattern

### Problem
One slow endpoint consumes all thread pool / connection pool resources, starving others.

### Solution
Isolate resources per service/endpoint. If one bulkhead fills, others still work.

```
┌─────────────────────────────────────┐
│          Shared Thread Pool          │  ← DANGEROUS
└─────────────────────────────────────┘

┌──────────┐ ┌──────────┐ ┌──────────┐
│ Orders   │ │ Payments │ │ Shipping │  ← SAFE (isolated)
│ Pool: 10 │ │ Pool: 5  │ │ Pool: 3  │
└──────────┘ └──────────┘ └──────────┘
```

---

## 7. Sidecar Pattern

### Problem
Cross-cutting concerns (logging, monitoring, config) need to be added to every service.

### Solution
Deploy a companion container alongside each service that handles these concerns.

### Use Cases
- Service mesh (Istio, Linkerd)
- Log collection (Fluentd sidecar)
- Config injection (Vault sidecar)

---

## 8. Strangler Fig Pattern

### Problem
Need to migrate from legacy system to new system WITHOUT big-bang rewrite.

### Solution
Gradually replace parts of the old system by routing specific requests to the new system.

```
Phase 1: 100% traffic → Legacy
Phase 2: /api/users → New, everything else → Legacy
Phase 3: /api/users + /api/orders → New
Phase N: 100% traffic → New, Legacy decommissioned
```

---

## Pattern Decision Matrix

| Pain Point | → Pattern | Complexity | Team Size |
|-----------|-----------|------------|-----------|
| Cascading failures | Circuit Breaker | Low | Any |
| Multi-service transactions | Saga | High | 5+ |
| Client calls too many services | API Gateway | Medium | 3+ |
| One endpoint starves others | Bulkhead | Low | Any |
| Legacy migration | Strangler Fig | Medium | Any |
| Cross-cutting concerns | Sidecar | Medium | 3+ |
| Monolith too big | Microservices | Very High | 8+ |
