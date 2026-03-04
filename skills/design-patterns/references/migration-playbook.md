# Migration Playbook — Step-by-Step Refactoring Guides

> Real-world migration paths with exact steps, verification checkpoints,
> and rollback plans. No hand-waving, no "just refactor it."

---

<HARD-GATE>
Before starting ANY migration:
1. ✅ All existing tests pass (green baseline)
2. ✅ Git branch created (never refactor on main)
3. ✅ Scope defined (ONE module per migration, never the whole app)
If ANY is missing → STOP. Set up first.
</HARD-GATE>

---

## Migration 1: Spaghetti Route → Service Layer

**Difficulty:** ⭐⭐ (Beginner-friendly)
**Time:** 1-2 hours per route
**Risk:** Low (no external changes)

### Before
```
src/
├── app/api/users/route.ts    ← 200+ LOC, DB + logic + HTTP mixed
```

### After
```
src/
├── app/api/users/route.ts    ← 20 LOC, HTTP only
├── services/user.service.ts  ← Business logic
├── repositories/user.repository.ts ← Data access
```

### Steps

**Step 1: Create empty service file**
```typescript
// services/user.service.ts
export class UserService {
  // Will be filled in Step 3
}
```

**Step 2: Create empty repository file**
```typescript
// repositories/user.repository.ts
import { prisma } from "@/lib/prisma";

export class UserRepository {
  // Will be filled in Step 3
}
```

**Step 3: Extract ONE function at a time**

Pick the simplest route handler. Move its logic:
```typescript
// BEFORE: app/api/users/route.ts
export async function GET() {
  const users = await prisma.user.findMany({ where: { active: true } });
  const formatted = users.map(u => ({ ...u, fullName: `${u.first} ${u.last}` }));
  return Response.json(formatted);
}
```

Extract:
```typescript
// repositories/user.repository.ts
export class UserRepository {
  static findActive() {
    return prisma.user.findMany({ where: { active: true } });
  }
}

// services/user.service.ts
export class UserService {
  static async getActiveUsers() {
    const users = await UserRepository.findActive();
    return users.map(u => ({ ...u, fullName: `${u.first} ${u.last}` }));
  }
}

// app/api/users/route.ts (NOW THIN)
export async function GET() {
  const users = await UserService.getActiveUsers();
  return Response.json(users);
}
```

**Step 4: Verify**
```bash
# Run existing tests
npm test
# Manual test the endpoint
curl http://localhost:3000/api/users
```

**Step 5: Repeat** for POST, PUT, DELETE handlers.

**Step 6: Commit**
```bash
git add -A && git commit -m "refactor: extract UserService + UserRepository from route handler"
```

### Rollback
```bash
git revert HEAD
```

---

## Migration 2: If/Else Chain → Strategy Pattern

**Difficulty:** ⭐⭐ (Beginner-friendly)
**Time:** 30 minutes
**Risk:** Low

### Before
```typescript
function calculateShipping(method: string, weight: number): number {
  if (method === "standard") return weight * 0.5;
  if (method === "express") return weight * 1.5 + 10;
  if (method === "overnight") return weight * 3 + 25;
  if (method === "drone") return weight * 5 + 50;  // Added last week
  throw new Error("Unknown method");
}
```

### Steps

**Step 1: Define interface**
```typescript
interface ShippingCalculator {
  calculate(weight: number): number;
}
```

**Step 2: Create implementations**
```typescript
class StandardShipping implements ShippingCalculator {
  calculate(weight: number) { return weight * 0.5; }
}
class ExpressShipping implements ShippingCalculator {
  calculate(weight: number) { return weight * 1.5 + 10; }
}
class OvernightShipping implements ShippingCalculator {
  calculate(weight: number) { return weight * 3 + 25; }
}
class DroneShipping implements ShippingCalculator {
  calculate(weight: number) { return weight * 5 + 50; }
}
```

**Step 3: Create registry**
```typescript
const shippingStrategies: Record<string, ShippingCalculator> = {
  standard: new StandardShipping(),
  express: new ExpressShipping(),
  overnight: new OvernightShipping(),
  drone: new DroneShipping(),
};

function calculateShipping(method: string, weight: number): number {
  const strategy = shippingStrategies[method];
  if (!strategy) throw new Error(`Unknown shipping method: ${method}`);
  return strategy.calculate(weight);
}
```

**Step 4: Verify** — existing tests should pass without changes.

### Why This Is Better
- Adding "same-day" shipping = add ONE class + ONE registry entry
- Zero modification to existing code (Open/Closed Principle)
- Each strategy is independently testable

---

## Migration 3: Scattered DB → Repository Pattern

**Difficulty:** ⭐⭐⭐ (Intermediate)
**Time:** 2-4 hours
**Risk:** Medium (must find ALL DB calls)

### Steps

**Step 1: Grep for all DB calls**
```bash
# Find every Prisma usage
grep -rn "prisma\." src/ --include="*.ts" | grep -v "node_modules"
# Group by entity
grep -rn "prisma\.user" src/ --include="*.ts"
grep -rn "prisma\.order" src/ --include="*.ts"
```

**Step 2: Create repository per entity**
```typescript
// repositories/user.repository.ts
import { prisma } from "@/lib/prisma";
import type { Prisma, User } from "@prisma/client";

export class UserRepository {
  static findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { id } });
  }
  static findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { email } });
  }
  static create(data: Prisma.UserCreateInput): Promise<User> {
    return prisma.user.create({ data });
  }
  static update(id: string, data: Prisma.UserUpdateInput): Promise<User> {
    return prisma.user.update({ where: { id }, data });
  }
  static delete(id: string): Promise<User> {
    return prisma.user.delete({ where: { id } });
  }
}
```

**Step 3: Replace EACH `prisma.user.*` call with `UserRepository.*`**

One file at a time. Run tests after each file.

**Step 4: Verify no direct Prisma calls remain**
```bash
# This should return 0 results outside repositories/
grep -rn "prisma\." src/ --include="*.ts" | grep -v "repositories/" | grep -v "lib/prisma"
```

---

## Migration 4: Monolith → Modular Monolith

**Difficulty:** ⭐⭐⭐⭐ (Advanced)
**Time:** 1-2 weeks
**Risk:** High

### Steps

**Step 1: Identify bounded contexts**

Run an Event Storming session or analyze your DB schema:
```
users + auth + profiles     → User Context
orders + payments + invoices → Order Context
products + categories + stock → Catalog Context
notifications + emails + sms → Notification Context
```

**Step 2: Create module folders**
```
src/
├── modules/
│   ├── users/
│   │   ├── user.module.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   └── user.types.ts
│   ├── orders/
│   │   ├── order.module.ts
│   │   ├── order.service.ts
│   │   ├── order.repository.ts
│   │   └── order.types.ts
│   └── catalog/
│       └── ...
└── shared/           ← Cross-cutting concerns ONLY
    ├── config.ts
    ├── logger.ts
    └── types.ts
```

**Step 3: Move files ONE module at a time**

Start with the module that has the FEWEST dependencies.

**Step 4: Enforce boundaries**
```json
// tsconfig.json — path aliases per module
{
  "paths": {
    "@users/*": ["src/modules/users/*"],
    "@orders/*": ["src/modules/orders/*"],
    "@shared/*": ["src/shared/*"]
  }
}
```

**Step 5: Module communication**
```typescript
// Modules communicate via events, NOT direct imports
// ✅ GOOD: orders/ emits "OrderPlaced" event, users/ listens
// ❌ BAD:  orders/ directly imports users/user.service.ts
```

---

## Migration 5: Modular Monolith → Microservices

**Difficulty:** ⭐⭐⭐⭐⭐ (Expert)
**Time:** Months
**Risk:** Very High

### Prerequisites
- ✅ Already have a well-structured Modular Monolith
- ✅ Team size > 5 (need independent team per service)
- ✅ Monitoring & logging infrastructure in place
- ✅ CI/CD pipeline per module

### Steps

**Step 1: Extract the LEAST-coupled module**

Use dependency analysis:
```bash
npx madge --image graph.svg src/modules/
```
Pick the module with fewest incoming arrows.

**Step 2: Create a new service**
```bash
# Clone and scaffold
npx create-next-app notifications-service
# Or for API:
npx -y create-fastapi-app notifications-service
```

**Step 3: Implement API Gateway**
```
Client → API Gateway → Monolith (users, orders)
                     → Notifications Service (new)
```

**Step 4: Data migration**
```sql
-- Move tables owned by extracted module to its own database
-- Set up data sync for transition period
```

**Step 5: Switch traffic gradually**
```
Week 1: 10% traffic → new service (canary)
Week 2: 50% traffic → new service
Week 3: 100% traffic → new service
Week 4: Remove old code from monolith
```

**Step 6: Repeat** for next least-coupled module.

---

## Migration Verification Checklist

After EVERY migration step:

```markdown
## Post-Migration Checklist

- [ ] All existing tests pass
- [ ] New unit tests written for extracted code
- [ ] No behavior change (same input → same output)
- [ ] No new circular dependencies (run `npx madge --circular`)
- [ ] Performance not degraded (run benchmark)
- [ ] Committed with descriptive message
- [ ] PR reviewed by at least 1 person
```
