# Testing Conventions

**Purpose**: Unit, integration, and end-to-end testing standards

## Test File Organization

```
src/modules/[feature]/tests/
├── [feature].service.spec.ts
├── [feature].controller.spec.ts
├── [feature].repository.spec.ts
└── fixtures/
    ├── [feature].mock.ts
    └── [feature].factory.ts
```

## Unit Test Template (Service)

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { [FeatureName]Service } from '../[feature-name].service';
import { [FeatureName]Entity } from '../entities/[feature-name].entity';
import { Create[FeatureName]Dto } from '../dto/create-[feature-name].dto';
import { NotFoundException, BadRequestException } from '@nestjs/common';

describe('[FeatureName]Service', () => {
  let service: [FeatureName]Service;
  let repository: any;

  const mockEntity = {
    id: 1,
    email: 'test@example.com',
    name: 'Test User',
    createdAt: new Date(),
    updatedAt: new Date(),
  };

  beforeEach(async () => {
    const mockRepository = {
      create: jest.fn(),
      save: jest.fn(),
      find: jest.fn(),
      findOne: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      softDelete: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        [FeatureName]Service,
        {
          provide: getRepositoryToken([FeatureName]Entity),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<[FeatureName]Service>([FeatureName]Service);
    repository = module.get(getRepositoryToken([FeatureName]Entity));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('create', () => {
    it('should create a new entity', async () => {
      const dto: Create[FeatureName]Dto = {
        email: 'test@example.com',
        name: 'Test User',
      };

      repository.findOne.mockResolvedValue(null);
      repository.create.mockReturnValue(dto);
      repository.save.mockResolvedValue(mockEntity);

      const result = await service.create(dto);

      expect(result).toEqual(mockEntity);
      expect(repository.create).toHaveBeenCalledWith(dto);
      expect(repository.save).toHaveBeenCalledWith(dto);
    });

    it('should throw BadRequestException if already exists', async () => {
      const dto: Create[FeatureName]Dto = {
        email: 'test@example.com',
        name: 'Test User',
      };

      repository.findOne.mockResolvedValue(mockEntity);

      await expect(service.create(dto)).rejects.toThrow(
        BadRequestException,
      );
    });
  });

  describe('findOne', () => {
    it('should return an entity by ID', async () => {
      repository.findOne.mockResolvedValue(mockEntity);

      const result = await service.findOne(1);

      expect(result).toEqual(mockEntity);
      expect(repository.findOne).toHaveBeenCalledWith({
        where: { id: 1 },
      });
    });

    it('should throw NotFoundException if not found', async () => {
      repository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(
        NotFoundException,
      );
    });
  });

  describe('update', () => {
    it('should update an entity', async () => {
      const updateDto = { name: 'Updated Name' };
      const updatedEntity = { ...mockEntity, ...updateDto };

      repository.findOne.mockResolvedValue(mockEntity);
      repository.save.mockResolvedValue(updatedEntity);

      const result = await service.update(1, updateDto);

      expect(result).toEqual(updatedEntity);
      expect(repository.save).toHaveBeenCalled();
    });
  });

  describe('remove', () => {
    it('should remove an entity', async () => {
      repository.findOne.mockResolvedValue(mockEntity);
      repository.softDelete.mockResolvedValue({ affected: 1 });

      await service.remove(1);

      expect(repository.softDelete).toHaveBeenCalledWith(1);
    });
  });
});
```

## Controller Test Template

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { [FeatureName]Controller } from '../[feature-name].controller';
import { [FeatureName]Service } from '../[feature-name].service';

describe('[FeatureName]Controller', () => {
  let controller: [FeatureName]Controller;
  let service: [FeatureName]Service;

  const mockEntity = {
    id: 1,
    email: 'test@example.com',
    name: 'Test User',
  };

  beforeEach(async () => {
    const mockService = {
      create: jest.fn().mockResolvedValue(mockEntity),
      findAll: jest.fn().mockResolvedValue({
        data: [mockEntity],
        total: 1,
        page: 1,
        limit: 10,
      }),
      findOne: jest.fn().mockResolvedValue(mockEntity),
      update: jest.fn().mockResolvedValue(mockEntity),
      remove: jest.fn().mockResolvedValue(undefined),
    };

    const module: TestingModule = await Test.createTestingModule({
      controllers: [[FeatureName]Controller],
      providers: [
        {
          provide: [FeatureName]Service,
          useValue: mockService,
        },
      ],
    }).compile();

    controller = module.get<[FeatureName]Controller>(
      [FeatureName]Controller,
    );
    service = module.get<[FeatureName]Service>([FeatureName]Service);
  });

  describe('POST /[feature-names]', () => {
    it('should create a new entity', async () => {
      const dto = {
        email: 'test@example.com',
        name: 'Test User',
      };

      const result = await controller.create(dto);

      expect(result).toEqual(mockEntity);
      expect(service.create).toHaveBeenCalledWith(dto);
    });
  });

  describe('GET /[feature-names]', () => {
    it('should return paginated list', async () => {
      const query = { page: 1, limit: 10 };
      const result = await controller.findAll(query);

      expect(result.data).toEqual([mockEntity]);
      expect(result.total).toBe(1);
    });
  });

  describe('GET /[feature-names]/:id', () => {
    it('should return an entity by ID', async () => {
      const result = await controller.findOne('1');

      expect(result).toEqual(mockEntity);
      expect(service.findOne).toHaveBeenCalledWith(1);
    });
  });
});
```

## Test Fixtures and Factories

```typescript
// Mock factory
export class [FeatureName]MockFactory {
  static create(overrides?: Partial<[FeatureName]Entity>): [FeatureName]Entity {
    return {
      id: 1,
      email: 'test@example.com',
      name: 'Test User',
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static createMany(count: number): [FeatureName]Entity[] {
    return Array.from({ length: count }, (_, i) =>
      this.create({ id: i + 1 }),
    );
  }
}

// Usage in tests
const entity = [FeatureName]MockFactory.create();
const entities = [FeatureName]MockFactory.createMany(5);
```

## E2E Testing Template

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../app.module';

describe('[FeatureName]E2E', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('POST /api/v1/[feature-names]', () => {
    it('should create a new entity', () => {
      const dto = {
        email: 'test@example.com',
        name: 'Test User',
        password: 'SecurePass123!',
      };

      return request(app.getHttpServer())
        .post('/api/v1/[feature-names]')
        .send(dto)
        .expect(201)
        .expect((res) => {
          expect(res.body.success).toBe(true);
          expect(res.body.data.email).toBe(dto.email);
        });
    });
  });

  describe('GET /api/v1/[feature-names]/:id', () => {
    it('should return 404 for non-existent entity', () => {
      return request(app.getHttpServer())
        .get('/api/v1/[feature-names]/999')
        .expect(404)
        .expect((res) => {
          expect(res.body.success).toBe(false);
          expect(res.body.error.code).toBe('NOT_FOUND');
        });
    });
  });
});
```

## Test Coverage Goals

- **Overall**: 80%+
- **Critical paths**: 100%
- **Services**: 90%+
- **Controllers**: 80%+
- **Repositories**: 85%+

## Best Practices

✅ **DO**
- Write tests for critical business logic
- Use descriptive test names
- Mock external dependencies
- Test both success and error cases
- Use factories for test data
- Test edge cases
- Keep tests focused and isolated
- Use beforeEach/afterEach for setup/cleanup
- Test error messages
- Mock repositories properly

❌ **DON'T**
- Test framework code
- Write tests without mocks
- Skip error case testing
- Create coupled tests
- Test implementation details
- Use hardcoded test data
- Write overly complex tests
- Skip setup/cleanup
- Test multiple concerns in one test
