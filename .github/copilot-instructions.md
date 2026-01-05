# LibreChat AI Agent Instructions

## Project Overview
LibreChat is a comprehensive AI chat platform that integrates multiple AI providers (OpenAI, Azure, Anthropic, Google, AWS Bedrock, etc.) with advanced features including agents, code interpreter, web search, and multimodal capabilities. The project uses a monorepo structure with separate packages for frontend, backend, and shared libraries.

## Architecture & Structure

### Monorepo Layout
```
LibreChat/
├── api/                    # Backend (Express.js + MongoDB)
├── client/                # Frontend (React + Vite + TypeScript)
├── packages/              # Shared libraries
│   ├── data-provider/     # API client & React Query hooks
│   ├── data-schemas/      # Mongoose models & TypeScript types
│   ├── api/               # Shared API utilities
│   └── client/            # Shared UI components
├── e2e/                   # Playwright end-to-end tests
├── config/                # Configuration scripts
└── docker-compose.yml     # Local development setup
```

### Key Service Boundaries
- **Backend API**: Express.js server handling authentication, AI provider integrations, file management, and real-time streaming
- **Frontend**: React SPA with Recoil state management, React Query for data fetching, and Tailwind CSS
- **Data Provider**: Shared TypeScript library providing API clients, React Query hooks, and type definitions
- **Data Schemas**: MongoDB models and TypeScript schemas shared across the application

### Data Flow Patterns
1. **Client → API**: React Query hooks from `packages/data-provider` call API endpoints
2. **API → AI Providers**: Backend routes integrate with various AI services (OpenAI, Anthropic, etc.)
3. **Real-time Updates**: Server-Sent Events (SSE) for streaming responses
4. **State Management**: Recoil for client state, React Query for server state

## Critical Developer Workflows

### Local Development Setup
```bash
# 1. Install dependencies
npm install

# 2. Configure environment
cp .env.example .env
# Edit .env with your API keys

# 3. Start backend (from root)
npm run backend:dev

# 4. Start frontend (from root)
npm run frontend:dev

# 5. Or use Docker
docker-compose up -d
```

### Build & Package Management
```bash
# Build all packages (critical for monorepo)
npm run build:packages

# Build specific packages
npm run build:data-provider
npm run build:data-schemas
npm run build:client-package

# Build frontend
npm run frontend

# Using Bun (alternative package manager)
npm run b:client  # Builds everything with Bun
```

### Testing Workflows
```bash
# API tests
npm run test:api

# Client tests
npm run test:client

# Package tests
npm run test:packages:api
npm run test:packages:data-provider
npm run test:packages:data-schemas

# E2E tests (requires running backend)
npm run e2e

# E2E with UI
npm run e2e:headed

# Run all tests
npm run test:all
```

### Code Quality
```bash
# Linting
npm run lint
npm run lint:fix

# Formatting
npm run format

# Type checking (client)
cd client && npm run typecheck
```

## Project-Specific Conventions

### File Naming & Structure
- **Backend controllers**: `PascalCase.js` (e.g., `EndpointController.js`)
- **Frontend components**: `PascalCase.tsx` (e.g., `EndpointSettings.tsx`)
- **Hooks**: `useCamelCase.ts` (e.g., `useEndpoints.ts`)
- **Utility functions**: `camelCase.js` (e.g., `loadCustomConfig.js`)

### Import Patterns
- **Backend**: Use `~` alias for absolute imports (configured in `api/jsconfig.json`)
  ```javascript
  const { getEndpointsConfig } = require('~/server/services/Config');
  ```
- **Frontend**: Use `~` alias for absolute imports (configured in `client/tsconfig.json`)
  ```typescript
  import { useEndpoints } from '~/hooks/Endpoint';
  ```

### API Integration Patterns
- **Data Provider**: All API calls go through `packages/data-provider/src/data-service.ts`
- **React Query**: Use hooks from `packages/data-provider/src/react-query/react-query-service.ts`
- **Endpoint Configuration**: Defined in `librechat.yaml` and loaded via `api/server/services/Config/`

### Configuration Management
- **Environment Variables**: `.env` file (see `.env.example` for all options)
- **Custom Config**: `librechat.yaml` for AI endpoints, features, and UI customization
- **Docker**: `docker-compose.yml` for local development, `deploy-compose.yml` for production

### Testing Patterns
- **Unit Tests**: Jest with `*.spec.js` files alongside source
- **E2E Tests**: Playwright in `e2e/specs/` with TypeScript
- **Mocking**: Use `__mocks__` directories and Jest mocks
- **Setup**: Global setup in `api/test/jestSetup.js` and `e2e/setup/global-setup.ts`

## Integration Points & Dependencies

### External Services
- **MongoDB**: Primary database (default: `mongodb://127.0.0.1:27017/LibreChat`)
- **MeiliSearch**: Search engine (optional, default: `http://meilisearch:7700`)
- **Redis**: Caching (optional, configured via `@keyv/redis`)
- **RAG API**: Separate service for retrieval-augmented generation

### AI Provider Integrations
- **OpenAI/Azure**: Direct API integration
- **Anthropic**: Claude models via API
- **Google/Vertex AI**: Google's AI services
- **AWS Bedrock**: Amazon's model marketplace
- **Custom Endpoints**: OpenAI-compatible APIs

### MCP (Model Context Protocol)
- **Server Support**: MCP servers can be configured in `librechat.yaml`
- **Client Integration**: MCP tools appear as available tools in agents
- **Tool Discovery**: Automatic tool loading via `api/server/services/initializeMCPs.js`

### Authentication Strategies
- **Local**: Email/password with optional 2FA
- **OAuth2**: Google, GitHub, etc. (configured via `ALLOW_SOCIAL_LOGIN`)
- **LDAP**: Enterprise authentication
- **JWT**: Token-based auth for API calls

## Key Files & Directories

### Backend Core
- `api/server/index.js` - Main server entry point
- `api/server/routes/index.js` - All API routes
- `api/server/services/Config/` - Configuration loading
- `api/models/` - MongoDB models
- `api/server/controllers/` - Request handlers

### Frontend Core
- `client/src/App.jsx` - React app root
- `client/src/routes/` - React Router setup
- `client/src/components/` - UI components
- `client/src/hooks/` - Custom React hooks
- `client/src/store/` - Recoil state atoms

### Shared Packages
- `packages/data-provider/src/` - API client & types
- `packages/data-schemas/src/` - Mongoose models & schemas
- `packages/client/src/` - Shared UI components
- `packages/api/src/` - Shared backend utilities

### Configuration
- `librechat.yaml` - Main configuration file
- `.env` - Environment variables
- `docker-compose.yml` - Development setup
- `eslint.config.mjs` - Linting rules

## Common Development Scenarios

### Adding a New AI Provider
1. Add provider config to `librechat.yaml`
2. Implement integration in `api/server/services/Endpoints/`
3. Add provider UI components in `client/src/components/Endpoints/`
4. Update data types in `packages/data-provider/src/types/`

### Creating a New Agent Tool
1. Define tool schema in `packages/data-provider/src/types/agents.ts`
2. Implement tool logic in `api/app/clients/tools/`
3. Register tool in `api/server/services/Config/getCachedTools.js`
4. Add UI for tool configuration in `client/src/components/Agents/`

### Modifying API Endpoints
1. Update routes in `api/server/routes/`
2. Implement controllers in `api/server/controllers/`
3. Update data service in `packages/data-provider/src/data-service.ts`
4. Update React Query hooks in `packages/data-provider/src/react-query/`

### Frontend Component Changes
1. Create/update component in `client/src/components/`
2. Update hooks in `client/src/hooks/` if needed
3. Update routes in `client/src/routes/` if adding new pages
4. Ensure proper TypeScript types in `packages/data-provider/src/types/`

## Important Notes

### Performance Considerations
- **Build Times**: Package builds are sequential; use `npm run build:packages` for all
- **Dev Server**: Frontend dev server proxies to backend on port 3080
- **Caching**: Redis caching is optional but recommended for production
- **Database**: MongoDB indexes are auto-created; monitor for performance

### Security Considerations
- **Environment Variables**: Never commit `.env` files
- **API Keys**: Use environment variables, never hardcode
- **User Data**: All user data is isolated per user/session
- **File Uploads**: Stored locally by default; configure S3/Firebase for production

### Debugging Tips
- **Backend**: Use `npm run backend:inspect` for debugger support
- **Frontend**: Vite dev server provides hot reload and error overlays
- **API**: Check `logs/` directory for server logs
- **Database**: Use MongoDB Compass for visual inspection

### Deployment
- **Docker**: Use `docker-compose.yml` for development, `deploy-compose.yml` for production
- **Cloud**: One-click deploy options available (Railway, Zeabur, Sealos)
- **Environment**: Set `NODE_ENV=production` for production builds
- **Build**: Run `npm run frontend` before deploying

This guide covers the essential patterns and workflows for working with LibreChat. Always refer to the official documentation at https://docs.librechat.ai/ for detailed configuration options and advanced features.
