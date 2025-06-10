# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ShellHub - Centralized SSH Gateway

ShellHub is a microservices-based SSH access management platform that allows secure remote access to devices through a centralized gateway. The system supports web UI, native SSH clients, and container access.

## Development Setup

### Initial Setup
```bash
# Generate required cryptographic keys
make keygen

# Set development environment  
echo "SHELLHUB_ENV=development" >> .env.override

# Start all services
make start

# Create initial user and namespace
./bin/cli user create <username> <password> <email>
./bin/cli namespace create <namespace> <owner> 00000000-0000-4000-0000-000000000000
```

Access the web UI at http://localhost:8080 (development mode).

### Important Note
When you open ShellHub UI for the first time, be sure to accept the pending device.

### Common Development Commands

**Building Services:**
```bash
# Build all services
make build

# Build specific service
make build SERVICE=api
```

**Testing:**
```bash
# Run unit tests for all projects
./devscripts/test-unit api,ssh,agent,ui

# Run tests for specific service
./devscripts/test-unit api
./devscripts/test-unit ui

# Run UI tests with coverage
cd ui && npm run test-coverage

# Run UI tests with timezone support
cd ui && TZ=UTC npm run test

# Run integration tests
cd tests && go test -v
```

**Linting:**
```bash
# Lint all code
./devscripts/lint-code all

# Lint specific language
./devscripts/lint-code go
./devscripts/lint-code vue

# Auto-fix linting issues
./devscripts/lint-code all --fix
```

**UI Development:**
```bash
cd ui
npm run dev          # Development server
npm run build        # Production build
npm run lint-fix     # Fix linting issues
npm run test-ui      # Run tests with UI interface
```

**Development Scripts:**
```bash
# Add fake devices for testing
./devscripts/add-device

# Get devices from API
./devscripts/get-devices

# Generate/update mock objects for testing
./devscripts/gen-mock

# Run native agent with specific tag
./devscripts/run-agent <tag>

# Update Go version across the project
./devscripts/update-go <version>
```

## Architecture Overview

### Core Services

**API Service** (`/api`): Central REST API server built with Go + Echo framework
- Device and session management
- Authentication/authorization (JWT-based)
- Background job processing (Asynq + Redis)
- MongoDB for persistence, Redis for caching

**SSH Service** (`/ssh`): SSH gateway handling SSH protocol connections
- SSH authentication and session establishment
- Session recording and terminal access
- Reverse tunneling to agents via Redis

**UI Service** (`/ui`): Vue.js 3 + TypeScript web interface
- Device dashboard and management
- Real-time terminal (xterm.js + WebSockets)
- Session playback and monitoring

**Gateway Service** (`/gateway`): Nginx reverse proxy
- HTTP/HTTPS termination and routing
- Auto SSL with Let's Encrypt
- Static file serving

**Agent** (`/agent`): Client-side agent for managed devices
- Device registration and heartbeat
- SSH server mode and container connector
- Auto-update capabilities

**CLI** (`/cli`): Administrative command-line interface

### Data Layer
- **MongoDB**: Primary database (devices, sessions, users, namespaces)
- **Redis**: Caching, task queue, real-time communication

### Communication Patterns
- Services communicate via REST API calls to the API service
- Real-time events through Redis pub/sub
- WebSocket connections for terminal sessions
- Reverse tunneling for SSH connections

## Key File Locations

**Configuration:**
- `docker-compose.yml` - Production service definitions
- `docker-compose.dev.yml` - Development overrides
- `Makefile` - Build and development commands
- `.env.override` - Local environment overrides

**Go Services:**
- `api/` - Main API server and business logic
- `ssh/` - SSH gateway service
- `agent/` - Device agent
- `cli/` - Command-line interface
- `pkg/` - Shared Go libraries and utilities

**Frontend:**
- `ui/src/` - Vue.js application source
- `ui/package.json` - NPM dependencies and scripts
- `ui/vite.config.ts` - Build configuration

**Development Tools:**
- `devscripts/` - Utility scripts for development
- `bin/` - Docker Compose wrapper and CLI tools

**Environment Configuration:**
- `.env` - Default environment variables
- `.env.override` - Local overrides (not committed)
- Key environment variables:
  - `SHELLHUB_ENV`: Set to "development" for dev mode
  - `SHELLHUB_DOMAIN`: Domain for the instance
  - `SHELLHUB_HTTP_PORT`: HTTP port (default: 8080)
  - `SHELLHUB_SSH_PORT`: SSH gateway port (default: 2222)
  - `SHELLHUB_TUNNELS`: Enable tunnel feature
  - `SHELLHUB_ENTERPRISE`: Enable enterprise features

## Working with the Codebase

### Adding New Features
1. **API Changes**: Modify `api/routes/` for new endpoints, `api/services/` for business logic
2. **Database**: Add migrations in `api/store/mongo/migrations/`
3. **UI Components**: Add to `ui/src/components/` following Vue 3 + Composition API patterns
4. **Agent Features**: Modify `agent/` and update protocol communication

### Testing Strategy
- Go services use `testify` for unit tests
- UI uses Vitest + Vue Test Utils
- Integration tests in `tests/` directory
- Mock generation with `./devscripts/gen-mock`
- Tests require running services (use `make start` first)

### Container Development
- Use `./bin/docker-compose` wrapper instead of direct `docker-compose`
- Development containers include dev tools and live reload
- Services auto-restart on code changes in development mode

### Multi-tenancy
- All data is scoped by namespace/tenant_id
- Role-based access control with granular permissions
- Tenant isolation enforced at database and API layers

## Common Patterns

**Error Handling**: Standardized error responses across services using `pkg/errors`

**Authentication**: JWT tokens with public/private key cryptography, MFA support

**Background Jobs**: Async processing using Asynq (Redis-based task queue)

**Logging**: Structured logging with configurable levels and formats

**Database Queries**: MongoDB queries with pagination, filtering, and sorting utilities

**API Versioning**: RESTful API design with consistent response formats

**WebSocket Protocol**: Real-time communication for terminal sessions using gorilla/websocket

**Service Health Checks**: All services expose `/healthcheck` endpoints

**MongoDB Migrations**: Automatic migration system with versioned schema changes

**SSH Protocol**: Custom SSH server implementation using gliderlabs/ssh

## Docker Compose Variants

- `docker-compose.yml`: Base production configuration
- `docker-compose.dev.yml`: Development overrides with hot reload
- `docker-compose.enterprise.yml`: Enterprise features
- `docker-compose.test.yml`: Testing environment
- `docker-compose.agent.yml`: Agent-specific setup
- `docker-compose.autossl.yml`: Auto SSL with Let's Encrypt

## API Authentication

**Public API**: Uses JWT tokens with API keys or user credentials
**Internal API**: Service-to-service communication with pre-shared keys
**SSH Authentication**: Public key or password-based authentication

## Database Migrations

Migrations are automatically applied on API service startup. To add a new migration:
1. Create `migration_XX.go` in `api/store/mongo/migrations/`
2. Implement `Apply()` and `Version()` methods
3. Add corresponding test in `migration_XX_test.go`

## Contributing

Before submitting PRs:
1. Run all tests: `./devscripts/test-unit all`
2. Run linter: `./devscripts/lint-code all`
3. Test your changes in development mode
4. Fork and create feature branches
5. Follow the project's Code of Conduct