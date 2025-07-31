# Server Package - CLAUDE.md

This file provides guidance specifically for the NestJS backend server.

## Package Commands

```bash
yarn test         # Run unit tests
yarn test:watch   # Run tests in watch mode
yarn test:cov     # Run tests with coverage
yarn test:e2e     # Run e2e tests
yarn seed         # Seed database with test data
```

## Server Architecture

### Backend (NestJS + GraphQL)

- **GraphQL API**: Schema-first approach with auto-generated schema at `src/schema.gql`
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Firebase Auth integration with JWT tokens
- **Modules**: Organized by feature (auth, user, project, crew, activity, timesheet)
- **Guards**: Role-based access control (RBAC) with custom decorators

### Directory Structure

```
packages/server/
├── src/
│   ├── [feature]/           # Feature modules
│   │   ├── input/          # GraphQL input types
│   │   ├── model/          # GraphQL object types
│   │   ├── [feature].module.ts
│   │   ├── [feature].resolver.ts
│   │   ├── [feature].service.ts
│   │   └── *.spec.ts       # Unit tests
│   ├── database/           # Prisma integration
│   ├── decorators/         # Custom decorators
│   ├── guards/             # Auth & role guards
│   ├── app.module.ts       # Root module
│   ├── main.ts            # Application entry
│   └── schema.gql         # Auto-generated GraphQL schema
├── prisma/
│   ├── schema.prisma      # Database schema
│   ├── migrations/        # Database migrations
│   └── seed.ts           # Database seeding
└── test/                 # E2E tests
```

## Creating a New Feature Module

1. **Create module directory** `src/[feature]/` with subdirectories:

```bash
mkdir -p src/feature/input src/feature/model
```

2. **Create the module file** (`[feature].module.ts`):

```typescript
import { Module } from "@nestjs/common";
import { PrismaModule } from "../database/primsa.module";
import { FeatureResolver } from "./feature.resolver";
import { FeatureService } from "./feature.service";

@Module({
  imports: [PrismaModule],
  providers: [FeatureResolver, FeatureService],
  exports: [FeatureService], // If needed by other modules
})
export class FeatureModule {}
```

3. **Register module** in `app.module.ts`:

```typescript
imports: [
  // ... other modules
  FeatureModule,
];
```

## Creating a Service

Create `[feature].service.ts`:

```typescript
import { BadRequestException, Injectable, Logger } from "@nestjs/common";
import { CurrentUserEntity } from "../auth/entity/current-user.entity";
import { PrismaService } from "../database/prisma.service";
import { CreateFeatureInput } from "./input/create-feature.input";
import { UpdateFeatureInput } from "./input/update-feature.input";
import { Feature } from "./model/feature.model";

@Injectable()
export class FeatureService {
  private readonly logger = new Logger(FeatureService.name);

  constructor(private readonly repository: PrismaService) {}

  async getFeatures(organizationId: string): Promise<Feature[]> {
    try {
      const features = await this.repository.feature.findMany({
        where: { organizationId },
        include: {
          // Include relations as needed
        },
        orderBy: { createdAt: "desc" },
      });

      return features;
    } catch (error) {
      this.logger.error(`Error getting features: ${error.message}`);
      throw new BadRequestException(error.message);
    }
  }

  async getFeature(id: string, organizationId: string): Promise<Feature> {
    try {
      const feature = await this.repository.feature.findFirstOrThrow({
        where: { id, organizationId },
        include: {
          // Include relations as needed
        },
      });

      return feature;
    } catch (error) {
      this.logger.error(`Error getting feature ${id}: ${error.message}`);
      throw new BadRequestException(error.message);
    }
  }

  async createFeature(
    input: CreateFeatureInput,
    user: CurrentUserEntity
  ): Promise<string> {
    try {
      this.logger.log(`Creating new feature: ${input.name}`);

      // Validate related entities exist
      // Create feature
      const feature = await this.repository.feature.create({
        data: {
          ...input,
          organizationId: user.organizationId,
          createdById: user.id,
        },
      });

      this.logger.log(`Created feature with id ${feature.id}`);
      return "ok";
    } catch (error) {
      this.logger.error(`Error creating feature: ${error.message}`);
      throw new BadRequestException(error.message);
    }
  }

  async updateFeature(
    input: UpdateFeatureInput,
    user: CurrentUserEntity
  ): Promise<string> {
    try {
      this.logger.log(`Updating feature ${input.id}`);

      // Verify feature exists and belongs to organization
      await this.repository.feature.findFirstOrThrow({
        where: {
          id: input.id,
          organizationId: user.organizationId,
        },
      });

      // Build update object dynamically
      const updateData = {};
      if (input.name) updateData["name"] = input.name;
      // Add other fields as needed

      await this.repository.feature.update({
        where: { id: input.id },
        data: updateData,
      });

      this.logger.log(`Updated feature ${input.id}`);
      return "ok";
    } catch (error) {
      this.logger.error(`Error updating feature: ${error.message}`);
      throw new BadRequestException(error.message);
    }
  }
}
```

## Creating GraphQL Types

1. **Input Types** (`input/create-feature.input.ts`):

```typescript
import { Field, InputType } from "@nestjs/graphql";

@InputType()
export class CreateFeatureInput {
  @Field()
  name: string;

  @Field({ nullable: true })
  description?: string;

  @Field(() => [String], { defaultValue: [] })
  tags: string[];
}
```

2. **Object Types** (`model/feature.model.ts`):

```typescript
import { Field, ObjectType } from "@nestjs/graphql";
import { User } from "../../user/model/user.model";

@ObjectType()
export class Feature {
  @Field()
  id: string;

  @Field()
  name: string;

  @Field({ nullable: true })
  description?: string;

  @Field(() => User)
  createdBy: User;

  @Field()
  createdAt: Date;

  @Field()
  updatedAt: Date;
}
```

## Authentication & Authorization

1. **Guards are applied at resolver level**:

   - `AuthGuard`: Validates JWT token and populates user
   - `RoleGuard`: Checks if user has required roles

2. **Access current user** in resolvers:

```typescript
@CurrentUser() user: CurrentUserEntity
```

3. **Role-based access**:

```typescript
@Roles(Role.ADMIN, Role.MANAGER) // At class level
@UseGuards(AuthGuard, RoleGuard)
```

## Database Operations

1. **Always use Prisma through the service**:

```typescript
this.repository.model.findMany();
this.repository.model.create();
this.repository.model.update();
this.repository.model.delete();
```

2. **Multi-tenancy**: Always filter by `organizationId`
3. **Error handling**: Wrap operations in try-catch blocks
4. **Logging**: Use NestJS Logger for debugging

## Testing

1. **Unit tests** (`[feature].service.spec.ts`):

```typescript
describe("FeatureService", () => {
  let service: FeatureService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [FeatureService, PrismaService],
    }).compile();

    service = module.get<FeatureService>(FeatureService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  // Test cases...
});
```

2. **Run tests**:

```bash
yarn test              # All tests
yarn test:watch       # Watch mode
yarn test:cov         # Coverage
yarn test feature     # Specific feature
```

## Common Patterns

1. **Return "ok" for mutations** that don't need to return data
2. **Use transactions** for complex operations:

```typescript
await this.repository.$transaction(async (tx) => {
  // Multiple operations
});
```

3. **Validate entity ownership** before updates/deletes
4. **Include relations** as needed in queries
5. **Sort results** consistently (usually by createdAt)
6. **Handle errors gracefully** with proper logging
7. Never ask to run any of the projects using the dev command, assume the user is already running the projects in development mode.

## Prisma Enum Usage

When using Prisma enums in GraphQL:

1. **Import from Prisma**: Import enum types directly from `@prisma/client`
2. **Register with GraphQL**: Create a separate file to register the Prisma enum with GraphQL:

   ```typescript
   import { registerEnumType } from "@nestjs/graphql";
   import { EnumName } from "@prisma/client";

   registerEnumType(EnumName, {
     name: "EnumName",
     description: "Description of the enum",
   });
   ```

3. **Import registration**: Import the enum registration file in the resolver to ensure it's registered

## Prisma Relations

When updating related fields in Prisma:

- Use `connect` syntax for relations: `{ connect: { id: userId } }`
- Not direct assignment: ~~`{ userId: userId }`~~
