# Response Conventions

**Purpose**: Standardized API response format and error handling

## Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2025-01-01T00:00:00Z"
  },
  "message": "Operation successful",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

### List Response (Paginated)
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Item 1"
    },
    {
      "id": 2,
      "name": "Item 2"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "totalPages": 3
  },
  "message": "Retrieved successfully",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "password",
        "message": "Password too short"
      }
    ]
  },
  "statusCode": 400,
  "timestamp": "2025-01-01T00:00:00Z"
}
```

## Response Classes

### Base Response DTO
```typescript
export class BaseResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  message?: string;
  statusCode?: number;
  timestamp: Date;
}
```

### Paginated Response DTO
```typescript
export class PaginatedResponse<T> {
  success: boolean;
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
  message: string;
  timestamp: Date;
}
```

## Interceptor for Response Formatting

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => {
        // Check if data already has pagination
        if (data && data.data && data.total) {
          return {
            success: true,
            data: data.data,
            pagination: {
              page: data.page,
              limit: data.limit,
              total: data.total,
              totalPages: Math.ceil(data.total / data.limit),
            },
            message: 'Retrieved successfully',
            timestamp: new Date(),
          };
        }

        // Single item response
        return {
          success: true,
          data,
          message: 'Operation successful',
          timestamp: new Date(),
        };
      }),
    );
  }
}
```

## Error Handling Filter

```typescript
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    response.status(status).json({
      success: false,
      error: {
        code: this.getErrorCode(status),
        message:
          (exceptionResponse as any)?.message || exception.message,
        details: (exceptionResponse as any)?.details || null,
      },
      path: request.url,
      statusCode: status,
      timestamp: new Date(),
    });
  }

  private getErrorCode(status: number): string {
    switch (status) {
      case HttpStatus.BAD_REQUEST:
        return 'BAD_REQUEST';
      case HttpStatus.UNAUTHORIZED:
        return 'UNAUTHORIZED';
      case HttpStatus.FORBIDDEN:
        return 'FORBIDDEN';
      case HttpStatus.NOT_FOUND:
        return 'NOT_FOUND';
      case HttpStatus.CONFLICT:
        return 'CONFLICT';
      case HttpStatus.UNPROCESSABLE_ENTITY:
        return 'VALIDATION_ERROR';
      case HttpStatus.INTERNAL_SERVER_ERROR:
        return 'INTERNAL_SERVER_ERROR';
      default:
        return 'ERROR';
    }
  }
}
```

## Error Codes

```typescript
export enum ErrorCode {
  // Validation errors
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  INVALID_EMAIL = 'INVALID_EMAIL',
  INVALID_PASSWORD = 'INVALID_PASSWORD',
  WEAK_PASSWORD = 'WEAK_PASSWORD',

  // Authentication errors
  UNAUTHORIZED = 'UNAUTHORIZED',
  INVALID_CREDENTIALS = 'INVALID_CREDENTIALS',
  TOKEN_EXPIRED = 'TOKEN_EXPIRED',
  TOKEN_INVALID = 'TOKEN_INVALID',

  // Authorization errors
  FORBIDDEN = 'FORBIDDEN',
  NO_PERMISSION = 'NO_PERMISSION',
  ROLE_REQUIRED = 'ROLE_REQUIRED',

  // Resource errors
  NOT_FOUND = 'NOT_FOUND',
  RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND',

  // Conflict errors
  ALREADY_EXISTS = 'ALREADY_EXISTS',
  EMAIL_ALREADY_REGISTERED = 'EMAIL_ALREADY_REGISTERED',
  DUPLICATE_ENTRY = 'DUPLICATE_ENTRY',

  // Server errors
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  DATABASE_ERROR = 'DATABASE_ERROR',
  EXTERNAL_SERVICE_ERROR = 'EXTERNAL_SERVICE_ERROR',
}
```

## Best Practices

✅ **DO**
- Use consistent response format
- Include pagination metadata for lists
- Return appropriate HTTP status codes
- Include error codes for programmatic handling
- Document response format in Swagger
- Include timestamps in responses
- Provide clear error messages
- Include relevant details in error responses
- Use interceptors for consistent formatting
- Validate before responding

❌ **DON'T**
- Return 200 for all responses
- Expose internal error details
- Include sensitive information in responses
- Use inconsistent response structures
- Return null for errors
- Skip error codes
- Include database errors directly
- Return raw exception messages
