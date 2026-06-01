# Repository Conventions

**Purpose**: Data access layer standards and database query best practices

## Custom Repository Template

```typescript
import { Injectable } from '@nestjs/common';
import { DataSource, Repository } from 'typeorm';
import { [FeatureName]Entity } from '../entities/[feature-name].entity';

@Injectable()
export class [FeatureName]Repository extends Repository<[FeatureName]Entity> {
  constructor(private dataSource: DataSource) {
    super(
      [FeatureName]Entity,
      dataSource.createEntityManager(),
    );
  }

  /**
   * Find active [feature-names] with pagination
   */
  async findActivePaginated(
    page: number = 1,
    limit: number = 10,
  ): Promise<{ data: [FeatureName]Entity[]; total: number }> {
    const [data, total] = await this.findAndCount({
      where: { isActive: true },
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return { data, total };
  }

  /**
   * Search [feature-names] by keyword
   */
  async searchByKeyword(keyword: string): Promise<[FeatureName]Entity[]> {
    return this.createQueryBuilder('[feature]')
      .where('[feature].name ILIKE :keyword', { keyword: `%${keyword}%` })
      .orWhere('[feature].email ILIKE :keyword', { keyword: `%${keyword}%` })
      .orderBy('[feature].createdAt', 'DESC')
      .getMany();
  }

  /**
   * Get with related entities
   */
  async findOneWithRelations(id: number): Promise<[FeatureName]Entity | null> {
    return this.createQueryBuilder('[feature]')
      .leftJoinAndSelect('[feature].posts', 'posts')
      .leftJoinAndSelect('[feature].comments', 'comments')
      .where('[feature].id = :id', { id })
      .getOne();
  }

  /**
   * Find with complex filter
   */
  async findByFilter(filter: {
    status?: string;
    minDate?: Date;
    maxDate?: Date;
    search?: string;
  }): Promise<[FeatureName]Entity[]> {
    let query = this.createQueryBuilder('[feature]');

    if (filter.status) {
      query = query.andWhere('[feature].status = :status', {
        status: filter.status,
      });
    }

    if (filter.minDate && filter.maxDate) {
      query = query.andWhere(
        '[feature].createdAt BETWEEN :minDate AND :maxDate',
        {
          minDate: filter.minDate,
          maxDate: filter.maxDate,
        },
      );
    }

    if (filter.search) {
      query = query.andWhere('[feature].name ILIKE :search', {
        search: `%${filter.search}%`,
      });
    }

    return query.orderBy('[feature].createdAt', 'DESC').getMany();
  }

  /**
   * Count by status
   */
  async countByStatus(status: string): Promise<number> {
    return this.count({ where: { status } });
  }
}
```

## Query Builder Patterns

### Basic Queries
```typescript
// Find all
const all = await this.repository.find();

// Find with condition
const active = await this.repository.find({
  where: { isActive: true },
});

// Find with relations
const withRelations = await this.repository.find({
  relations: ['posts', 'comments'],
  where: { isActive: true },
});

// Find with pagination
const paginated = await this.repository.find({
  skip: 0,
  take: 10,
});

// Find with sorting
const sorted = await this.repository.find({
  order: { createdAt: 'DESC' },
});
```

### Query Builder
```typescript
// Basic query builder
const users = await this.repository
  .createQueryBuilder('user')
  .where('user.isActive = :isActive', { isActive: true })
  .getMany();

// With joins
const usersWithPosts = await this.repository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .where('user.isActive = :isActive', { isActive: true })
  .getMany();

// With aggregation
const result = await this.repository
  .createQueryBuilder('user')
  .select('user.status', 'status')
  .addSelect('COUNT(user.id)', 'count')
  .groupBy('user.status')
  .getRawMany();

// With subquery
const activeUsersWithPosts = await this.repository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .where(
    'user.id IN (' +
    this.repository
      .createQueryBuilder('u')
      .select('u.id')
      .where('u.isActive = true')
      .getQuery() +
    ')',
  )
  .getMany();
```

### Pagination with Query Builder
```typescript
async paginate(
  page: number = 1,
  limit: number = 10,
  sortBy: string = 'createdAt',
  sortOrder: 'ASC' | 'DESC' = 'DESC',
): Promise<{ data: Entity[]; total: number; page: number; limit: number }> {
  const [data, total] = await this.repository
    .createQueryBuilder('entity')
    .orderBy(`entity.${sortBy}`, sortOrder)
    .skip((page - 1) * limit)
    .take(limit)
    .getManyAndCount();

  return {
    data,
    total,
    page,
    limit,
  };
}
```

## N+1 Query Prevention

### Problem: N+1 Queries
```typescript
// ❌ N+1 problem - loads users, then posts individually
const users = await this.userRepository.find();
for (const user of users) {
  user.posts = await this.postRepository.find({
    where: { userId: user.id },
  });
}
```

### Solution: Eager Loading
```typescript
// ✅ Single query with join
const users = await this.userRepository.find({
  relations: ['posts'],
});
```

### Solution: Query Builder
```typescript
// ✅ Using QueryBuilder with leftJoinAndSelect
const users = await this.userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .getMany();
```

## Best Practices

✅ **DO**
- Use TypeORM query builder for complex queries
- Implement custom repositories for reusable queries
- Add indexes on frequently queried columns
- Use eager loading for relationships
- Paginate large result sets
- Filter sensitive data at database level
- Use transactions for multi-table operations
- Document complex queries
- Cache frequently accessed data
- Monitor query performance

❌ **DON'T**
- Load all data then filter in code
- Create N+1 queries
- Skip pagination on large datasets
- Use hardcoded limit values
- Skip relationship configuration
- Perform heavy calculations in database
- Use raw queries when ORM provides solution
- Expose sensitive fields
- Skip error handling

## Performance Tips

1. **Use Indexes**
   ```sql
   CREATE INDEX idx_email ON users(email);
   CREATE INDEX idx_created_at ON users(created_at DESC);
   ```

2. **Projection (Select specific columns)**
   ```typescript
   const users = await this.repository
     .createQueryBuilder('user')
     .select(['user.id', 'user.name', 'user.email'])
     .getMany();
   ```

3. **Limit Results**
   ```typescript
   const recent = await this.repository.find({
     take: 10,
     order: { createdAt: 'DESC' },
   });
   ```

4. **Avoid Circular References**
   - Load only what you need
   - Use depth limiting in relations
