# DTO Conventions

**Purpose**: Data Transfer Object structure and validation rules

## DTO Template

### Create DTO
```typescript
import {
  IsString,
  IsEmail,
  IsNotEmpty,
  IsOptional,
  MinLength,
  MaxLength,
  Matches,
  ValidateIf,
} from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { Transform, Type } from 'class-transformer';

export class Create[FeatureName]Dto {
  @IsEmail()
  @IsNotEmpty()
  @ApiProperty({ example: 'user@example.com' })
  email: string;

  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(255)
  @ApiProperty({ example: 'John Doe' })
  name: string;

  @IsString()
  @IsNotEmpty()
  @MinLength(8)
  @Matches(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[a-zA-Z\d@$!%*?&]{8,}$/,
    { message: 'Password too weak' },
  )
  @ApiProperty({
    example: 'SecurePass123!',
    description: 'Min 8 chars: uppercase, lowercase, number, special char',
  })
  password: string;

  @IsOptional()
  @IsString()
  @MaxLength(1000)
  @ApiPropertyOptional({ example: 'User description' })
  description?: string;

  @IsOptional()
  @Transform(({ value }) => value?.toLowerCase())
  @ApiPropertyOptional({ example: 'role' })
  role?: string;
}
```

### Update DTO
```typescript
import { PartialType } from '@nestjs/mapped-types';
import { Create[FeatureName]Dto } from './create-[feature-name].dto';

export class Update[FeatureName]Dto extends PartialType(
  Create[FeatureName]Dto,
) {}
```

### Query DTO
```typescript
import { IsOptional, IsNumber, Min, Max, IsString } from 'class-validator';
import { Type } from 'class-transformer';
import { ApiPropertyOptional } from '@nestjs/swagger';

export class Query[FeatureName]Dto {
  @IsOptional()
  @Type(() => Number)
  @Min(1)
  @ApiPropertyOptional({ example: 1, description: 'Page number' })
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @Min(1)
  @Max(100)
  @ApiPropertyOptional({ example: 10, description: 'Items per page' })
  limit?: number = 10;

  @IsOptional()
  @IsString()
  @ApiPropertyOptional({ example: 'name', description: 'Sort by field' })
  sortBy?: string = 'id';

  @IsOptional()
  @IsString()
  @ApiPropertyOptional({ example: 'ASC', description: 'Sort order' })
  sortOrder?: 'ASC' | 'DESC' = 'DESC';

  @IsOptional()
  @IsString()
  @ApiPropertyOptional({ example: 'search text' })
  search?: string;
}
```

## Validation Rules

### Common Validators
```typescript
// String validators
@IsString()           // Check if string
@IsNotEmpty()         // Check if not empty
@MinLength(5)         // Minimum length
@MaxLength(255)       // Maximum length
@Matches(/regex/)     // Match regex pattern

// Email & URL
@IsEmail()            // Valid email format
@IsUrl()              // Valid URL format

// Number validators
@IsNumber()           // Check if number
@Min(0)               // Minimum value
@Max(100)             // Maximum value
@IsPositive()         // Positive number
@IsNegative()         // Negative number

// Date validators
@IsDate()             // Valid date
@IsBefore()           // Before date
@IsAfter()            // After date

// Type validators
@IsBoolean()          // Boolean value
@IsArray()            // Array type
@IsEnum(MyEnum)       // Enum value

// Conditional validators
@IsOptional()         // Optional field
@ValidateIf(o => o.condition === true)
@ConditionalValidator()
```

## Transformers

```typescript
import { Transform, Type } from 'class-transformer';

// Trim whitespace
@Transform(({ value }) => value?.trim())
name: string;

// Convert to lowercase
@Transform(({ value }) => value?.toLowerCase())
email: string;

// Convert to number
@Type(() => Number)
page: number;

// Custom transformer
@Transform(({ value }) => {
  if (typeof value === 'string') {
    return value.split(',').map(v => v.trim());
  }
  return value;
})
tags: string[];
```

## Nested DTOs

```typescript
import { Type } from 'class-transformer';
import { ValidateNested } from 'class-validator';

export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

## Array Validation

```typescript
import { ArrayMinSize, ArrayMaxSize, ValidateNested } from 'class-validator';

// Array of strings
@IsArray()
@ArrayMinSize(1)
@ArrayMaxSize(10)
@IsString({ each: true })
tags: string[];

// Array of objects
@ValidateNested({ each: true })
@Type(() => ItemDto)
items: ItemDto[];
```

## Best Practices

✅ **DO**
- Use separate DTOs for create/update
- Use @ApiProperty for Swagger documentation
- Validate all inputs
- Transform inputs (trim, lowercase, etc.)
- Use descriptive error messages
- Type inputs correctly
- Document complex validation rules
- Use nested DTOs for complex objects

❌ **DON'T**
- Use entities directly as DTOs
- Skip validation decorators
- Expose internal fields
- Skip API documentation
- Mix validation concerns
- Use generic validation messages
- Skip type transformation
- Validate at controller level only
