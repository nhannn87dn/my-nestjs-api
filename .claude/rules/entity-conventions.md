# Entity Conventions

**Purpose**: Standard entity structure and TypeORM decorators usage

## Entity Template

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  ManyToOne,
  OneToMany,
  Index,
} from 'typeorm';
import { ApiProperty } from '@nestjs/swagger';

@Entity('[entity-table-name]')
@Index('idx_unique_email', { unique: true, where: '"deletedAt" IS NULL' })
@Index('idx_created_at')
export class [FeatureName]Entity {
  @PrimaryGeneratedColumn()
  @ApiProperty({ example: 1 })
  id: number;

  @Column({ type: 'varchar', length: 255, unique: true })
  @ApiProperty({ example: 'example@email.com' })
  email: string;

  @Column({ type: 'varchar', length: 255 })
  @ApiProperty({ example: 'John Doe' })
  name: string;

  @Column({ type: 'varchar', length: 255, nullable: true })
  @ApiProperty({ example: 'Description', nullable: true })
  description?: string;

  @Column({ type: 'boolean', default: true })
  @ApiProperty({ example: true })
  isActive: boolean;

  @CreateDateColumn()
  @ApiProperty({ example: '2025-01-01T00:00:00Z' })
  createdAt: Date;

  @UpdateDateColumn()
  @ApiProperty({ example: '2025-01-01T00:00:00Z' })
  updatedAt: Date;

  @DeleteDateColumn({ nullable: true })
  @ApiProperty({ example: null, nullable: true })
  deletedAt?: Date;

  // Relationships
  @ManyToOne(() => [ParentEntity], parent => parent.children, {
    onDelete: 'CASCADE',
  })
  parent: [ParentEntity];

  @OneToMany(() => [ChildEntity], child => child.parent)
  children: [ChildEntity][];
}
```

## Column Types

### String Columns
```typescript
@Column({ type: 'varchar', length: 255 })           // Standard string
@Column({ type: 'text' })                           // Long text
@Column({ type: 'char', length: 2 })               // Fixed length
```

### Numeric Columns
```typescript
@Column({ type: 'int' })                           // Integer
@Column({ type: 'bigint' })                        // Big integer
@Column({ type: 'decimal', precision: 10, scale: 2 }) // Money
@Column({ type: 'float' })                         // Float
```

### Date Columns
```typescript
@CreateDateColumn()                                // Auto created at
@UpdateDateColumn()                                // Auto updated at
@DeleteDateColumn()                                // Soft delete
@Column({ type: 'timestamp' })                    // Custom timestamp
```

### Boolean Columns
```typescript
@Column({ type: 'boolean', default: false })      // Boolean
@Column({ type: 'boolean', default: true })       // With default
```

### JSON Columns
```typescript
@Column({ type: 'jsonb', nullable: true })        // PostgreSQL JSONB
@Column({ type: 'simple-json' })                  // Generic JSON
```

## Relationships

### One to Many
```typescript
@OneToMany(() => PostEntity, post => post.author)
posts: PostEntity[];
```

### Many to One
```typescript
@ManyToOne(() => UserEntity, user => user.posts, {
  onDelete: 'CASCADE',
  eager: true,  // Load automatically
})
author: UserEntity;

@Column()
authorId: number;
```

### Many to Many
```typescript
@ManyToMany(() => TagEntity, tag => tag.posts)
@JoinTable({
  name: 'post_tags',
  joinColumn: { name: 'post_id' },
  inverseJoinColumn: { name: 'tag_id' },
})
tags: TagEntity[];
```

## Indexes

```typescript
// Single column index
@Index('idx_email')
@Column()
email: string;

// Composite index
@Index('idx_user_status', ['id', 'status'])

// Unique index
@Index('idx_unique_email', { unique: true })

// Partial index (PostgreSQL)
@Index('idx_active_users', { where: '"isActive" = true' })
```

## Column Constraints

```typescript
// Not null & unique
@Column({ nullable: false, unique: true })
email: string;

// With default value
@Column({ default: true })
isActive: boolean;

// With comment
@Column({ comment: 'User email address' })
email: string;

// Readonly column
@Column({ update: false })
createdBy: string;
```

## Best Practices

✅ **DO**
- Add @ApiProperty decorators for Swagger
- Add indexes on frequently queried columns
- Use soft delete (@DeleteDateColumn) for audit trail
- Define relationships clearly
- Use appropriate data types
- Add column constraints
- Add indexes on foreign keys
- Document complex columns

❌ **DON'T**
- Expose internal IDs in API responses
- Store sensitive data unencrypted
- Create circular relationships without proper handling
- Skip relationships definition
- Use generic types for specific data
- Create too many indexes (performance)
- Skip database constraints
