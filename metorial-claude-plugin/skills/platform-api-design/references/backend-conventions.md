# Backend Conventions Reference

Deep patterns and conventions for Metorial platform backend development. Load this reference for complex API implementations.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Hono Application                        │
├─────────────────────────────────────────────────────────────┤
│  Middleware Layer                                           │
│  ├── CORS                                                   │
│  ├── Request Logging                                        │
│  ├── Error Handling                                         │
│  └── Authentication                                         │
├─────────────────────────────────────────────────────────────┤
│  Route Layer                                                │
│  ├── Validation (Zod)                                       │
│  ├── Authorization                                          │
│  └── Response Formatting                                    │
├─────────────────────────────────────────────────────────────┤
│  Service Layer                                              │
│  ├── Business Logic                                         │
│  ├── Data Transformation                                    │
│  └── External API Calls                                     │
├─────────────────────────────────────────────────────────────┤
│  Data Layer (Prisma)                                        │
│  └── Database Operations                                    │
└─────────────────────────────────────────────────────────────┘
```

## Application Setup

### Main Application Entry

```typescript
// src/backend/index.ts
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { logger } from 'hono/logger';
import { errorMiddleware } from './middleware/error';

// Import route modules
import authRoutes from './routes/auth';
import userRoutes from './routes/users';
import serverRoutes from './routes/servers';

const app = new Hono();

// Global middleware
app.use('*', logger());
app.use('*', cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  credentials: true,
}));
app.use('*', errorMiddleware);

// Mount routes
app.route('/api/v1/auth', authRoutes);
app.route('/api/v1/users', userRoutes);
app.route('/api/v1/servers', serverRoutes);

// Health check
app.get('/health', (c) => c.json({ status: 'ok' }));

export default app;
```

### Environment Configuration

```typescript
// src/backend/lib/env.ts
import { z } from 'zod';

const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  DATABASE_URL: z.string(),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRY: z.string().default('7d'),
  ALLOWED_ORIGINS: z.string().optional(),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

export const env = EnvSchema.parse(process.env);
```

## Database Patterns

### Prisma Client Singleton

```typescript
// src/backend/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma = globalThis.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'error', 'warn']
    : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalThis.prisma = prisma;
}
```

### Transaction Pattern

```typescript
async function createOrderWithItems(
  userId: string,
  items: OrderItemInput[]
) {
  return prisma.$transaction(async (tx) => {
    const order = await tx.order.create({
      data: {
        userId,
        status: 'pending',
      },
    });

    await tx.orderItem.createMany({
      data: items.map(item => ({
        orderId: order.id,
        productId: item.productId,
        quantity: item.quantity,
      })),
    });

    return tx.order.findUnique({
      where: { id: order.id },
      include: { items: true },
    });
  });
}
```

### Soft Delete Pattern

```typescript
// In Prisma schema
model Item {
  id        String    @id @default(cuid())
  name      String
  deletedAt DateTime?

  @@index([deletedAt])
}

// Middleware for automatic filtering
prisma.$use(async (params, next) => {
  if (params.model === 'Item') {
    if (params.action === 'findMany' || params.action === 'findFirst') {
      params.args = params.args || {};
      params.args.where = {
        ...params.args.where,
        deletedAt: null,
      };
    }
  }
  return next(params);
});

// Soft delete
await prisma.item.update({
  where: { id },
  data: { deletedAt: new Date() },
});
```

## Authentication Patterns

### JWT Token Generation

```typescript
// src/backend/lib/auth.ts
import { sign, verify } from 'hono/jwt';
import { env } from './env';

interface TokenPayload {
  sub: string;  // User ID
  email: string;
  exp: number;
}

export async function generateToken(user: { id: string; email: string }) {
  const payload: TokenPayload = {
    sub: user.id,
    email: user.email,
    exp: Math.floor(Date.now() / 1000) + parseExpiry(env.JWT_EXPIRY),
  };

  return sign(payload, env.JWT_SECRET);
}

export async function verifyToken(token: string): Promise<TokenPayload | null> {
  try {
    return await verify(token, env.JWT_SECRET) as TokenPayload;
  } catch {
    return null;
  }
}

function parseExpiry(expiry: string): number {
  const match = expiry.match(/^(\d+)([dhms])$/);
  if (!match) return 7 * 24 * 60 * 60; // Default 7 days

  const [, value, unit] = match;
  const multipliers = { d: 86400, h: 3600, m: 60, s: 1 };
  return parseInt(value) * multipliers[unit as keyof typeof multipliers];
}
```

### Refresh Token Pattern

```typescript
// Store refresh tokens in database
model RefreshToken {
  id        String   @id @default(cuid())
  token     String   @unique
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
  createdAt DateTime @default(now())
  revokedAt DateTime?

  @@index([userId])
  @@index([token])
}

// Refresh endpoint
app.post('/auth/refresh', async (c) => {
  const { refreshToken } = await c.req.json();

  const stored = await prisma.refreshToken.findUnique({
    where: { token: refreshToken },
    include: { user: true },
  });

  if (!stored || stored.expiresAt < new Date() || stored.revokedAt) {
    return c.json({ error: 'Invalid refresh token' }, 401);
  }

  // Revoke old token
  await prisma.refreshToken.update({
    where: { id: stored.id },
    data: { revokedAt: new Date() },
  });

  // Generate new tokens
  const accessToken = await generateToken(stored.user);
  const newRefreshToken = await createRefreshToken(stored.user.id);

  return c.json({ accessToken, refreshToken: newRefreshToken });
});
```

### Role-Based Access Control

```typescript
// Middleware factory for role checking
export function requireRole(...roles: string[]) {
  return async (c: Context, next: Next) => {
    const userId = c.get('userId');

    const user = await prisma.user.findUnique({
      where: { id: userId },
      select: { role: true },
    });

    if (!user || !roles.includes(user.role)) {
      return c.json({ error: 'Forbidden' }, 403);
    }

    await next();
  };
}

// Usage
app.delete('/:id', authMiddleware, requireRole('admin'), async (c) => {
  // Only admins can delete
});
```

## Error Handling

### Custom Error Classes

```typescript
// src/backend/lib/errors.ts
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(404, 'NOT_FOUND', `${resource} not found`);
    this.name = 'NotFoundError';
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(409, 'CONFLICT', message);
    this.name = 'ConflictError';
  }
}

export class ValidationError extends AppError {
  constructor(public details: Record<string, string[]>) {
    super(400, 'VALIDATION_ERROR', 'Validation failed');
    this.name = 'ValidationError';
  }
}
```

### Error Middleware

```typescript
// src/backend/middleware/error.ts
export const errorMiddleware = async (c: Context, next: Next) => {
  try {
    await next();
  } catch (error) {
    // Log error for debugging
    console.error('[Error]', {
      path: c.req.path,
      method: c.req.method,
      error: error instanceof Error ? error.message : error,
    });

    if (error instanceof AppError) {
      return c.json({
        error: error.message,
        code: error.code,
        details: (error as ValidationError).details,
      }, error.statusCode);
    }

    if (error instanceof Prisma.PrismaClientKnownRequestError) {
      return handlePrismaError(c, error);
    }

    return c.json({
      error: 'Internal server error',
      code: 'INTERNAL_ERROR',
    }, 500);
  }
};

function handlePrismaError(c: Context, error: Prisma.PrismaClientKnownRequestError) {
  switch (error.code) {
    case 'P2002':
      return c.json({
        error: 'Resource already exists',
        code: 'DUPLICATE',
        field: (error.meta?.target as string[])?.[0],
      }, 409);
    case 'P2025':
      return c.json({
        error: 'Resource not found',
        code: 'NOT_FOUND',
      }, 404);
    default:
      return c.json({
        error: 'Database error',
        code: 'DB_ERROR',
      }, 500);
  }
}
```

## Query Patterns

### Search with Full-Text

```typescript
app.get('/search', zValidator('query', SearchSchema), async (c) => {
  const { q, page = 1, perPage = 20 } = c.req.valid('query');

  const items = await prisma.item.findMany({
    where: {
      OR: [
        { name: { contains: q, mode: 'insensitive' } },
        { description: { contains: q, mode: 'insensitive' } },
      ],
    },
    skip: (page - 1) * perPage,
    take: perPage,
    orderBy: { createdAt: 'desc' },
  });

  const total = await prisma.item.count({
    where: {
      OR: [
        { name: { contains: q, mode: 'insensitive' } },
        { description: { contains: q, mode: 'insensitive' } },
      ],
    },
  });

  return c.json({
    data: items,
    meta: { total, page, perPage, totalPages: Math.ceil(total / perPage) },
  });
});
```

### Filtering and Sorting

```typescript
const FilterSchema = z.object({
  status: z.enum(['active', 'inactive']).optional(),
  createdAfter: z.coerce.date().optional(),
  sortBy: z.enum(['name', 'createdAt']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
});

app.get('/', zValidator('query', FilterSchema), async (c) => {
  const { status, createdAfter, sortBy, sortOrder } = c.req.valid('query');

  const items = await prisma.item.findMany({
    where: {
      ...(status && { status }),
      ...(createdAfter && { createdAt: { gte: createdAfter } }),
    },
    orderBy: { [sortBy]: sortOrder },
  });

  return c.json({ data: items });
});
```

## Caching Patterns

### In-Memory Cache

```typescript
// Simple cache with TTL
const cache = new Map<string, { value: unknown; expires: number }>();

export function cached<T>(
  key: string,
  ttlSeconds: number,
  fn: () => Promise<T>
): Promise<T> {
  const cached = cache.get(key);
  if (cached && cached.expires > Date.now()) {
    return Promise.resolve(cached.value as T);
  }

  return fn().then(value => {
    cache.set(key, { value, expires: Date.now() + ttlSeconds * 1000 });
    return value;
  });
}

// Usage
app.get('/config', async (c) => {
  const config = await cached('app-config', 300, async () => {
    return prisma.config.findMany();
  });
  return c.json({ data: config });
});
```

## File Upload Pattern

```typescript
import { Hono } from 'hono';

app.post('/upload', authMiddleware, async (c) => {
  const formData = await c.req.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return c.json({ error: 'No file provided' }, 400);
  }

  // Validate file type
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  if (!allowedTypes.includes(file.type)) {
    return c.json({ error: 'Invalid file type' }, 400);
  }

  // Validate file size (5MB)
  if (file.size > 5 * 1024 * 1024) {
    return c.json({ error: 'File too large' }, 400);
  }

  // Upload to storage (S3, etc.)
  const url = await uploadToStorage(file);

  return c.json({ url });
});
```

## Rate Limiting

```typescript
import { rateLimiter } from 'hono-rate-limiter';

// Apply to specific routes
app.use('/api/v1/auth/*', rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 100,
  standardHeaders: true,
  keyGenerator: (c) => c.req.header('x-forwarded-for') || 'unknown',
}));

// Stricter limit for login
app.use('/api/v1/auth/login', rateLimiter({
  windowMs: 60 * 60 * 1000, // 1 hour
  limit: 5,
  keyGenerator: (c) => c.req.header('x-forwarded-for') || 'unknown',
}));
```

## Logging Patterns

```typescript
// Structured logging
import pino from 'pino';

export const logger = pino({
  level: env.LOG_LEVEL,
  transport: env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
});

// Request logging middleware
app.use('*', async (c, next) => {
  const start = Date.now();
  const requestId = crypto.randomUUID();

  c.set('requestId', requestId);

  await next();

  logger.info({
    requestId,
    method: c.req.method,
    path: c.req.path,
    status: c.res.status,
    duration: Date.now() - start,
  });
});
```

---

**Note**: For the most current patterns, use `drift_context` to analyze the actual codebase.
