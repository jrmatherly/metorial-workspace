---
name: MCP Server Development
description: This skill should be used when the user asks to "create MCP server", "build server", "add server to catalog", "implement MCP", "new server", mentions "metorial/servers/", or discusses Model Context Protocol server development. Provides comprehensive guidance for building MCP servers that match existing Metorial catalog patterns using the SDK and established conventions.
version: 1.0.0
---

# MCP Server Development

This skill provides guidance for creating MCP (Model Context Protocol) servers for the Metorial catalog. The catalog contains 33+ production servers providing patterns to follow.

## Context Loading

Before implementing a new MCP server, gather context from existing tools:

### 1. Pattern Context (Drift)

Use Drift to understand existing server patterns:

```
drift_context targeting metorial/servers/
```

This provides detected patterns for:
- Tool definitions and schemas
- Error handling approaches
- Resource management
- Authentication patterns

### 2. Workflow Context (Serena)

Use Serena for development workflow:

```
read_memory('development_workflow')
```

This provides:
- Build commands (`mise run catalog:build`)
- Test patterns
- Package manager usage (Yarn for catalog)

### 3. SDK Types (Serena)

Navigate SDK types with Serena:

```
find_symbol targeting metorial/packages/sdk/
```

Key types: `MCPServer`, `Tool`, `Resource`, `Prompt`

## Server Structure

Create servers in `metorial/servers/<server-name>/`:

```
<server-name>/
├── server.ts          # Main server implementation
├── metorial.json      # Server metadata manifest
├── package.json       # Dependencies
├── tsconfig.json      # TypeScript configuration
└── README.md          # Server documentation
```

## Implementation Workflow

### Phase 1: Planning

1. Define server purpose and scope
2. List tools the server will provide
3. Identify external dependencies/APIs
4. Review similar servers with `drift_code_examples`

### Phase 2: Manifest Creation

Create `metorial.json`:

```json
{
  "name": "server-name",
  "title": "Human Readable Title",
  "description": "What this server does",
  "version": "1.0.0",
  "tools": [
    {
      "name": "tool_name",
      "description": "What the tool does"
    }
  ],
  "configuration": {
    "apiKey": {
      "type": "string",
      "description": "API key for authentication",
      "required": true,
      "env": "SERVICE_API_KEY"
    }
  }
}
```

### Phase 3: Server Implementation

Follow the standard server pattern:

```typescript
import { MCPServer } from '@metorial/sdk';

const server = new MCPServer({
  name: 'server-name',
  version: '1.0.0',
});

// Define tools
server.tool({
  name: 'tool_name',
  description: 'Tool description',
  inputSchema: {
    type: 'object',
    properties: {
      param: { type: 'string', description: 'Parameter description' }
    },
    required: ['param']
  },
  handler: async (input) => {
    // Implementation
    return { content: [{ type: 'text', text: 'Result' }] };
  }
});

// Start server
server.start();
```

### Phase 4: Validation

1. **Build validation**:
   ```bash
   mise run catalog:build
   ```

2. **Pattern compliance**:
   ```
   drift_validate_change
   ```

3. **Local testing**:
   ```bash
   bun run dev
   ```

## Best Practices

### Error Handling

Follow established error patterns:

```typescript
try {
  const result = await apiCall();
  return { content: [{ type: 'text', text: JSON.stringify(result) }] };
} catch (error) {
  return {
    content: [{
      type: 'text',
      text: `Error: ${error instanceof Error ? error.message : 'Unknown error'}`
    }],
    isError: true
  };
}
```

### Configuration

Use environment variables for sensitive data:

```typescript
const apiKey = process.env.SERVICE_API_KEY;
if (!apiKey) {
  throw new Error('SERVICE_API_KEY environment variable required');
}
```

### Tool Naming

Follow conventions:
- Use snake_case for tool names
- Prefix with service name for clarity: `github_create_issue`
- Use descriptive names that indicate action

### Input Validation

Validate inputs before processing:

```typescript
handler: async (input) => {
  if (!input.param || typeof input.param !== 'string') {
    return {
      content: [{ type: 'text', text: 'Invalid param: must be a non-empty string' }],
      isError: true
    };
  }
  // Process valid input
}
```

## Common Patterns

### API Client Pattern

For servers wrapping external APIs:

```typescript
class ApiClient {
  constructor(private apiKey: string, private baseUrl: string) {}

  async request<T>(endpoint: string, options?: RequestInit): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    if (!response.ok) {
      throw new Error(`API error: ${response.status} ${response.statusText}`);
    }

    return response.json();
  }
}
```

### Resource Provider Pattern

For servers providing file/data resources:

```typescript
server.resource({
  uri: 'resource://server-name/item/{id}',
  name: 'Item Resource',
  description: 'Access item by ID',
  handler: async (uri) => {
    const id = uri.pathname.split('/').pop();
    const item = await fetchItem(id);
    return {
      contents: [{
        uri: uri.toString(),
        mimeType: 'application/json',
        text: JSON.stringify(item)
      }]
    };
  }
});
```

## Validation Checklist

Before submitting a new server:

- [ ] Server builds without errors (`mise run catalog:build`)
- [ ] `metorial.json` manifest is complete and accurate
- [ ] All tools have descriptive names and documentation
- [ ] Error handling follows established patterns
- [ ] Configuration uses environment variables
- [ ] README.md documents usage and configuration
- [ ] Pattern compliance verified (`drift_validate_change`)

## Additional Resources

### Reference Files

For detailed patterns and examples:

- **`references/server-patterns.md`** - Deep patterns from existing 33+ servers
- **`references/sdk-api.md`** - SDK API reference (when created)

### External References

- Serena memory: `development_workflow` for build/test commands
- Drift patterns: `drift_context` for existing conventions
- SDK source: `metorial/packages/sdk/`
