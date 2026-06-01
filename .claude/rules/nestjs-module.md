# NestJS Module Conventions

**Purpose**: Standard module structure and organization for NestJS features

## Module File Structure

Every feature should follow this structure:

```
src/modules/[feature-name]/
├── [feature-name].module.ts          # Module definition
├── [feature-name].controller.ts      # HTTP endpoints
├── [feature-name].service.ts         # Business logic
├── dto/
│   ├── create-[feature-name].dto.ts
│   ├── update-[feature-name].dto.ts
│   └── query-[feature-name].dto.ts
├── entities/
│   └── [feature-name].entity.ts
├── repositories/
│   └── [feature-name].repository.ts  # Optional: custom repository
├── guards/
│   └── [feature-name].guard.ts       # Optional: authorization
├── decorators/
│   └── [feature-name].decorator.ts   # Optional: custom decorators
├── interfaces/
│   └── [feature-name].interface.ts   # Optional: type definitions
├── constants/
│   └── [feature-name].constant.ts    # Optional: constants
└── tests/
    ├── [feature-name].service.spec.ts
    ├── [feature-name].controller.spec.ts
    └── [feature-name].repository.spec.ts
```

## Naming Conventions

### Directories
- **Format**: kebab-case
- **Example**: `user-management`, `product-catalog`, `order-processing`
- **Rule**: All lowercase, hyphens between words

### Classes
- **Format**: PascalCase
- **Example**: `UserModule`, `UserService`, `UserController`
- **Rule**: Each word capitalized, no hyphens

### Files
- **Format**: kebab-case.type.ts
- **Examples**:
  - `user.module.ts`
  - `user.service.ts`
  - `user.controller.ts`
  - `create-user.dto.ts`
  - `user.entity.ts`
  - `user.repository.ts`

### Variables & Constants
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Example**: `const MAX_RETRIES = 3;`, `let userName = 'John';`

## Module Declaration

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { [FeatureName]Controller } from './[feature-name].controller';
import { [FeatureName]Service } from './[feature-name].service';
import { [FeatureName]Repository } from './repositories/[feature-name].repository';
import { [FeatureName]Entity } from './entities/[feature-name].entity';

@Module({
  imports: [TypeOrmModule.forFeature([FeatureNameEntity])],
  controllers: [[FeatureName]Controller],
  providers: [[FeatureName]Service, [FeatureName]Repository],
  exports: [[FeatureName]Service],
})
export class [FeatureName]Module {}
```

## Best Practices

✅ **DO**
- One module per feature
- Services export from modules
- Custom repositories for complex queries
- Separate guards for authorization
- Group related DTOs in dto folder
- Use dependency injection
- Type safety with TypeScript
- Write tests for business logic

❌ **DON'T**
- Business logic in controllers
- Direct database calls from controllers
- Circular dependencies between modules
- Skip DTOs for inputs
- Ignore error handling
- Hard-coded values
- Missing type annotations
- Skipping tests

## Module Registration

Register modules in `src/app.module.ts`:

```typescript
import { [FeatureName]Module } from './modules/[feature-name]/[feature-name].module';

@Module({
  imports: [
    // ... other modules
    [FeatureName]Module,
  ],
})
export class AppModule {}
```
