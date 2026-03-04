# Architecture-Level Design Patterns

> Clean Architecture, Hexagonal, DDD, CQRS, Service Layer, Repository.
> Each pattern has: When to Use → Structure → Framework Example → Migration Path.

---

<HARD-GATE>
Architecture patterns are EXPENSIVE to apply and EXPENSIVE to undo.
Before selecting one, answer:
1. Is this project > 10 endpoints? (If no → skip, use simple MVC)
2. Will this project live > 1 year? (If no → skip, keep it simple)
3. Does the team understand this pattern? (If no → train first, then apply)
</HARD-GATE>

---

## 1. Service Layer Pattern

**The most important pattern for 90% of web apps.**

### Problem
Business logic scattered across:
- API route handlers (Next.js `route.ts`, FastAPI `@router`)
- Database models
- Utility functions
- Frontend components (!)

### Solution
Extract ALL business logic into dedicated Service classes/modules.

### The Rule
```
Controller → Service → Repository → Database
     ↓           ↓          ↓
  HTTP only   Logic only  Data only
```

**Controller knows:** HTTP verbs, request/response shapes, status codes.
**Service knows:** Business rules, validation, orchestration.
**Repository knows:** Database queries, ORM calls.

### Next.js App Router Example

```
src/
├── app/
│   └── api/users/
│       └── route.ts          ← Controller (HTTP only)
├── services/
│   └── user.service.ts       ← Business logic
├── repositories/
│   └── user.repository.ts    ← Data access
└── types/
    └── user.types.ts         ← Shared types
```

```typescript
// app/api/users/route.ts — Controller (THIN)
import { UserService } from "@/services/user.service";
import { CreateUserSchema } from "@/types/user.types";

export async function POST(req: Request) {
  const body = await req.json();
  const validated = CreateUserSchema.parse(body); // ← validate before passing to service
  const result = await UserService.create(validated);
  return Response.json(result, { status: 201 });
}

// services/user.service.ts — Business Logic (FAT)
import { UserRepository } from "@/repositories/user.repository";
import { hashPassword } from "@/lib/crypto";

export class UserService {
  static async create(data: CreateUserDTO) {
    const existing = await UserRepository.findByEmail(data.email);
    if (existing) throw new ConflictError("Email already exists");

    const hashedPassword = await hashPassword(data.password);
    return UserRepository.save({ ...data, password: hashedPassword });
  }
}

// repositories/user.repository.ts — Data Access (PURE)
import { prisma } from "@/lib/prisma";

export class UserRepository {
  static findByEmail(email: string) {
    return prisma.user.findUnique({ where: { email } });
  }
  static save(data: Prisma.UserCreateInput) {
    return prisma.user.create({ data });
  }
}
```

### FastAPI Example

```python
# routers/user.py — Controller
from fastapi import APIRouter, Depends
from schemas.user import CreateUserDTO, User
from services.user_service import UserService

router = APIRouter()

@router.post("/users", status_code=201, response_model=User)
async def create_user(data: CreateUserDTO, service: UserService = Depends()):
    return await service.create(data)

# services/user_service.py — Business Logic
from fastapi import Depends, HTTPException
from repositories.user_repository import UserRepository
from utils.security import hash_password

class UserService:
    def __init__(self, repo: UserRepository = Depends()):
        self.repo = repo

    async def create(self, data: CreateUserDTO) -> User:
        existing = await self.repo.find_by_email(data.email)
        if existing:
            raise HTTPException(409, "Email already exists")
        hashed = hash_password(data.password)
        return await self.repo.save({**data.model_dump(), "password": hashed})

# repositories/user_repository.py — Data Access
from db.client import database

class UserRepository:
    async def find_by_email(self, email: str) -> User | None:
        return await database.users.find_one({"email": email})

    async def save(self, data: dict) -> User:
        return await database.users.insert_one(data)
```

---

## 2. Repository Pattern

### Problem
Database queries (Prisma, SQLAlchemy, raw SQL) called directly from services or worse, from controllers.

### Solution
Abstract ALL data access behind a Repository interface.

### Benefits
- **Swap databases** without changing business logic (PostgreSQL → MongoDB)
- **Unit test services** by mocking the repository
- **Centralize queries** — no more scattered `.findMany()` calls

### Red Flag: DON'T use Repository if
- App has < 5 database interactions
- You're using an ORM that already provides a repository-like API (TypeORM)
- Single data source, never changing

---

## 3. Clean Architecture

### The Dependency Rule
> Source code dependencies must point INWARD. Nothing in an inner circle can know about an outer circle.

```
┌─────────────────────────────────────┐
│         Frameworks & Drivers        │  ← Express, Prisma, React
│  ┌───────────────────────────────┐  │
│  │          Adapters             │  │  ← Controllers, Gateways
│  │  ┌───────────────────────┐   │  │
│  │  │    Use Cases           │   │  │  ← Application logic
│  │  │  ┌─────────────────┐  │   │  │
│  │  │  │    Entities      │  │   │  │  ← Business rules (CORE)
│  │  │  └─────────────────┘  │   │  │
│  │  └───────────────────────┘   │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### When to Use
- Project will live 3+ years
- Team has 5+ developers
- Complex business rules that change independently of UI/DB
- Need to support multiple interfaces (web, mobile, CLI)

### When NOT to Use
- CRUD app with minimal business logic
- Prototype / MVP (speed > architecture)
- Solo developer project < 6 months

---

## 4. Hexagonal Architecture (Ports & Adapters)

### Core Idea
The application core defines "Ports" (interfaces). External systems connect via "Adapters".

```
                    ┌──────────────┐
   HTTP Adapter ──→ │              │ ←── Database Adapter
   CLI  Adapter ──→ │  Application │ ←── Cache Adapter
   gRPC Adapter ──→ │     Core     │ ←── Email Adapter
                    │  (Ports)     │
                    └──────────────┘
```

### When to Use
- Multiple data sources (PostgreSQL + Redis + S3)
- Need to swap infrastructure without touching core logic
- Migrating from one framework to another

---

## 5. Domain-Driven Design (DDD)

### When to Use
- Complex business domain (fintech, healthcare, logistics)
- Domain experts available for collaboration
- Multiple bounded contexts with clear boundaries

### Key Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| **Entity** | Object with unique identity | User(id), Order(id) |
| **Value Object** | Object defined by attributes, no identity | Money(amount, currency), Address |
| **Aggregate** | Cluster of entities with a root | Order (root) + OrderItems |
| **Repository** | Persistence abstraction per aggregate | OrderRepository |
| **Domain Service** | Logic that doesn't belong to any entity | PricingService |
| **Domain Event** | Something that happened | OrderPlaced, PaymentReceived |
| **Bounded Context** | Boundary of a model | Billing vs Shipping vs Inventory |

### NestJS Example (DDD Structure)

```
src/
├── modules/
│   ├── orders/
│   │   ├── domain/
│   │   │   ├── entities/order.entity.ts
│   │   │   ├── value-objects/money.vo.ts
│   │   │   ├── events/order-placed.event.ts
│   │   │   └── repositories/order.repository.interface.ts
│   │   ├── application/
│   │   │   ├── commands/create-order.command.ts
│   │   │   ├── queries/get-order.query.ts
│   │   │   └── services/order.service.ts
│   │   └── infrastructure/
│   │       ├── persistence/order.prisma.repository.ts
│   │       └── controllers/order.controller.ts
```

---

## 6. CQRS (Command Query Responsibility Segregation)

### Problem
Read operations and write operations have VERY different needs:
- **Reads:** Need joins, aggregations, denormalized views
- **Writes:** Need validation, business rules, consistency

### Solution
Separate the Read model from the Write model.

### When to Use
- Dashboard-heavy apps (lots of complex reads)
- Event-sourced systems
- High-traffic apps where read/write ratio is > 10:1

### When NOT to Use
- Simple CRUD with 1:1 read/write ratio
- Small team (overhead not worth it)
- No complex aggregation needs

```typescript
// Command side (Write)
class CreateOrderCommand {
  constructor(
    readonly userId: string,
    readonly items: OrderItemDTO[],
  ) {}
}

class CreateOrderHandler {
  async execute(cmd: CreateOrderCommand): Promise<string> {
    // Validate, apply business rules, save
    const order = Order.create(cmd.userId, cmd.items);
    await this.repo.save(order);
    await this.eventBus.publish(new OrderCreated(order.id));
    return order.id;
  }
}

// Query side (Read) — Optimized for speed
class GetOrderSummaryQuery {
  constructor(readonly orderId: string) {}
}

class GetOrderSummaryHandler {
  async execute(query: GetOrderSummaryQuery) {
    // Direct DB query, denormalized view, no business logic
    return this.readDb.query(`
      SELECT o.*, u.name, COUNT(i.id) as item_count
      FROM orders o JOIN users u ON o.user_id = u.id
      JOIN order_items i ON i.order_id = o.id
      WHERE o.id = $1
      GROUP BY o.id, u.name
    `, [query.orderId]);
  }
}
```

---

## Migration Paths (How to Refactor Incrementally)

### From Monolith MVC → Service Layer
```
Step 1: Create /services folder
Step 2: Move ONE route's logic to a service (the simplest one)
Step 3: Verify tests pass
Step 4: Repeat for next route
Step 5: After all routes use services → create /repositories
```

### From Service Layer → Clean Architecture
```
Step 1: Create /domain/entities — move type definitions
Step 2: Create /domain/repositories — extract interfaces
Step 3: Create /infrastructure — move implementations behind interfaces
Step 4: Wire with dependency injection
```

### From Monolith → Microservices
```
Step 1: Identify bounded contexts (DDD event storming)
Step 2: Extract ONE context that has fewest dependencies
Step 3: Add API gateway between monolith and new service
Step 4: Repeat, extract next least-coupled context
```
