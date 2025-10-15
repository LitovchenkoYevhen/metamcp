# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MetaMCP is an MCP (Model Context Protocol) proxy that aggregates multiple MCP servers into unified endpoints with middleware support. It acts as an orchestrator, aggregator, and gateway for MCP servers, with a web UI for configuration management.

## Architecture

### Monorepo Structure
- **Monorepo**: Turborepo-based with pnpm workspaces
- **Backend** (`apps/backend`): Express.js server handling MCP proxying, tRPC API, and authentication
- **Frontend** (`apps/frontend`): Next.js 15 app with App Router, i18n support (en/zh)
- **Shared packages**:
  - `packages/trpc`: Shared tRPC router definitions
  - `packages/zod-types`: Shared validation schemas
  - `packages/typescript-config`: Shared TypeScript configs
  - `packages/eslint-config`: Shared ESLint configs

### Core Concepts
1. **MCP Server**: Configurations defining how to start MCP servers (STDIO, SSE, or STREAMABLE_HTTP)
2. **Namespace**: Groups one or more MCP servers; supports enabling/disabling servers or individual tools
3. **Endpoint**: Public routing endpoints that expose namespaces via SSE/Streamable HTTP/OpenAPI
4. **Middleware**: Functional middleware pattern for intercepting MCP requests/responses (similar to Express)

### Key Architecture Patterns

#### MetaMCP Proxy Flow
The backend creates a unified MCP server that aggregates multiple child servers:
- `apps/backend/src/lib/metamcp/metamcp-proxy.ts`: Core proxy implementation
- `apps/backend/src/lib/metamcp/mcp-server-pool.ts`: Manages pooled connections to MCP servers
- Tools/prompts/resources are prefixed with server name: `{serverName}__{originalName}`
- Supports middleware composition for filtering, logging, etc.

#### Middleware System
Located in `apps/backend/src/lib/metamcp/metamcp-middleware/`:
- **Functional middleware pattern**: Similar to Express.js middleware
- Built-in: `filter-tools.functional.ts` filters inactive tools based on namespace tool mappings
- Middleware can intercept list_tools and call_tool operations
- Uses composition pattern via `compose()` function

#### Database Schema
Located in `apps/backend/src/db/schema.ts`:
- **Multi-tenancy**: `user_id` fields with nullable for public resources
- **Repository pattern**: Separate repo files in `apps/backend/src/db/repositories/`
- **Serializers**: Convert DB models to API types in `apps/backend/src/db/serializers/`
- **Key tables**: mcp_servers, namespaces, endpoints, namespace_server_mappings, namespace_tool_mappings, api_keys

#### tRPC API
- Backend implementation: `apps/backend/src/trpc/*.impl.ts`
- Shared router definitions: `packages/trpc/src/routers/frontend/*.ts`
- Frontend client: `apps/frontend/lib/trpc.ts`
- Protected by session cookies for frontend, API keys for external access

## Development Commands

### Setup
```bash
pnpm install                    # Install dependencies
cp example.env .env            # Create env file (requires manual editing)
```

### Development
```bash
pnpm dev                       # Start all services in development mode (uses .env.local if present)
```

Frontend runs on port 12008, backend on port 12009.

### Building
```bash
pnpm build                     # Build all apps and packages
pnpm lint                      # Lint all packages
pnpm check-types               # Type-check all packages
```

### Backend-specific
```bash
cd apps/backend
pnpm dev                       # Dev mode with tsx watch
pnpm build                     # Production build with tsup
pnpm db:generate:dev           # Generate Drizzle migrations (uses .env.local)
pnpm db:migrate:dev            # Run Drizzle migrations (uses .env.local)
pnpm db:generate               # Generate migrations (uses .env)
pnpm db:migrate                # Run migrations (uses .env)
```

### Frontend-specific
```bash
cd apps/frontend
pnpm dev                       # Next.js dev server on port 12008
pnpm build                     # Next.js production build
pnpm check-types               # TypeScript type checking
```

## Database Migrations

Uses Drizzle ORM with PostgreSQL:
- Schema: `apps/backend/src/db/schema.ts`
- Config: `apps/backend/drizzle.config.ts`
- Generate migrations after schema changes: `pnpm db:generate:dev`
- Apply migrations: `pnpm db:migrate:dev`
- Migrations stored in `apps/backend/drizzle/`

## Authentication

- **Better Auth** for session management (cookie-based)
- **API Keys** for external endpoint access (format: `sk_mt_...`)
- **OIDC Support**: Optional OpenID Connect for enterprise SSO (configure via OIDC_* env vars)
- Auth routes handled at `/api/auth`
- Frontend auth client: `apps/frontend/lib/auth-client.ts`
- Backend auth: `apps/backend/src/auth.ts`

## Environment Variables

Required variables (see `example.env`):
- `DATABASE_URL`: PostgreSQL connection string
- `APP_URL`: Backend URL
- `NEXT_PUBLIC_APP_URL`: Frontend URL (must match APP_URL for CORS)
- `BETTER_AUTH_SECRET`: Auth session secret
- Optional OIDC variables: `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET`, `OIDC_DISCOVERY_URL`

## Frontend Routing Structure

Next.js App Router with i18n:
- `[locale]/(sidebar)/*`: Main authenticated app pages
  - `/mcp-servers`: List and manage MCP server configurations
  - `/mcp-servers/[uuid]`: Individual server with tool management
  - `/namespaces`: Namespace management
  - `/namespaces/[uuid]`: Namespace details with server/tool mappings
  - `/endpoints`: Endpoint management
  - `/api-keys`: API key management
  - `/mcp-inspector`: Test MCP connections
  - `/live-logs`: Real-time logging
  - `/search`: Search functionality
  - `/settings`: App settings
- `[locale]/login`, `[locale]/register`: Auth pages

## Key Backend Routes

- `/api/auth/*`: Better Auth endpoints
- `/metamcp/{endpointName}/sse`: SSE transport for MCP
- `/metamcp/{endpointName}/mcp`: Streamable HTTP transport for MCP
- `/metamcp/{endpointName}/openapi`: OpenAPI endpoint
- `/mcp-proxy/*`: Internal MCP proxy routes (authenticated)
- `/trpc/*`: tRPC API routes

## MCP Server Pool

The `mcpServerPool` (in `mcp-server-pool.ts`) manages:
- Connection pooling to MCP servers
- Idle session pre-allocation (default: 1 idle session per server)
- Session lifecycle and cleanup
- OAuth session management for MCP servers

## Code Style

- **Indentation**: Tabs (not spaces)
- **TypeScript**: Strict mode enabled
- **Imports**: Use workspace protocol for monorepo packages (`@repo/*`)
- **Async/await**: Preferred over promises
- **Error handling**: Try-catch with console.error for logging

## Internationalization

Frontend uses next-i18next with locale files in `apps/frontend/public/locales/{locale}/*.json`.
Supported locales: `en`, `zh`

## Docker Deployment

- Production: `docker compose up -d`
- Development: `docker compose -f docker-compose.dev.yml up`
- Custom Dockerfile may be needed for MCP servers with special dependencies
- SSE configuration required for nginx reverse proxies (see `nginx.conf.example`)

## Important Implementation Notes

- **Tool naming convention**: Tools are prefixed as `{sanitizedServerName}__{originalToolName}`
- **Middleware caching**: Tool status middleware uses in-memory cache with TTL (default 1s)
- **Server name sanitization**: Use `sanitizeName()` utility from `apps/backend/src/lib/metamcp/utils.ts`
- **Multi-tenancy**: Always check `user_id` context; null means public/shared resource
- **Session management**: Sessions are pooled and reused; cleanup is automatic
- **Idle session invalidation**: When updating server/namespace configs, idle sessions are invalidated (see `invalidation.md`)
