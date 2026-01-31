# MCP Server Patterns Reference

Deep patterns extracted from 33+ existing servers in the Metorial catalog. Load this reference when implementing complex server features or troubleshooting.

## Server Categories

### API Wrapper Servers

Servers that wrap external REST/GraphQL APIs.

**Pattern characteristics:**
- API client class with authentication
- Rate limiting awareness
- Pagination handling
- Error mapping to MCP format

**Examples in catalog:**
- `github` - GitHub API wrapper
- `linear` - Linear API wrapper
- `notion` - Notion API wrapper

### Database Servers

Servers providing database access.

**Pattern characteristics:**
- Connection pooling
- Transaction support
- Query builders
- Schema introspection

**Examples in catalog:**
- `postgres` - PostgreSQL access
- `sqlite` - SQLite database

### File System Servers

Servers for file operations.

**Pattern characteristics:**
- Path validation (prevent traversal)
- Permission checking
- Streaming for large files
- MIME type detection

**Examples in catalog:**
- `filesystem` - Local file access

### Integration Servers

Servers connecting multiple services.

**Pattern characteristics:**
- Multiple API clients
- Data transformation
- Workflow orchestration
- State management

## Authentication Patterns

### API Key Authentication

Most common pattern:

```typescript
const apiKey = process.env.SERVICE_API_KEY;
if (!apiKey) {
  throw new Error('SERVICE_API_KEY required');
}

const headers = {
  'Authorization': `Bearer ${apiKey}`,
  'Content-Type': 'application/json',
};
```

### OAuth Flow

For services requiring OAuth:

```typescript
// Store tokens securely
interface TokenStore {
  accessToken: string;
  refreshToken: string;
  expiresAt: Date;
}

async function getValidToken(store: TokenStore): Promise<string> {
  if (new Date() >= store.expiresAt) {
    const newTokens = await refreshAccessToken(store.refreshToken);
    // Update store
    return newTokens.accessToken;
  }
  return store.accessToken;
}
```

### Service Account

For Google/cloud services:

```typescript
const credentials = JSON.parse(
  process.env.GOOGLE_SERVICE_ACCOUNT_JSON || '{}'
);
```

## Error Handling Patterns

### Standard Error Response

```typescript
function errorResponse(message: string, details?: unknown) {
  return {
    content: [{
      type: 'text' as const,
      text: JSON.stringify({
        error: true,
        message,
        details: details ?? undefined,
      }, null, 2)
    }],
    isError: true,
  };
}
```

### API Error Mapping

```typescript
function handleApiError(error: unknown): MCPResponse {
  if (error instanceof Response) {
    return errorResponse(`API Error: ${error.status} ${error.statusText}`);
  }
  if (error instanceof Error) {
    return errorResponse(error.message);
  }
  return errorResponse('Unknown error occurred');
}
```

### Validation Error

```typescript
function validateInput<T>(input: unknown, schema: ZodSchema<T>): T {
  const result = schema.safeParse(input);
  if (!result.success) {
    throw new ValidationError(
      `Invalid input: ${result.error.errors.map(e => e.message).join(', ')}`
    );
  }
  return result.data;
}
```

## Pagination Patterns

### Cursor-Based

```typescript
interface PaginatedResult<T> {
  items: T[];
  nextCursor?: string;
  hasMore: boolean;
}

server.tool({
  name: 'list_items',
  inputSchema: {
    type: 'object',
    properties: {
      cursor: { type: 'string', description: 'Pagination cursor' },
      limit: { type: 'number', default: 20, maximum: 100 },
    },
  },
  handler: async ({ cursor, limit = 20 }) => {
    const result = await api.listItems({ cursor, limit });
    return {
      content: [{
        type: 'text',
        text: JSON.stringify({
          items: result.items,
          nextCursor: result.nextCursor,
          hasMore: result.hasMore,
        }, null, 2)
      }]
    };
  }
});
```

### Offset-Based

```typescript
server.tool({
  name: 'search_items',
  inputSchema: {
    type: 'object',
    properties: {
      query: { type: 'string' },
      page: { type: 'number', default: 1 },
      perPage: { type: 'number', default: 20 },
    },
    required: ['query'],
  },
  handler: async ({ query, page = 1, perPage = 20 }) => {
    const offset = (page - 1) * perPage;
    const result = await api.search(query, { offset, limit: perPage });
    return {
      content: [{
        type: 'text',
        text: JSON.stringify({
          items: result.items,
          total: result.total,
          page,
          totalPages: Math.ceil(result.total / perPage),
        }, null, 2)
      }]
    };
  }
});
```

## Rate Limiting Patterns

### Token Bucket

```typescript
class RateLimiter {
  private tokens: number;
  private lastRefill: number;

  constructor(
    private maxTokens: number,
    private refillRate: number // tokens per second
  ) {
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
  }

  async acquire(): Promise<void> {
    this.refill();
    if (this.tokens < 1) {
      const waitTime = (1 - this.tokens) / this.refillRate * 1000;
      await new Promise(resolve => setTimeout(resolve, waitTime));
      this.refill();
    }
    this.tokens -= 1;
  }

  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.maxTokens, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }
}
```

### Retry with Backoff

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  let lastError: Error | undefined;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error));

      if (attempt < maxRetries) {
        const delay = baseDelay * Math.pow(2, attempt);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError;
}
```

## Resource Patterns

### Static Resource

```typescript
server.resource({
  uri: 'resource://server/schema',
  name: 'API Schema',
  description: 'OpenAPI schema for the service',
  mimeType: 'application/json',
  handler: async () => ({
    contents: [{
      uri: 'resource://server/schema',
      mimeType: 'application/json',
      text: JSON.stringify(openApiSchema, null, 2),
    }]
  })
});
```

### Dynamic Resource Template

```typescript
server.resourceTemplate({
  uriTemplate: 'resource://server/item/{id}',
  name: 'Item by ID',
  description: 'Fetch item details by ID',
  handler: async (uri, { id }) => {
    const item = await api.getItem(id);
    return {
      contents: [{
        uri: uri.toString(),
        mimeType: 'application/json',
        text: JSON.stringify(item, null, 2),
      }]
    };
  }
});
```

## Testing Patterns

### Unit Testing Tools

```typescript
import { describe, it, expect } from 'bun:test';

describe('tool_name', () => {
  it('should handle valid input', async () => {
    const result = await server.callTool('tool_name', {
      param: 'value'
    });

    expect(result.isError).toBeFalsy();
    expect(result.content[0].text).toContain('expected');
  });

  it('should handle invalid input', async () => {
    const result = await server.callTool('tool_name', {
      param: ''
    });

    expect(result.isError).toBeTruthy();
  });
});
```

### Integration Testing

```typescript
describe('server integration', () => {
  it('should connect to API', async () => {
    // Skip if no API key
    if (!process.env.SERVICE_API_KEY) {
      return;
    }

    const result = await server.callTool('health_check', {});
    expect(result.isError).toBeFalsy();
  });
});
```

## TypeScript Patterns

### Strict Type Definitions

```typescript
// Define explicit types for all inputs
interface CreateItemInput {
  name: string;
  description?: string;
  tags?: string[];
}

// Use Zod for runtime validation
const CreateItemSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(1000).optional(),
  tags: z.array(z.string()).max(10).optional(),
});

// Type-safe handler
const handler = async (input: unknown): Promise<MCPResponse> => {
  const validated = CreateItemSchema.parse(input);
  // validated is now CreateItemInput
};
```

### Response Type Helpers

```typescript
type TextContent = { type: 'text'; text: string };
type ImageContent = { type: 'image'; data: string; mimeType: string };
type Content = TextContent | ImageContent;

interface MCPResponse {
  content: Content[];
  isError?: boolean;
}

function textResponse(text: string): MCPResponse {
  return { content: [{ type: 'text', text }] };
}

function jsonResponse(data: unknown): MCPResponse {
  return textResponse(JSON.stringify(data, null, 2));
}
```

## Performance Patterns

### Caching

```typescript
const cache = new Map<string, { data: unknown; expiry: number }>();

async function cachedFetch<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttlMs = 60000
): Promise<T> {
  const cached = cache.get(key);
  if (cached && cached.expiry > Date.now()) {
    return cached.data as T;
  }

  const data = await fetcher();
  cache.set(key, { data, expiry: Date.now() + ttlMs });
  return data;
}
```

### Batch Operations

```typescript
server.tool({
  name: 'batch_create',
  description: 'Create multiple items in a single request',
  inputSchema: {
    type: 'object',
    properties: {
      items: {
        type: 'array',
        items: { /* item schema */ },
        maxItems: 100,
      },
    },
    required: ['items'],
  },
  handler: async ({ items }) => {
    const results = await Promise.allSettled(
      items.map(item => api.createItem(item))
    );

    const successful = results.filter(r => r.status === 'fulfilled');
    const failed = results.filter(r => r.status === 'rejected');

    return jsonResponse({
      created: successful.length,
      failed: failed.length,
      errors: failed.map(r => (r as PromiseRejectedResult).reason?.message),
    });
  }
});
```

---

**Note**: For the most current patterns, always use `drift_context` to analyze the actual codebase. These patterns are guidelines based on common implementations.
