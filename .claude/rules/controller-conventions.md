# Controller Conventions

**Purpose**: HTTP endpoint standards and API design

## Controller Template

```typescript
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Patch,
  Body,
  Param,
  Query,
  HttpCode,
  HttpStatus,
  UseGuards,
  UseInterceptors,
  Logger,
} from '@nestjs/common';
import { ApiBearerAuth, ApiOperation, ApiResponse, ApiTags } from '@nestjs/swagger';
import { [FeatureName]Service } from './[feature-name].service';
import { Create[FeatureName]Dto, Update[FeatureName]Dto, Query[FeatureName]Dto } from './dto';
import { [FeatureName]Entity } from './entities/[feature-name].entity';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';

@ApiTags('[feature-names]')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard)
@Controller('[feature-names]')
export class [FeatureName]Controller {
  private readonly logger = new Logger([FeatureName]Controller.name);

  constructor(private readonly service: [FeatureName]Service) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({ summary: 'Create new [feature-name]' })
  @ApiResponse({
    status: 201,
    description: 'Successfully created',
    type: [FeatureName]Entity,
  })
  async create(@Body() dto: Create[FeatureName]Dto): Promise<[FeatureName]Entity> {
    this.logger.debug(`Creating [feature-name]: ${JSON.stringify(dto)}`);
    return this.service.create(dto);
  }

  @Get()
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Get all [feature-names]' })
  @ApiResponse({
    status: 200,
    description: 'List of [feature-names]',
    type: [FeatureName]Entity,
    isArray: true,
  })
  async findAll(
    @Query() query: Query[FeatureName]Dto,
  ): Promise<{
    data: [FeatureName]Entity[];
    total: number;
    page: number;
    limit: number;
  }> {
    return this.service.findAll(query);
  }

  @Get(':id')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Get [feature-name] by ID' })
  @ApiResponse({
    status: 200,
    description: 'Found [feature-name]',
    type: [FeatureName]Entity,
  })
  @ApiResponse({
    status: 404,
    description: 'Not found',
  })
  async findOne(@Param('id') id: string): Promise<[FeatureName]Entity> {
    return this.service.findOne(+id);
  }

  @Put(':id')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Update [feature-name]' })
  @ApiResponse({
    status: 200,
    description: 'Successfully updated',
    type: [FeatureName]Entity,
  })
  async update(
    @Param('id') id: string,
    @Body() dto: Update[FeatureName]Dto,
  ): Promise<[FeatureName]Entity> {
    return this.service.update(+id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  @ApiOperation({ summary: 'Delete [feature-name]' })
  @ApiResponse({
    status: 204,
    description: 'Successfully deleted',
  })
  async remove(@Param('id') id: string): Promise<void> {
    return this.service.remove(+id);
  }
}
```

## REST Endpoint Naming

```typescript
// Collection endpoints
GET    /api/v1/[resources]              // List all
POST   /api/v1/[resources]              // Create new

// Item endpoints
GET    /api/v1/[resources]/:id          // Get one
PUT    /api/v1/[resources]/:id          // Replace (full update)
PATCH  /api/v1/[resources]/:id          // Partial update
DELETE /api/v1/[resources]/:id          // Delete

// Nested resources
GET    /api/v1/[resources]/:id/[sub-resources]
POST   /api/v1/[resources]/:id/[sub-resources]

// Actions on resources
POST   /api/v1/[resources]/:id/activate
POST   /api/v1/[resources]/:id/deactivate
```

## HTTP Status Codes

```typescript
// 2xx Success
HttpStatus.OK = 200                    // GET, PUT, PATCH success
HttpStatus.CREATED = 201               // POST success
HttpStatus.NO_CONTENT = 204            // DELETE, or empty response
HttpStatus.ACCEPTED = 202              // Async operation accepted

// 4xx Client errors
HttpStatus.BAD_REQUEST = 400            // Invalid request
HttpStatus.UNAUTHORIZED = 401           // Missing authentication
HttpStatus.FORBIDDEN = 403              // No permission
HttpStatus.NOT_FOUND = 404              // Resource not found
HttpStatus.CONFLICT = 409               // Resource conflict
HttpStatus.UNPROCESSABLE_ENTITY = 422   // Validation error

// 5xx Server errors
HttpStatus.INTERNAL_SERVER_ERROR = 500  // Server error
HttpStatus.SERVICE_UNAVAILABLE = 503    // Service unavailable
```

## Swagger/OpenAPI Decorators

```typescript
// Endpoint documentation
@ApiOperation({ summary: 'Short description' })
@ApiTags('feature')

// Response documentation
@ApiResponse({
  status: 200,
  description: 'Success',
  type: EntityClass,
  isArray: true,
})

// Request body documentation
@ApiBody({ type: CreateDto })

// Parameter documentation
@ApiParam({ name: 'id', type: 'number', example: 1 })
@ApiQuery({ name: 'page', type: 'number', example: 1 })

// Security
@ApiBearerAuth()
@ApiSecurity('api_key')
```

## Error Handling

```typescript
import { HttpExceptionFilter } from './filters/http-exception.filter';

// Apply globally in main.ts
app.useGlobalFilters(new HttpExceptionFilter());

// Or apply to controller
@UseFilters(HttpExceptionFilter)
@Controller('items')
export class ItemController {}
```

## Guards and Interceptors

```typescript
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { TransformInterceptor } from '../interceptors/transform.interceptor';

@UseGuards(JwtAuthGuard)
@UseInterceptors(TransformInterceptor)
@Controller('items')
export class ItemController {}

// Or on specific methods
@Get()
@UseGuards(JwtAuthGuard)
async findAll() {}
```

## Query Parameters

```typescript
// Pagination
@Query('page') page: number = 1
@Query('limit') limit: number = 10

// Sorting
@Query('sortBy') sortBy: string = 'id'
@Query('sortOrder') sortOrder: 'ASC' | 'DESC' = 'DESC'

// Filtering
@Query('search') search: string
@Query('status') status: string

// Use Query DTO for validation
@Query() query: QueryDto
```

## Best Practices

✅ **DO**
- Keep controllers thin (no business logic)
- Use proper HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Return appropriate status codes
- Document with Swagger decorators
- Use DTOs for validation
- Use guards for authorization
- Log important operations
- Handle errors in services, not controllers
- Use route parameters for IDs
- Use query parameters for filtering/pagination

❌ **DON'T**
- Put business logic in controllers
- Use GET for mutations
- Return 200 for all responses
- Skip API documentation
- Validate at controller level
- Expose database errors directly
- Use custom error status codes
- Return raw database entities
- Mix authentication with business logic
- Create side effects in GET requests
