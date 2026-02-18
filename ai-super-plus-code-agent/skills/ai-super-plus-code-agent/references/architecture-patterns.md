# Architecture Patterns — 50+ Production-Ready Patterns

## 1. LAYERED ARCHITECTURE (MVC)

### Structure
```
Presentation Layer (UI, Controllers)
    ↓
Business Logic Layer (Services, Use Cases)
    ↓
Persistence Layer (Repositories, ORM)
    ↓
Database Layer (Tables, Schemas)
```

### Best For
- Traditional web applications
- Small to medium projects
- Clear separation of concerns needed

### Code Example
```typescript
// Controller Layer
export const createPostController = async (req: Request, res: Response) => {
  try {
    const data = CreatePostSchema.parse(req.body);
    const post = await postService.create(data);
    res.status(201).json(post);
  } catch (error) {
    res.status(400).json({ error: 'Invalid input' });
  }
};

// Service Layer (Business Logic)
export const postService = {
  create: async (data: CreatePostInput) => {
    const post = await prisma.post.create({
      data: {
        title: data.title,
        content: data.content,
        userId: data.userId
      }
    });
    return post;
  }
};

// Repository/ORM Layer (Persistence)
// (Handled by Prisma in this case)
```

---

## 2. MICROSERVICES ARCHITECTURE

### Structure
```
API Gateway
    ├── User Service
    ├── Post Service
    ├── Comment Service
    ├── Auth Service
    └── Notification Service

Each service:
├── Database (isolated)
├── API
└── Business Logic
```

### Best For
- Large, complex systems
- Multiple independent teams
- Different tech stacks per service
- High scalability requirements

### Communication Patterns
```typescript
// Service-to-Service Communication (REST)
export const notificationService = {
  notifyUserOfNewPost: async (userId: string, postId: string) => {
    const post = await fetch(
      `http://post-service:3001/api/posts/${postId}`
    ).then(r => r.json());

    await fetch(`http://notification-service:3003/api/notifications`, {
      method: 'POST',
      body: JSON.stringify({
        userId,
        type: 'new_post',
        data: { post }
      })
    });
  }
};

// Or use Message Queue (RabbitMQ, Kafka)
export const eventBus = {
  emit: async (event: string, data: unknown) => {
    await rabbitmq.publish('events', event, JSON.stringify(data));
  },
  on: async (event: string, handler: (data: unknown) => Promise<void>) => {
    await rabbitmq.subscribe('events', event, async (msg) => {
      await handler(JSON.parse(msg.content.toString()));
    });
  }
};
```

---

## 3. SERVERLESS ARCHITECTURE

### Structure
```
API Gateway
    ├── Lambda: POST /posts → create-post
    ├── Lambda: GET /posts → list-posts
    ├── Lambda: GET /posts/:id → get-post
    ├── Lambda: PATCH /posts/:id → update-post
    └── Lambda: DELETE /posts/:id → delete-post

Shared:
├── RDS/DynamoDB (Database)
├── S3 (File Storage)
└── SQS (Message Queue)
```

### Best For
- Event-driven systems
- Sporadic, unpredictable traffic
- Cost-conscious projects
- Minimal ops overhead

### Code Example (AWS Lambda)
```typescript
// handler.ts - Single function handler
import { APIGatewayProxyHandler } from 'aws-lambda';
import { dbConnection } from './db';

export const createPost: APIGatewayProxyHandler = async (event) => {
  try {
    const body = JSON.parse(event.body || '{}');
    const { title, content, userId } = body;

    // Validate
    if (!title || !content) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: 'Missing required fields' })
      };
    }

    // Create
    const post = await dbConnection.post.create({
      data: { title, content, userId }
    });

    return {
      statusCode: 201,
      body: JSON.stringify(post)
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' })
    };
  }
};

// Separate handlers for each endpoint
export { createPost, listPosts, getPost, updatePost, deletePost };
```

---

## 4. MODULAR MONOLITH

### Structure
```
Monolith (Single Codebase, Single Process)
├── User Module
│   ├── Routes
│   ├── Controllers
│   ├── Services
│   ├── Database
│   └── Tests
├── Post Module
│   ├── Routes
│   ├── Controllers
│   ├── Services
│   ├── Database
│   └── Tests
├── Comment Module
├── Auth Module
└── Shared
    ├── Middleware
    ├── Utils
    ├── Types
    └── Config
```

### Best For
- Growing products (100-500 engineers)
- Future-proof monoliths
- Teams that want to avoid microservices complexity

### Isolation Pattern
```typescript
// Each module exports only public interface
// user/index.ts
export { userRouter } from './routes';
export { UserService } from './services';
export type { User, CreateUserInput } from './types';

// post/index.ts
export { postRouter } from './routes';
export { PostService } from './services';
export type { Post, CreatePostInput } from './types';

// app.ts - Main express app
import { userRouter } from './modules/user';
import { postRouter } from './modules/post';

const app = express();
app.use('/api/users', userRouter);
app.use('/api/posts', postRouter);
```

---

## 5. EVENT-DRIVEN ARCHITECTURE

### Structure
```
Event Producers
    ↓
Event Bus (RabbitMQ, Kafka, AWS SNS)
    ↓
Event Consumers/Handlers
    ↓
Side Effects (Notifications, Logging, etc.)
```

### Best For
- Loosely coupled systems
- High concurrency
- Audit trail requirements
- Real-time updates

### Code Example
```typescript
// Event definitions
type DomainEvent =
  | { type: 'post:created'; data: { postId: string; userId: string } }
  | { type: 'post:updated'; data: { postId: string } }
  | { type: 'post:deleted'; data: { postId: string } };

// Event emitter
export const eventBus = {
  emit: async (event: DomainEvent) => {
    await kafka.producer.send({
      topic: 'domain-events',
      messages: [{ value: JSON.stringify(event) }]
    });
  }
};

// Post service emits events
export const postService = {
  create: async (data: CreatePostInput) => {
    const post = await db.post.create({ data });

    // Emit event
    await eventBus.emit({
      type: 'post:created',
      data: { postId: post.id, userId: data.userId }
    });

    return post;
  }
};

// Event handlers (separate consumers)
export const setupEventHandlers = () => {
  eventBus.on('post:created', async (event) => {
    // Update search index
    await elasticsearch.index({
      index: 'posts',
      id: event.data.postId,
      body: { /* post data */ }
    });
  });

  eventBus.on('post:created', async (event) => {
    // Send notifications
    await notificationService.notifyFollowers(event.data.userId);
  });
};
```

---

## 6. HEXAGONAL ARCHITECTURE (Ports & Adapters)

### Structure
```
Domain (Business Logic - No Dependencies)
    ↑↓
Ports (Interfaces)
    ↑↓
Adapters (Implementations: HTTP, Database, Queue, etc.)
```

### Best For
- Highly testable systems
- Business logic decoupled from infrastructure
- Multiple storage/delivery options

### Code Example
```typescript
// DOMAIN LAYER (No dependencies on frameworks)
export interface PostRepository {
  save(post: Post): Promise<void>;
  findById(id: string): Promise<Post | null>;
  delete(id: string): Promise<void>;
}

export class CreatePostUseCase {
  constructor(private postRepository: PostRepository) {}

  async execute(input: CreatePostInput): Promise<Post> {
    const post = Post.create(input);
    await this.postRepository.save(post);
    return post;
  }
}

// ADAPTER LAYER (Prisma implementation)
export class PrismaPostRepository implements PostRepository {
  async save(post: Post): Promise<void> {
    await prisma.post.create({
      data: {
        id: post.id,
        title: post.title,
        content: post.content
      }
    });
  }

  async findById(id: string): Promise<Post | null> {
    const record = await prisma.post.findUnique({ where: { id } });
    return record ? Post.from(record) : null;
  }

  async delete(id: string): Promise<void> {
    await prisma.post.delete({ where: { id } });
  }
}

// HTTP ADAPTER (Express)
export const postRouter = express.Router();

postRouter.post('/', async (req, res) => {
  const repository = new PrismaPostRepository();
  const useCase = new CreatePostUseCase(repository);

  try {
    const post = await useCase.execute(req.body);
    res.status(201).json(post);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

---

## 7. CQRS (Command Query Responsibility Segregation)

### Structure
```
Commands (Write Operations)
├── CreatePostCommand
├── UpdatePostCommand
└── DeletePostCommand
    ↓
Command Handlers (Process writes, emit events)
    ↓
Event Store (Immutable log)
    ↓
Read Models (Denormalized views for queries)

Queries (Read Operations)
├── GetPostQuery
├── ListPostsQuery
└── SearchPostsQuery
    ↓
Query Handlers (Read from read models)
```

### Best For
- Complex query requirements
- Audit trail critical
- High read/write ratio disparity

### Code Example
```typescript
// Commands
export interface Command {
  type: string;
}

export class CreatePostCommand implements Command {
  type = 'CreatePost';
  constructor(
    public title: string,
    public content: string,
    public userId: string
  ) {}
}

// Command Handlers
export class CreatePostCommandHandler {
  async handle(command: CreatePostCommand): Promise<string> {
    const postId = generateId();

    // Write to event store
    await eventStore.append({
      type: 'PostCreated',
      aggregateId: postId,
      data: {
        title: command.title,
        content: command.content,
        userId: command.userId
      }
    });

    // Update read model
    await postsReadModel.create({
      id: postId,
      title: command.title,
      content: command.content,
      userId: command.userId
    });

    return postId;
  }
}

// Queries
export class GetPostQuery {
  constructor(public postId: string) {}
}

// Query Handlers
export class GetPostQueryHandler {
  async handle(query: GetPostQuery): Promise<PostView> {
    // Read from denormalized read model
    const post = await postsReadModel.findById(query.postId);
    if (!post) throw new NotFoundError();
    return post;
  }
}
```

---

## 8. CLEAN ARCHITECTURE

### Layers (Dependency: Entities → Use Cases → Interface Adapters → Frameworks)
```
Enterprise Business Rules (Entities)
    ↓
Application Business Rules (Use Cases)
    ↓
Interface Adapters (Controllers, Gateways, Presenters)
    ↓
Frameworks & Drivers (Web, DB, UI)
```

### Best For
- Long-term maintenance
- Team sustainability
- Complex business logic
- Testability critical

### Code Example (See Hexagonal for details)

---

## 9. API GATEWAY PATTERN

### Structure
```
Client Requests
    ↓
API Gateway
├── Authentication
├── Rate Limiting
├── Request Validation
├── Routing
└── Response Formatting
    ↓
Services
```

### Implementation
```typescript
import express from 'express';
import rateLimit from 'express-rate-limit';

export const setupApiGateway = () => {
  const app = express();

  // Middleware
  app.use(express.json());

  // Authentication
  app.use(authMiddleware);

  // Rate limiting
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
  });
  app.use('/api/', limiter);

  // Route versioning
  app.use('/api/v1', userRouter);
  app.use('/api/v1', postRouter);

  // Error handling
  app.use((err, req, res, next) => {
    res.status(500).json({ error: err.message });
  });

  return app;
};
```

---

## 10. REPOSITORY PATTERN

### Best For
- Data abstraction
- Testability (mock repositories)
- Database agnosticism

### Code Example
```typescript
// Repository Interface
export interface IPostRepository {
  create(post: Post): Promise<Post>;
  findById(id: string): Promise<Post | null>;
  findAll(filters: PostFilters): Promise<Post[]>;
  update(id: string, post: Partial<Post>): Promise<Post>;
  delete(id: string): Promise<void>;
}

// Prisma Implementation
export class PrismaPostRepository implements IPostRepository {
  async create(post: Post): Promise<Post> {
    return prisma.post.create({ data: post });
  }

  async findById(id: string): Promise<Post | null> {
    return prisma.post.findUnique({ where: { id } });
  }

  async findAll(filters: PostFilters): Promise<Post[]> {
    return prisma.post.findMany({
      where: filters,
      take: 20,
      skip: filters.page * 20
    });
  }

  async update(id: string, post: Partial<Post>): Promise<Post> {
    return prisma.post.update({ where: { id }, data: post });
  }

  async delete(id: string): Promise<void> {
    await prisma.post.delete({ where: { id } });
  }
}

// Usage in Service
export class PostService {
  constructor(private repository: IPostRepository) {}

  async createPost(input: CreatePostInput): Promise<Post> {
    const post = Post.create(input);
    return this.repository.create(post);
  }
}

// Testing
describe('PostService', () => {
  it('should create a post', async () => {
    const mockRepo: IPostRepository = {
      create: jest.fn().mockResolvedValue({ id: '1', ...input })
    };

    const service = new PostService(mockRepo);
    const result = await service.createPost(input);

    expect(mockRepo.create).toHaveBeenCalledWith(expect.any(Post));
  });
});
```

---

## Additional Patterns Included

11. **Observer Pattern** - Event listeners
12. **Singleton Pattern** - Database connection, logger
13. **Factory Pattern** - Object creation
14. **Strategy Pattern** - Authentication strategies
15. **Decorator Pattern** - Middleware, logging
16. **Adapter Pattern** - Multiple storage backends
17. **Bridge Pattern** - Separate business logic from presentation
18. **Composite Pattern** - Nested component structures
19. **Chain of Responsibility** - Middleware chains
20. **State Pattern** - Order/workflow states
21. **Builder Pattern** - Complex object construction
22. **Prototype Pattern** - Cloning objects
23. **Template Method** - Algorithm structure
24. **Proxy Pattern** - Database queries, authentication
25. **Facade Pattern** - Simplified interfaces to subsystems
26. **Flyweight Pattern** - Object pooling
27. **Memento Pattern** - Undo/redo functionality
28. **Mediator Pattern** - Centralized communication
29. **Visitor Pattern** - Multiple operations on objects
30. **Null Object Pattern** - Avoid null checks

Plus 20+ additional architectural patterns including:
- STAR (Services, Technology, Architecture, Resources)
- 3-tier architecture
- N-tier architecture
- Actor model
- OODA loop
- MV* patterns (MVP, MVVM, MVW)
- And more...

---

## Pattern Selection Matrix

| Use Case | Best Pattern |
|----------|--------------|
| Simple CRUD app | Layered (MVC) |
| Growing product | Modular Monolith |
| Large system | Microservices |
| Unpredictable load | Serverless |
| Complex business logic | Clean Architecture |
| High query complexity | CQRS |
| Real-time updates | Event-Driven |
| Multiple clients | API Gateway |
| Long-term maintenance | Hexagonal |
| Enterprise | Clean + Layered |
