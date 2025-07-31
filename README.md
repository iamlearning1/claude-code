## 🤖 Using Claude Code

This repository is configured for optimal use with [Claude Code](https://claude.ai/code), an AI-powered development assistant. The `.claude/` directory contains specialized agents and commands that enhance Claude's ability to work with this codebase.

### Getting Started with Claude Code

1. **Open in Claude Code**: Load this repository in Claude Code
2. **Review the setup**: Claude will automatically read the `.claude/` configuration
3. **Use specialized agents**: Access expert knowledge for different aspects of the stack
4. **Run custom commands**: Use predefined workflows for common tasks

### Available Agents

The `.claude/agents/` directory contains specialized AI agents for different domains:

#### 🏗️ **Architecture & Backend**

- **`aws-solutions-architect`** - AWS infrastructure design and optimization
- **`aws-devops-engineer`** - CI/CD, IaC, monitoring, and DevOps practices
- **`backend-architect`** - REST/GraphQL API design and microservices
- **`nestjs-architect`** - Clean architecture with NestJS and dependency injection
- **`graphql-architect`** - GraphQL schema design and performance optimization

#### 🎨 **Frontend & Mobile**

- **`frontend-engineer`** - Next.js, React, shadcn/ui, and Tailwind CSS
- **`ui-ux-designer`** - Design systems, accessibility, and user experience

#### 🗄️ **Database**

- **`postgresql-architect`** - PostgreSQL optimization, queries, and schema design
- **`mongodb-architect`** - NoSQL document design and aggregation pipelines

#### 🛡️ **Quality & Security**

- **`security-auditor`** - Security reviews, authentication, and OWASP compliance
- **`test-automation-expert`** - TDD/BDD, testing strategies, and quality assurance

#### 📚 **Documentation**

- **`technical-documentation-expert`** - API docs, architecture docs, and user guides

### How to Use Agents

Simply mention an agent in your conversation with Claude Code:

```
@aws-solutions-architect Can you help me design a scalable architecture for this app?

@nestjs-architect Review this service for clean architecture principles

@security-auditor Audit this authentication flow for vulnerabilities
```

### Custom Commands

The `.claude/commands/` directory contains predefined workflows:

#### 🎫 **Issue Management**

```bash
# Create a new GitHub issue
@claude /create-issue "Add user profile settings page"

# Implement a GitHub issue end-to-end
@claude /feature #123
```

The `/feature` command provides a complete development workflow:

1. Fetches issue details from GitHub
2. Creates a feature branch
3. Implements the changes
4. Runs tests and quality checks
5. Updates documentation
6. Creates a pull request

### Project Structure

```
├── .claude/                    # Claude Code configuration
│   ├── agents/                # Specialized AI agents
│   └── commands/              # Custom workflows
├── packages/
│   ├── server/                # NestJS + GraphQL + Prisma
│   ├── dashboard/             # Next.js + React + Chakra UI
├── CLAUDE.md                  # Project guidance for Claude
└── README.md                  # This file
```

Each package has its own `CLAUDE.md` file with specific guidance:

- **[Server Guide](./packages/server/CLAUDE.md)** - Backend development with NestJS
- **[Dashboard Guide](./packages/dashboard/CLAUDE.md)** - Frontend development with Next.js

## 🤝 Development Workflow with Claude Code

### 1. Feature Development

Use the custom `/feature` command for end-to-end feature implementation:

```bash
@claude /feature #123
```

This will:

- Analyze the GitHub issue
- Create a feature branch
- Implement the necessary changes
- Write and run tests
- Create a pull request

### 2. Code Reviews

Ask Claude to review your code using specialized agents:

```bash
@security-auditor Review this authentication middleware for vulnerabilities

@nestjs-architect Check if this service follows clean architecture principles

@test-automation-expert Design tests for this new feature
```

### 3. Architecture Decisions

Get expert guidance on architectural choices:

```bash
@aws-solutions-architect Should we use ECS or Lambda for this microservice?

@postgresql-architect How should we optimize this database query?

@graphql-architect Design a GraphQL schema for user management
```

### 4. Documentation

Generate and maintain documentation:

```bash
@technical-documentation-expert Create API documentation for this endpoint

@ui-ux-designer Document this design system component
```
