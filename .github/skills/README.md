# Agent Skills

This directory contains [Agent Skills](https://agentskills.io) - production-ready implementation guides that AI agents (GitHub Copilot, Claude Code, VS Code) automatically discover and use.

## Available Skills (17)

### Resilience
| Skill | Description | Time |
|-------|-------------|------|
| [circuit-breaker](./circuit-breaker/) | Prevent cascade failures with circuit breaker pattern | ~4h |
| [retry-fallback](./retry-fallback/) | Exponential backoff with graceful fallbacks | ~2h |
| [graceful-shutdown](./graceful-shutdown/) | Clean shutdown with job tracking and buffer draining | ~3h |

### Security
| Skill | Description | Time |
|-------|-------------|------|
| [jwt-auth](./jwt-auth/) | JWT authentication with refresh tokens | ~4h |
| [middleware-protection](./middleware-protection/) | Route protection with Next.js middleware | ~2h |
| [audit-logging](./audit-logging/) | Compliance-ready audit trails | ~4h |
| [webhook-security](./webhook-security/) | Secure webhook signature verification | ~3h |

### API
| Skill | Description | Time |
|-------|-------------|------|
| [rate-limiting](./rate-limiting/) | Subscription-aware API rate limiting | ~4h |
| [request-validation](./request-validation/) | Schema validation with Zod/Pydantic | ~2h |
| [api-versioning](./api-versioning/) | Backward-compatible API evolution | ~3h |
| [idempotency](./idempotency/) | Safe retry handling for critical operations | ~4h |

### Operations
| Skill | Description | Time |
|-------|-------------|------|
| [health-checks](./health-checks/) | Kubernetes-ready health endpoints | ~2h |
| [logging-observability](./logging-observability/) | Structured logging with correlation IDs | ~4h |

### Errors
| Skill | Description | Time |
|-------|-------------|------|
| [error-handling](./error-handling/) | Consistent error responses and logging | ~3h |

### Foundations
| Skill | Description | Time |
|-------|-------------|------|
| [typescript-strict](./typescript-strict/) | Strict TypeScript with branded types and Result patterns | ~1h |

### Frontend & UI
| Skill | Description | Time |
|-------|-------------|------|
| [better-icons](./better-icons/) | Search and retrieve icons from 200+ libraries via Iconify CLI/MCP | ~1h |
| [vercel-react-best-practices](./vercel-react-best-practices/) | React/Next.js performance optimization (45 rules, 8 categories) | Reference |

## Usage

Skills are automatically discovered by compatible AI agents. Just ask naturally:

- "Add circuit breaker to my API client"
- "Implement rate limiting for my endpoints"
- "Add JWT authentication"
- "Set up health checks for Kubernetes"
- "Add structured logging with correlation IDs"

## Source

Skills sourced from [Drift Detect](https://github.com/dadbodgeoff/drift) skills library.
Additional skills available in `.local_docs/drift-docs/skills/` (75 total).

## Adding More Skills

To add more skills from the local documentation:

```bash
cp -r .local_docs/drift-docs/skills/<skill-name> .github/skills/
```

## License

MIT - Use these skills freely in your projects.
