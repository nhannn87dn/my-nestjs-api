# Configuration Conventions

**Purpose**: Environment configuration and setup standards

## Environment Files

```
.env                    # Environment variables (local, not in git)
.env.example            # Template for required env variables
.env.development        # Development environment
.env.staging            # Staging environment
.env.production         # Production environment
```

## .env.example Template

```bash
# Application
NODE_ENV=development
PORT=3000
APP_URL=http://localhost:3000
APP_NAME=MyNestJsAPI

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=password
DB_NAME=mydb

# JWT
JWT_SECRET=your-secret-key-here
JWT_EXPIRATION=1h
JWT_REFRESH_SECRET=your-refresh-secret-key
JWT_REFRESH_EXPIRATION=7d

# Logging
LOG_LEVEL=debug

# CORS
CORS_ORIGIN=http://localhost:3000

# Rate Limiting
RATE_LIMIT_TTL=60000
RATE_LIMIT_MAX_REQUESTS=100

# Email (Optional)
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USER=user@example.com
MAIL_PASSWORD=password
MAIL_FROM=noreply@example.com

# Redis (Optional)
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
```

## Configuration Module

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [
        `.env.${process.env.NODE_ENV || 'development'}.local`,
        `.env.${process.env.NODE_ENV || 'development'}`,
        '.env.local',
        '.env',
      ],
      expandVariables: true,
    }),
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USER'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        autoLoadEntities: true,
        synchronize: configService.get('NODE_ENV') === 'development',
        logging: configService.get('LOG_LEVEL') === 'debug',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class ConfigurationModule {}
```

## Typed Configuration

```typescript
import { IsString, IsNumber, IsOptional, validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

class EnvironmentVariables {
  @IsString()
  NODE_ENV: string;

  @IsNumber()
  PORT: number;

  @IsString()
  DB_HOST: string;

  @IsNumber()
  DB_PORT: number;

  @IsString()
  DB_USER: string;

  @IsString()
  DB_PASSWORD: string;

  @IsString()
  DB_NAME: string;

  @IsString()
  JWT_SECRET: string;

  @IsOptional()
  @IsString()
  MAIL_HOST?: string;
}

export async function validateConfig(
  config: Record<string, unknown>,
): Promise<EnvironmentVariables> {
  const validatedConfig = plainToClass(EnvironmentVariables, config, {
    enableImplicitConversion: true,
  });

  const errors = await validate(validatedConfig);

  if (errors.length > 0) {
    throw new Error(
      `Config validation error: ${errors.map((e) => e.toString()).join(', ')}`,
    );
  }

  return validatedConfig;
}
```

## Database Configuration

```typescript
// src/config/database.config.ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';
import { ConfigService } from '@nestjs/config';

export const getDatabaseConfig = (
  configService: ConfigService,
): TypeOrmModuleOptions => ({
  type: 'postgres',
  host: configService.get('DB_HOST', 'localhost'),
  port: configService.get('DB_PORT', 5432),
  username: configService.get('DB_USER', 'postgres'),
  password: configService.get('DB_PASSWORD', 'password'),
  database: configService.get('DB_NAME', 'mydb'),
  entities: [__dirname + '/../modules/**/*.entity{.ts,.js}'],
  migrations: [__dirname + '/../database/migrations/*{.ts,.js}'],
  subscribers: [__dirname + '/../database/subscribers/*{.ts,.js}'],
  autoLoadEntities: true,
  synchronize: process.env.NODE_ENV === 'development',
  logging: process.env.LOG_LEVEL === 'debug',
  logger: 'advanced-console',
  maxQueryExecutionTime: 1000,
});
```

## JWT Configuration

```typescript
// src/config/jwt.config.ts
import { JwtModuleOptions } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';

export const getJwtConfig = (configService: ConfigService): JwtModuleOptions => ({
  secret: configService.get('JWT_SECRET'),
  signOptions: {
    expiresIn: configService.get('JWT_EXPIRATION', '1h'),
  },
});

export const getJwtRefreshConfig = (configService: ConfigService): JwtModuleOptions => ({
  secret: configService.get('JWT_REFRESH_SECRET'),
  signOptions: {
    expiresIn: configService.get('JWT_REFRESH_EXPIRATION', '7d'),
  },
});
```

## Best Practices

✅ **DO**
- Use environment variables for configuration
- Never commit .env files
- Provide .env.example template
- Validate configuration on startup
- Use typed configuration
- Document all configuration options
- Use defaults for optional values
- Separate config by environment
- Load config from ConfigService
- Never hardcode secrets

❌ **DON'T**
- Commit .env files to git
- Hardcode configuration values
- Skip validation
- Use loose typing for config
- Mix configuration across modules
- Load config multiple times
- Use global variables for config
