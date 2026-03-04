# Framework-Specific Pattern Catalog

> How design patterns look in YOUR framework.
> Not generic theory — production-ready patterns for Next.js, FastAPI, NestJS, Django, Express, and Go.

---

## Next.js (App Router)

### Recommended Architecture

```
src/
├── app/                     ← Routes (Controllers)
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── api/
│   │   ├── users/route.ts   ← API Route (thin controller)
│   │   └── orders/route.ts
│   └── dashboard/
│       └── page.tsx
├── services/                ← Business Logic (FAT)
│   ├── user.service.ts
│   └── order.service.ts
├── repositories/            ← Data Access
│   ├── user.repository.ts
│   └── order.repository.ts
├── domain/                  ← Entities & Value Objects
│   ├── entities/
│   │   ├── user.entity.ts
│   │   └── order.entity.ts
│   └── events/
│       └── order-placed.event.ts
├── lib/                     ← Infrastructure
│   ├── prisma.ts
│   ├── redis.ts
│   └── auth.ts
└── types/                   ← Shared Types
    └── index.ts
```

### Key Patterns for Next.js

| Pattern | Where | Example |
|---------|-------|---------|
| **Service Layer** | `services/` | All business logic extracted from route.ts |
| **Repository** | `repositories/` | All Prisma calls behind repository interface |
| **Factory** | `lib/` | Creating different auth providers |
| **Strategy** | `services/` | Different pricing calculations |
| **Decorator** | `middleware.ts` | Auth + rate limiting middleware chain |
| **Observer** | `domain/events/` | Domain events for side effects |

### Server Actions Pattern

```typescript
// app/actions/user.actions.ts
"use server"
import { UserService } from "@/services/user.service";
import { revalidatePath } from "next/cache";

export async function createUser(formData: FormData) {
  const data = Object.fromEntries(formData);
  await UserService.create(data);
  revalidatePath("/dashboard/users");
}
```

### Data Fetching Pattern (React Server Components)

```typescript
// app/dashboard/users/page.tsx
import { UserService } from "@/services/user.service";

// Server Component — NO useEffect, NO useState
export default async function UsersPage() {
  const users = await UserService.getAll(); // Direct service call
  return <UserTable users={users} />;
}
```

---

## FastAPI (Python)

### Recommended Architecture

```
app/
├── main.py                  ← App entry + middleware
├── routers/                 ← Controllers (thin)
│   ├── user_router.py
│   └── order_router.py
├── services/                ← Business Logic (fat)
│   ├── user_service.py
│   └── order_service.py
├── repositories/            ← Data Access
│   ├── user_repository.py
│   └── order_repository.py
├── domain/                  ← Entities
│   ├── models.py            ← SQLAlchemy / Pydantic models
│   └── events.py
├── schemas/                 ← Request/Response DTOs
│   ├── user_schemas.py
│   └── order_schemas.py
├── core/                    ← Config, Security, DB
│   ├── config.py
│   ├── security.py
│   └── database.py
└── dependencies/            ← Dependency Injection
    └── auth.py
```

### Key Patterns for FastAPI

| Pattern | Implementation | Example |
|---------|---------------|---------|
| **Dependency Injection** | `Depends()` | Service injection into routes |
| **Repository** | Class with DB session | `UserRepository(session)` |
| **Strategy** | Protocol/ABC | Different pricing strategies |
| **Factory** | `Depends()` factory functions | Create service with dependencies |
| **Decorator** | Python decorators + middleware | `@require_auth`, `@rate_limit` |
| **Observer** | Background tasks | `background_tasks.add_task(send_email)` |

### Dependency Injection Pattern

```python
# dependencies/auth.py
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)):
    user = await verify_token(token.credentials)
    if not user:
        raise HTTPException(401, "Invalid token")
    return user

# routers/user_router.py
@router.get("/me")
async def get_profile(user: User = Depends(get_current_user)):
    return user
```

---

## NestJS (TypeScript)

### Recommended Architecture (DDD-Ready)

```
src/
├── modules/
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── users.controller.ts   ← HTTP Layer
│   │   ├── users.service.ts      ← Business Logic
│   │   ├── users.repository.ts   ← Data Access
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   └── update-user.dto.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   └── guards/
│   │       └── roles.guard.ts
│   └── orders/
│       └── ...
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── interceptors/
│   └── pipes/
└── config/
    ├── database.config.ts
    └── app.config.ts
```

### NestJS Built-in Patterns

| Pattern | NestJS Feature | Usage |
|---------|---------------|-------|
| **DI Container** | `@Injectable()` | Auto-injection everywhere |
| **Decorator** | `@UseGuards()`, `@UseInterceptors()` | Cross-cutting concerns |
| **Strategy** | Custom providers | Swappable implementations |
| **Observer** | `@EventEmitter2` | Domain events |
| **CQRS** | `@nestjs/cqrs` | Command/Query separation |
| **Middleware** | `NestMiddleware` | Request pipeline |

---

## Django (Python)

### Recommended Architecture (Beyond default MTV)

```
project/
├── apps/
│   ├── users/
│   │   ├── models.py           ← Data Model (thin)
│   │   ├── views.py            ← Controller (thin)
│   │   ├── services.py         ← Business Logic (fat) ← ADD THIS
│   │   ├── repositories.py     ← Data Access ← ADD THIS
│   │   ├── serializers.py      ← REST Framework DTOs
│   │   ├── urls.py
│   │   └── tests/
│   └── orders/
│       └── ...
├── core/
│   ├── settings.py
│   └── urls.py
└── shared/
    ├── base_service.py
    └── base_repository.py
```

**The Django Trap:** Django encourages putting logic in Views and Models (Fat Models).
**The Fix:** Add `services.py` and `repositories.py` to every app.

---

## Express.js (Node.js)

### Recommended Architecture

```
src/
├── routes/                  ← Thin controllers (routers)
│   ├── user.routes.ts
│   └── order.routes.ts
├── controllers/             ← Request/Response handling
│   ├── user.controller.ts
│   └── order.controller.ts
├── services/                ← Business logic
│   ├── user.service.ts
│   └── order.service.ts
├── repositories/            ← Data access
│   ├── user.repository.ts
│   └── order.repository.ts
├── middleware/              ← Cross-cutting
│   ├── auth.middleware.ts
│   ├── error.middleware.ts
│   └── rate-limit.middleware.ts
├── models/                  ← DB schemas
│   └── user.model.ts
├── types/                   ← TypeScript types
│   └── index.ts
└── app.ts                   ← Express app setup
```

---

## Go (Golang)

### Recommended Architecture

```
cmd/
└── api/
    └── main.go              ← Entry point
internal/
├── handler/                 ← HTTP handlers (controllers)
│   ├── user_handler.go
│   └── order_handler.go
├── service/                 ← Business logic
│   ├── user_service.go
│   └── order_service.go
├── repository/              ← Data access (interfaces)
│   ├── user_repository.go
│   └── postgres/
│       └── user_postgres.go ← Implementation
├── domain/                  ← Entities
│   ├── user.go
│   └── order.go
├── middleware/               ← HTTP middleware
│   ├── auth.go
│   └── logging.go
└── config/
    └── config.go
pkg/                         ← Shared packages
├── validator/
└── response/
```

### Go Interface Pattern (Dependency Inversion)

```go
// repository/user_repository.go — Interface
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*domain.User, error)
    Save(ctx context.Context, user *domain.User) error
}

// repository/postgres/user_postgres.go — Implementation
type postgresUserRepo struct {
    db *sql.DB
}

func (r *postgresUserRepo) FindByID(ctx context.Context, id string) (*domain.User, error) {
    // SQL query here
}

// service/user_service.go — Depends on interface, not implementation
type UserService struct {
    repo repository.UserRepository // Interface!
}

func NewUserService(repo repository.UserRepository) *UserService {
    return &UserService{repo: repo}
}
```

---

## Framework Selection Guide

| Need | → Framework | Why |
|------|------------|-----|
| Full-stack React + API | **Next.js** | Server Components, Server Actions, best DX |
| Pure REST API (Python) | **FastAPI** | Async, auto-docs, type hints |
| Enterprise (TypeScript) | **NestJS** | Built-in DI, CQRS, microservices |
| Rapid prototyping (Python) | **Django** | Batteries-included, admin panel |
| Minimal API (Node.js) | **Express** | Flexible, huge ecosystem |
| High-performance API | **Go** | Goroutines, compiled, minimal overhead |
