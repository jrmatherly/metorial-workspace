# Self-Hosting Workflow

## Load When
- Setting up self-hosted deployment
- Debugging Docker services
- Building local images

---

## Quick Start

```bash
# 1. Start infrastructure services
mise run selfhost:infra-up

# 2. Initialize databases (creates 'engine' db)
mise run selfhost:init-db

# 3. Build local Docker images
mise run selfhost:build-all

# 4. Start all services
mise run selfhost:up

# 5. Check status
mise run selfhost:status
```

## Available Tasks

| Task | Description |
|------|-------------|
| `selfhost:infra-up` | Start infrastructure (postgres, redis, mongo, etc.) |
| `selfhost:infra-down` | Stop infrastructure services |
| `selfhost:init-db` | Initialize databases (creates engine db) |
| `selfhost:build-api` | Build API Docker image locally |
| `selfhost:build-engine` | Build MCP Engine Docker image locally |
| `selfhost:build-frontend` | Build Dashboard frontend Docker image locally |
| `selfhost:build-all` | Build all Docker images |
| `selfhost:up` | Start all services |
| `selfhost:down` | Stop all services |
| `selfhost:status` | Show service status and health |
| `selfhost:logs` | Tail service logs |
| `selfhost:reset` | Reset all data (DESTRUCTIVE) |

## Shell Aliases

When mise shell integration is active:
- `selfup` - Start all services
- `selfdown` - Stop all services
- `selfstatus` - Check status
- `selflogs` - Tail logs
- `selfbuild` - Build all images
- `selfreset` - Reset data

## Configuration

### Environment Variables

Copy `self-hosting/.env.example` to `self-hosting/.env`:

```bash
# Required
SECRET=your-secret-key-min-32-chars
HOST=http://localhost

# Optional: Use remote images instead of local builds
# METORIAL_API_IMAGE=ghcr.io/metorial/metorial-api:dev
# METORIAL_ENGINE_IMAGE=ghcr.io/metorial/metorial-mcp-engine-unified:dev
```

### Image Selection

By default, uses locally built images:
- `metorial-api:local`
- `metorial-engine:local`

To use remote images, set environment variables or edit docker-compose.yml.

## Service Endpoints

### Application
| Port | Service |
|------|---------|
| 4300 | Dashboard Frontend |
| 4310 | Core API |
| 4311 | MCP API |
| 4312 | Marketplace API |
| 4313 | OAuth Provider |
| 4314 | Private API |
| 4315 | Portals API |
| 4316 | Integrations API |
| 4321 | ID API |
| 50050 | Engine gRPC |

### Infrastructure
| Port | Service | Credentials |
|------|---------|-------------|
| 35432 | PostgreSQL | postgres/postgres |
| 36379 | Redis | (none) |
| 32707 | MongoDB | mongo/mongo |
| 37700 | Meilisearch | (none) |
| 9000 | MinIO | minio/minio123 |
| 9001 | MinIO Console | minio/minio123 |
| 9200 | OpenSearch | admin/admin |
| 5601 | OpenSearch Dashboards | admin/admin |

## First Login / Authentication

Metorial uses **self-registration** - there are no default credentials for the frontend.

### Creating Your First Account

1. Open `http://localhost:4300`
2. Click **"Sign up"**
3. Enter:
   - Name (display name)
   - Email (unique)
   - Password (minimum 8 characters)
4. You're logged in automatically

### Authentication API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/_/auth/signup` | POST | Create account (name, email, password) |
| `/_/auth/login` | POST | Login (email, password) |
| `/_/auth/logout` | POST | End session |

### Important Notes

- **No default admin account** - first user you create is the first user
- Each user gets their own organization and workspace automatically
- Infrastructure credentials (postgres, mongo, etc.) are separate from app login
- Password minimum: 8 characters

---

## Databases

PostgreSQL creates two databases:
- `postgres` - Main application database
- `engine` - MCP engine database

Database init script: `self-hosting/init-db/01-create-databases.sql`

## Docker Socket

The engine service mounts `/var/run/docker.sock` to spawn MCP server containers.

## File Locations

```
metorial-platform/self-hosting/
├── docker-compose.yml     # Main compose file
├── .env.example          # Environment template
├── .env                  # Your configuration (gitignored)
├── init-db/              # PostgreSQL init scripts
│   └── 01-create-databases.sql
└── .gitignore
```

---

## Engine Network Configuration

The engine uses separate listen and advertise addresses for Docker networking:

| Environment Variable | Default | Purpose |
|---------------------|---------|---------|
| `ENGINE_LISTEN_ADDRESS` | `:50050` | Bind address (all interfaces) |
| `ENGINE_ADVERTISE_ADDRESS` | `localhost:50050` | Address returned to API clients |
| `ENGINE_LAUNCHER_ADDRESS` | Derived from advertise | Launcher worker address |
| `ENGINE_REMOTE_ADDRESS` | Derived from advertise | Remote worker address |

**Docker Compose Configuration:**
```yaml
engine:
  environment:
    - ENGINE_LISTEN_ADDRESS=:50050
    - ENGINE_ADVERTISE_ADDRESS=engine:50050
    - ENGINE_LAUNCHER_ADDRESS=engine:50052
    - ENGINE_REMOTE_ADDRESS=engine:50053
```

**Why this matters:** The API connects to the engine using the address returned by `ListManagers`. If the engine advertises `localhost:50050`, the API (in a different container) cannot reach it. By advertising `engine:50050` (the Docker service name), cross-container communication works.

## Troubleshooting

### API cannot connect to Engine
**Symptom:** `Error checking manager localhost:50050: UNAVAILABLE`

**Cause:** Engine advertising localhost instead of Docker service name.

**Fix:** Ensure `ENGINE_ADVERTISE_ADDRESS=engine:50050` is set in docker-compose.yml.

### Frontend Shows Blank Page / "CORE_API_URL is not defined"
**Symptom:** Browser shows blank white page with console error `CORE_API_URL is not defined`

**Cause:** Vite bakes `VITE_*` environment variables into JavaScript at **build time**, not runtime. The frontend was built without the required environment variables.

**Fix:** Rebuild the frontend with correct environment:
```bash
# Ensure self-hosting/.env has correct HOST value
mise run selfhost:build-frontend
mise run selfhost:up
```

**Important:** The build script (`selfhost:build-frontend`) reads `HOST` from `self-hosting/.env` and exports required `VITE_*` variables before running the Vite build. Changing `HOST` requires rebuilding the frontend image.

**Required VITE_ variables** (set automatically by build script):
- `VITE_CORE_API_URL` - Core API endpoint
- `VITE_METORIAL_ENV` - Environment (production)
- `VITE_PRIVATE_API_URL` - Private API endpoint
- `VITE_MCP_API_URL` - MCP API endpoint

---

**Last Updated**: 2026-02-02
