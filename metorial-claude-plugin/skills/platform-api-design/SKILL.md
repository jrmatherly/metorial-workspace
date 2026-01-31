---
name: Platform API Design
description: This skill should be used when the user asks to "add endpoint", "create API", "platform backend", "Hono route", "Prisma model", "backend service", mentions "metorial-platform/src/backend/", or discusses TypeScript backend development for the Metorial platform. Provides guidance for building APIs following platform conventions with Hono, Prisma, and Bun.
version: 1.0.0
---

# Platform API Design

This skill provides guidance for developing backend APIs in the Metorial platform. The platform uses TypeScript with Hono framework, Prisma ORM, and Bun runtime.

## Context Loading

Before implementing platform APIs, gather context from existing tools:

### 1. Pattern Context (Drift)

Use Drift to understand existing API patterns:

```
drift_context targeting metorial-platform/src/backend/
```

This provides detected patterns for:
- Route structure and naming
- Request/response handling
- Authentication middleware
- Error handling

### 2. Workflow Context (Serena)

Use Serena for development workflow:

```
read_memory('development_workflow')
```

This provides:
- Build commands (`mise run platform:build`)
- Test commands (`mise run platform:test`)
- Prisma commands (`mise run platform:prisma:generate`)

### 3. Schema Navigation (Serena)

Navigate existing code with Serena:

```
find_symbol targeting metorial-platform/src/backend/
```

## Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Runtime | Bun | Fast JS/TS runtime |
| Framework | Hono | Web framework |
| ORM | Prisma | Database access |
| Validation | Zod | Schema validation |
| Auth | JWT | Authentication |

## Project Structure

```
metorial-platform/src/backend/
├── routes/              # API route handlers
│   ├── auth/           # Authentication routes
│   ├── users/          # User management
│   └── servers/        # MCP server routes
├── middleware/          # Hono middleware
│   ├── auth.ts         # JWT validation
│   └── error.ts        # Error handling
├── services/            # Business logic
├── lib/                 # Shared utilities
│   ├── prisma.ts       # Prisma client
│   └── validation.ts   # Zod schemas
└── index.ts            # App entry point
```

## Implementation Patterns

### Route Definition

Create routes in `routes/<domain>/index.ts`:

```typescript
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';
import { authMiddleware } from '../../middleware/auth';
import { prisma } from '../../lib/prisma';

const app = new Hono();

// Input validation schema
const CreateItemSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(1000).optional(),
});

// Protected route with validation
app.post(
  '/',
  authMiddleware,
  zValidator('json', CreateItemSchema),
  async (c) => {
    const { name, description } = c.req.valid('json');
    const userId = c.get('userId');

    const item = await prisma.item.create({
      data: {
        name,
        description,
        userId,
      },
    });

    return c.json(item, 201);
  }
);

export default app;
```

### Prisma Model Definition

Add models to `prisma/schema.prisma`:

```prisma
model Item {
  id          String   @id @default(cuid())
  name        String
  description String?
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([userId])
}
```

After modifying schema:

```bash
mise run platform:prisma:generate
mise run platform:prisma:push  # or prisma migrate dev
```

### Authentication Middleware

Standard auth pattern:

```typescript
import { Context, Next } from 'hono';
import { verify } from 'hono/jwt';

export const authMiddleware = async (c: Context, next: Next) => {
  const authHeader = c.req.header('Authorization');

  if (!authHeader?.startsWith('Bearer ')) {
    return c.json({ error: 'Unauthorized' }, 401);
  }

  const token = authHeader.slice(7);

  try {
    const payload = await verify(token, process.env.JWT_SECRET!);
    c.set('userId', payload.sub);
    await next();
  } catch {
    return c.json({ error: 'Invalid token' }, 401);
  }
};
```

### Error Handling

Consistent error responses:

```typescript
import { Context, Next } from 'hono';
import { HTTPException } from 'hono/http-exception';

export const errorMiddleware = async (c: Context, next: Next) => {
  try {
    await next();
  } catch (error) {
    if (error instanceof HTTPException) {
      return c.json({ error: error.message }, error.status);
    }

    if (error instanceof Prisma.PrismaClientKnownRequestError) {
      if (error.code === 'P2002') {
        return c.json({ error: 'Resource already exists' }, 409);
      }
      if (error.code === 'P2025') {
        return c.json({ error: 'Resource not found' }, 404);
      }
    }

    console.error('Unhandled error:', error);
    return c.json({ error: 'Internal server error' }, 500);
  }
};
```

## API Conventions

### URL Structure

```
/api/v1/<resource>           # Collection
/api/v1/<resource>/:id       # Individual resource
/api/v1/<resource>/:id/sub   # Nested resource
```

### HTTP Methods

| Method | Purpose | Response Code |
|--------|---------|---------------|
| GET | Retrieve resource(s) | 200 |
| POST | Create resource | 201 |
| PUT | Replace resource | 200 |
| PATCH | Update resource | 200 |
| DELETE | Remove resource | 204 |

### Response Format

**Success response:**

```json
{
  "data": { ... },
  "meta": {
    "total": 100,
    "page": 1,
    "perPage": 20
  }
}
```

**Error response:**

```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": { ... }
}
```

### Pagination

Use cursor-based pagination for lists:

```typescript
const PaginationSchema = z.object({
  cursor: z.string().optional(),
  limit: z.coerce.number().min(1).max(100).default(20),
});

app.get('/', zValidator('query', PaginationSchema), async (c) => {
  const { cursor, limit } = c.req.valid('query');

  const items = await prisma.item.findMany({
    take: limit + 1,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
  });

  const hasMore = items.length > limit;
  const data = hasMore ? items.slice(0, -1) : items;

  return c.json({
    data,
    meta: {
      nextCursor: hasMore ? data[data.length - 1].id : null,
      hasMore,
    },
  });
});
```

## Validation Patterns

### Request Validation

Use Zod for all request validation:

```typescript
// Define schemas in lib/validation/
export const schemas = {
  // Path params
  IdParam: z.object({
    id: z.string().cuid(),
  }),

  // Query params
  SearchQuery: z.object({
    q: z.string().min(1).max(100),
    page: z.coerce.number().positive().default(1),
  }),

  // Body schemas
  CreateUser: z.object({
    email: z.string().email(),
    name: z.string().min(1).max(100),
    password: z.string().min(8),
  }),
};
```

### Response Validation

For type safety, define response types:

```typescript
// Types derived from Prisma
type User = Prisma.UserGetPayload<{
  select: {
    id: true;
    email: true;
    name: true;
    createdAt: true;
  };
}>;

// API response type
interface ApiResponse<T> {
  data: T;
  meta?: {
    total?: number;
    page?: number;
    perPage?: number;
  };
}
```

## Service Layer Pattern

Separate business logic from routes:

```typescript
// services/user.service.ts
import { prisma } from '../lib/prisma';
import { hash, verify } from '../lib/auth';

export const userService = {
  async create(data: CreateUserInput) {
    const hashedPassword = await hash(data.password);

    return prisma.user.create({
      data: {
        ...data,
        password: hashedPassword,
      },
      select: {
        id: true,
        email: true,
        name: true,
      },
    });
  },

  async findById(id: string) {
    return prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        name: true,
      },
    });
  },

  async authenticate(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } });

    if (!user || !(await verify(password, user.password))) {
      return null;
    }

    return { id: user.id, email: user.email };
  },
};
```

## Testing Patterns

### Route Testing

```typescript
import { describe, it, expect, beforeAll } from 'bun:test';
import app from './index';

describe('POST /api/v1/items', () => {
  let authToken: string;

  beforeAll(async () => {
    // Setup test auth token
    authToken = await getTestToken();
  });

  it('creates item with valid input', async () => {
    const res = await app.request('/api/v1/items', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${authToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        name: 'Test Item',
        description: 'Description',
      }),
    });

    expect(res.status).toBe(201);
    const data = await res.json();
    expect(data.name).toBe('Test Item');
  });

  it('rejects invalid input', async () => {
    const res = await app.request('/api/v1/items', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${authToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        name: '', // Invalid: empty
      }),
    });

    expect(res.status).toBe(400);
  });
});
```

## Validation Checklist

Before submitting API changes:

- [ ] Routes follow URL conventions
- [ ] All inputs validated with Zod
- [ ] Authentication applied where needed
- [ ] Error handling covers edge cases
- [ ] Prisma migrations created if schema changed
- [ ] Tests cover success and error cases
- [ ] Pattern compliance verified (`drift_validate_change`)

## Additional Resources

### Reference Files

For detailed patterns and examples:

- **`references/backend-conventions.md`** - Deep backend patterns

### External References

- Serena memory: `development_workflow` for build/test commands
- Drift patterns: `drift_context` for existing conventions
- Platform source: `metorial-platform/src/backend/`
