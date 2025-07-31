# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Admin is a monorepo containing three packages:

- **server**: NestJS backend with GraphQL API and Prisma ORM
- **dashboard**: Next.js frontend with internationalization (i18n)
- **app**: React Native mobile application with Expo

## Package-Specific Documentation

For detailed guidance on each package, refer to the local CLAUDE.md files:

- **[Server Package](./packages/server/CLAUDE.md)**: Backend development guide (NestJS, GraphQL, Prisma)
- **[Dashboard Package](./packages/dashboard/CLAUDE.md)**: Frontend development guide (Next.js, React, Chakra UI)

## Development Commands

### From root directory:

```bash
# Install dependencies for all workspaces
yarn install

# Run both dashboard and server concurrently
yarn dev

# Run individual packages
yarn dev:dashboard    # Start Next.js dashboard
yarn dev:server      # Start NestJS server
yarn dev:app         # Start React Native app

# Build commands
yarn build           # Build both server and dashboard
yarn build:server    # Build server only
yarn build:dashboard # Build dashboard only

# Linting
yarn lint            # Lint both server and dashboard
yarn lint:server     # Lint server code
yarn lint:dashboard  # Lint dashboard code

# Testing
yarn test:server     # Run server tests
```

### Package-specific commands:

**Server (packages/server):**

```bash
npm run test         # Run unit tests
npm run test:watch   # Run tests in watch mode
npm run test:cov     # Run tests with coverage
npm run test:e2e     # Run e2e tests
npm run seed         # Seed database with test data
```

**Dashboard (packages/dashboard):**

```bash
npm run lint:translations  # Lint translation files
```

**App (packages/app):**

```bash
npm run start        # Start Expo development server
npm run android      # Run on Android
npm run ios          # Run on iOS
npm run test         # Run tests
```

## Architecture Overview

### Backend (NestJS + GraphQL)

- **GraphQL API**: Schema-first approach with auto-generated schema at `src/schema.gql`
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Firebase Auth integration with JWT tokens
- **Modules**: Organized by feature (auth, user, project, crew, activity, timesheet)
- **Guards**: Role-based access control (RBAC) with custom decorators

### Frontend (Next.js)

- **Routing**: App Router with internationalization support (en/es)
- **State Management**: Apollo Client for GraphQL
- **UI Framework**: Chakra UI with Tailwind CSS
- **Authentication**: Firebase Auth integration
- **Form Handling**: React Hook Form

### Mobile App (React Native + Expo)

- **Navigation**: Expo Router
- **State Management**: Apollo Client
- **Authentication**: Firebase Auth via @react-native-firebase

## Database Schema

Key entities:

- Organization: Company/tenant model
- User: System users with roles (ADMIN, MEMBER, LEADER, CREW, MANAGER, READONLY)

## GraphQL Integration

- Server generates schema automatically
- Frontend uses Apollo Client with type generation from schema
- Queries/Mutations organized in hooks (e.g., `useGetProjects`, `useCreateProject`)

## Authentication Flow

1. User authenticates with Firebase
2. Firebase token sent to backend
3. Backend validates token and creates/updates user
4. JWT token issued for subsequent requests
5. Guards protect GraphQL resolvers based on roles

## Important Notes

- Never ask to run any of the projects using the dev command, assume the user is already running the projects in development mode
- For package-specific guidance, always refer to the local CLAUDE.md files in each package directory
- All development should follow the patterns and conventions outlined in the package-specific documentation

## Development Best Practices

- Always use yarn instead of npm
