# Service Conventions

**Purpose**: Business logic layer standards and best practices

## Service Template

```typescript
import { Injectable, NotFoundException, BadRequestException, InternalServerErrorException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { [FeatureName]Repository } from './repositories/[feature-name].repository';
import { [FeatureName]Entity } from './entities/[feature-name].entity';
import { Create[FeatureName]Dto, Update[FeatureName]Dto, Query[FeatureName]Dto } from './dto';

@Injectable()
export class [FeatureName]Service {
  constructor(
    @InjectRepository([FeatureName]Entity)
    private readonly repository: [FeatureName]Repository,
  ) {}

  /**
   * Create a new [feature-name]
   * @param dto - Create DTO
   * @returns Created entity
   * @throws BadRequestException if validation fails
   */
  async create(dto: Create[FeatureName]Dto): Promise<[FeatureName]Entity> {
    try {
      // Check if already exists
      const existing = await this.repository.findOne({
        where: { email: dto.email },
      });

      if (existing) {
        throw new BadRequestException('Already exists');
      }

      const entity = this.repository.create(dto);
      return await this.repository.save(entity);
    } catch (error) {
      if (error instanceof BadRequestException) {
        throw error;
      }
      throw new InternalServerErrorException('Failed to create');
    }
  }

  /**
   * Find all with pagination
   * @param query - Query DTO with pagination
   * @returns Paginated results
   */
  async findAll(query: Query[FeatureName]Dto): Promise<{
    data: [FeatureName]Entity[];
    total: number;
    page: number;
    limit: number;
  }> {
    const { page = 1, limit = 10, sortBy = 'id', sortOrder = 'DESC', search } = query;

    let qb = this.repository.createQueryBuilder('[feature]');

    // Add search filter
    if (search) {
      qb = qb.where('[feature].email LIKE :search', {
        search: `%${search}%`,
      });
    }

    const total = await qb.getCount();

    // Add pagination and sorting
    const data = await qb
      .orderBy(`[feature].${sortBy}`, sortOrder)
      .skip((page - 1) * limit)
      .take(limit)
      .getMany();

    return {
      data,
      total,
      page,
      limit,
    };
  }

  /**
   * Find one by ID
   * @param id - Entity ID
   * @returns Entity
   * @throws NotFoundException if not found
   */
  async findOne(id: number): Promise<[FeatureName]Entity> {
    const entity = await this.repository.findOne({
      where: { id },
    });

    if (!entity) {
      throw new NotFoundException(`Not found with id ${id}`);
    }

    return entity;
  }

  /**
   * Update by ID
   * @param id - Entity ID
   * @param dto - Update DTO
   * @returns Updated entity
   * @throws NotFoundException if not found
   */
  async update(
    id: number,
    dto: Update[FeatureName]Dto,
  ): Promise<[FeatureName]Entity> {
    const entity = await this.findOne(id);

    try {
      Object.assign(entity, dto);
      return await this.repository.save(entity);
    } catch (error) {
      throw new InternalServerErrorException('Failed to update');
    }
  }

  /**
   * Delete by ID (soft delete)
   * @param id - Entity ID
   * @throws NotFoundException if not found
   */
  async remove(id: number): Promise<void> {
    const entity = await this.findOne(id);
    await this.repository.softRemove(entity);
  }

  /**
   * Restore soft deleted entity
   * @param id - Entity ID
   */
  async restore(id: number): Promise<[FeatureName]Entity> {
    await this.repository.restore(id);
    return this.findOne(id);
  }
}
```

## Service Best Practices

### Error Handling
```typescript
// ✅ Use specific NestJS exceptions
import {
  NotFoundException,
  BadRequestException,
  ConflictException,
  UnauthorizedException,
  ForbiddenException,
  InternalServerErrorException,
} from '@nestjs/common';

// Handle different error types
if (!entity) {
  throw new NotFoundException('Entity not found');
}

if (entity.isLocked) {
  throw new ConflictException('Entity is locked');
}

if (!hasPermission) {
  throw new ForbiddenException('No permission');
}
```

### Async Operations
```typescript
// ✅ Always use async/await
async create(dto: CreateDto): Promise<Entity> {
  try {
    const entity = this.repository.create(dto);
    return await this.repository.save(entity);
  } catch (error) {
    throw new InternalServerErrorException();
  }
}

// ❌ Don't use .then().catch()
```

### Transactions
```typescript
import { DataSource } from 'typeorm';

constructor(private dataSource: DataSource) {}

async complexOperation(): Promise<void> {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();

  try {
    // Multiple operations
    await queryRunner.manager.save(entity1);
    await queryRunner.manager.save(entity2);
    await queryRunner.commitTransaction();
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

### Dependency Injection
```typescript
// ✅ Use constructor injection
constructor(
  @InjectRepository(UserEntity)
  private userRepository: Repository<UserEntity>,
  private configService: ConfigService,
) {}

// ❌ Don't create instances directly
const service = new SomeService(); // Wrong
```

## Common Patterns

### Pagination
```typescript
async findAllPaginated(
  page: number = 1,
  limit: number = 10,
): Promise<{ data: T[]; total: number; page: number; limit: number }> {
  const [data, total] = await this.repository.findAndCount({
    skip: (page - 1) * limit,
    take: limit,
  });

  return { data, total, page, limit };
}
```

### Soft Delete
```typescript
// Mark as deleted
async softDelete(id: number): Promise<void> {
  await this.repository.softDelete(id);
}

// Restore
async restore(id: number): Promise<void> {
  await this.repository.restore(id);
}

// Permanently delete
async hardDelete(id: number): Promise<void> {
  await this.repository.remove(await this.findOne(id));
}
```

### Caching
```typescript
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class CachedService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  async findOneWithCache(id: number): Promise<Entity> {
    const cacheKey = `entity:${id}`;
    const cached = await this.cacheManager.get<Entity>(cacheKey);

    if (cached) {
      return cached;
    }

    const entity = await this.repository.findOne({ where: { id } });
    await this.cacheManager.set(cacheKey, entity, 60000); // 1 minute
    return entity;
  }
}
```

## Best Practices Summary

✅ **DO**
- Handle all errors explicitly
- Use async/await
- Inject dependencies via constructor
- Document public methods
- Use TypeORM query builder for complex queries
- Implement proper error messages
- Use transactions for multiple operations
- Cache frequently accessed data
- Validate business logic
- Return typed responses

❌ **DON'T**
- Expose database errors directly
- Use callback hell
- Create instances with `new`
- Skip error handling
- Hard-code configuration
- Return raw database entities
- Perform I/O operations in loops
- Skip validation
