# Code Generation Pipeline — Complete Implementation Guide

## Overview
The 14-phase pipeline transforms requirements into production-ready systems. This document provides implementation details for each phase.

---

## Phase 0: PROJECT INTAKE — IMPLEMENTATION

### Input Analysis Template
```yaml
Project Analysis:
  name: string
  description: string
  requirements:
    functional:
      - feature description
      - acceptance criteria
    non_functional:
      - scalability: number of users
      - performance: response time targets
      - availability: uptime SLA
      - security: compliance requirements
    integrations:
      - third_party_service: API/webhook

  existing_project:
    has_existing_code: boolean
    tech_stack:
      language: string
      framework: string
      database: string
      hosting: string
    current_status: string

  constraints:
    budget: string (time/money)
    timeline: string
    team_size: string
    technical_debt: string

  deliverables:
    - code
    - tests
    - docs
    - deployment_config
    - infrastructure
```

### Complexity Classification Logic
```
IF requirements.features < 3 AND requirements.scope <= 1 week
  COMPLEXITY = FEATURE
ELSE IF requirements.features < 10 AND scope <= 2 weeks
  COMPLEXITY = MODULE
ELSE IF requirements.services == 1 AND requirements.features < 50
  COMPLEXITY = APP
ELSE IF requirements.services >= 2 AND requirements.features < 200
  COMPLEXITY = SYSTEM
ELSE
  COMPLEXITY = PLATFORM
```

### Deliverables Manifest
```json
{
  "deliverables": {
    "code": {
      "frontend": true,
      "backend": true,
      "database": true,
      "files_estimated": 0
    },
    "tests": {
      "unit": true,
      "integration": true,
      "e2e": true,
      "coverage_target": 0.8
    },
    "documentation": {
      "readme": true,
      "api_docs": true,
      "architecture": true,
      "deployment": true
    },
    "infrastructure": {
      "dockerfile": true,
      "docker_compose": true,
      "ci_cd": true,
      "deployment_config": true
    },
    "security": {
      "audit": true,
      "fixes": true
    }
  },
  "estimated_effort": "X hours",
  "estimated_files": "X files",
  "complexity": "APP"
}
```

---

## Phase 1: ARCHITECTURE DESIGN — IMPLEMENTATION

### Architecture Pattern Selection
```
Monolith (MVP/Small Projects)
├── Advantages: Simple, fast to develop, easy to deploy
├── Disadvantages: Harder to scale, one language/framework
└── Use: MVP, startups, < 5 developers

Microservices (Large Systems)
├── Advantages: Independent scaling, polyglot, resilient
├── Disadvantages: Complex, distributed systems issues, ops overhead
└── Use: 50+ engineers, high availability requirements, multiple teams

Serverless (Cost-effective, Auto-scale)
├── Advantages: No infrastructure, auto-scale, pay per use
├── Disadvantages: Cold starts, vendor lock-in, debugging difficulty
└── Use: Event-driven, sporadic traffic, API backends

Modular Monolith (Best of Both)
├── Advantages: Scalability without microservices complexity
├── Disadvantages: Requires discipline, careful architecture
└── Use: Growing products, multiple teams, future-proofing
```

### Tech Stack Template
```json
{
  "architecture": "monolith",
  "frontend": {
    "framework": "Next.js 14",
    "language": "TypeScript",
    "styling": "Tailwind CSS",
    "state_management": "Zustand",
    "forms": "React Hook Form + Zod",
    "data_fetching": "TanStack Query",
    "testing": "Vitest + Playwright"
  },
  "backend": {
    "runtime": "Node.js 20",
    "framework": "Express",
    "language": "TypeScript",
    "orm": "Prisma",
    "validation": "Zod",
    "auth": "NextAuth.js",
    "testing": "Jest"
  },
  "database": {
    "primary": "PostgreSQL 15",
    "cache": "Redis 7",
    "search": "Elasticsearch (optional)"
  },
  "infrastructure": {
    "hosting": "Vercel (frontend) + AWS EC2 (backend)",
    "container": "Docker",
    "orchestration": "Docker Compose (dev), Kubernetes (prod)",
    "ci_cd": "GitHub Actions"
  },
  "monitoring": {
    "logs": "CloudWatch / DataDog",
    "errors": "Sentry",
    "performance": "New Relic",
    "uptime": "Healthchecks.io"
  }
}
```

### Data Model Design (ER Diagram)
```
User
├── id: UUID (PK)
├── email: String (UNIQUE)
├── password_hash: String
├── first_name: String
├── last_name: String
├── created_at: DateTime
├── updated_at: DateTime
└── Relationships:
    ├── 1:N → Post
    ├── 1:N → Comment
    └── M:N → Team (through TeamMember)

Post
├── id: UUID (PK)
├── user_id: UUID (FK)
├── title: String
├── content: Text
├── published: Boolean
├── created_at: DateTime
├── updated_at: DateTime
└── Relationships:
    ├── N:1 → User
    └── 1:N → Comment

Comment
├── id: UUID (PK)
├── post_id: UUID (FK)
├── user_id: UUID (FK)
├── content: Text
├── created_at: DateTime
├── updated_at: DateTime
└── Relationships:
    ├── N:1 → Post
    └── N:1 → User

Team
├── id: UUID (PK)
├── name: String
├── created_at: DateTime
└── Relationships:
    └── M:N → User (through TeamMember)

TeamMember (Join Table)
├── team_id: UUID (PK, FK)
├── user_id: UUID (PK, FK)
├── role: String (admin, member, viewer)
├── joined_at: DateTime
```

### API Surface Design (REST)
```
Authentication
POST   /api/auth/register        Create account
POST   /api/auth/login           Login
POST   /api/auth/logout          Logout
POST   /api/auth/refresh         Refresh JWT
POST   /api/auth/forgot-password Request password reset

Users
GET    /api/users/me             Current user profile
GET    /api/users/:id            User profile
PATCH  /api/users/me             Update profile
DELETE /api/users/me             Delete account

Posts
GET    /api/posts                List posts (paginated)
POST   /api/posts                Create post
GET    /api/posts/:id            Get post details
PATCH  /api/posts/:id            Update post
DELETE /api/posts/:id            Delete post
GET    /api/posts/:id/comments   List comments on post
POST   /api/posts/:id/comments   Add comment

Comments
GET    /api/comments/:id         Get comment
PATCH  /api/comments/:id         Update comment
DELETE /api/comments/:id         Delete comment

Teams
GET    /api/teams                List teams
POST   /api/teams                Create team
GET    /api/teams/:id            Get team
PATCH  /api/teams/:id            Update team
DELETE /api/teams/:id            Delete team
GET    /api/teams/:id/members    List members
POST   /api/teams/:id/members    Add member
DELETE /api/teams/:id/members/:userId  Remove member
```

---

## Phase 2: SPECIFICATION — IMPLEMENTATION

### OpenAPI Specification Template
```yaml
openapi: 3.0.0
info:
  title: Blog API
  version: 1.0.0
  description: |
    Blog platform API for creating, reading, updating posts and comments.

paths:
  /api/posts:
    get:
      summary: List all posts
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Post'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    post:
      summary: Create a new post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreatePostRequest'
      responses:
        201:
          description: Post created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Post'
        400:
          $ref: '#/components/responses/ValidationError'
        401:
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    Post:
      type: object
      required: [id, title, content, userId, createdAt]
      properties:
        id:
          type: string
          format: uuid
        title:
          type: string
          minLength: 1
          maxLength: 255
        content:
          type: string
          minLength: 1
        userId:
          type: string
          format: uuid
        published:
          type: boolean
          default: false
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    CreatePostRequest:
      type: object
      required: [title, content]
      properties:
        title:
          type: string
          minLength: 1
          maxLength: 255
        content:
          type: string
          minLength: 1
        published:
          type: boolean
          default: false

    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object

  responses:
    ValidationError:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

---

## Phase 3: SCAFFOLD — IMPLEMENTATION

### Project Directory Structure
```
my-app/
├── .github/workflows/
│   ├── ci.yml                # Lint, test, build
│   └── deploy.yml            # Deploy to production
├── frontend/
│   ├── src/
│   │   ├── pages/            # Route components
│   │   ├── components/       # Reusable components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── services/         # API clients
│   │   ├── store/            # State management
│   │   ├── styles/           # Global styles
│   │   ├── utils/            # Utilities
│   │   ├── types/            # TypeScript types
│   │   ├── app.tsx           # Root component
│   │   └── main.tsx          # Entry point
│   ├── public/               # Static assets
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── vitest.config.ts
│   ├── .eslintrc.json
│   └── .prettierrc.json
├── backend/
│   ├── src/
│   │   ├── routes/           # Express routes
│   │   ├── controllers/      # Route handlers
│   │   ├── services/         # Business logic
│   │   ├── models/           # Database models (Prisma)
│   │   ├── middleware/       # Express middleware
│   │   ├── utils/            # Utilities
│   │   ├── types/            # TypeScript types
│   │   ├── config/           # Configuration
│   │   ├── db/
│   │   │   ├── migrations/   # Database migrations
│   │   │   └── seeds/        # Seed data
│   │   └── server.ts         # Server entry point
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   ├── .env.example
│   ├── package.json
│   ├── tsconfig.json
│   ├── jest.config.js
│   ├── .eslintrc.json
│   └── .prettierrc.json
├── docker-compose.yml
├── Dockerfile
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── LICENSE
└── CONTRIBUTING.md
```

### Package.json for Node/Express Backend
```json
{
  "name": "blog-api",
  "version": "1.0.0",
  "description": "Blog platform API",
  "type": "module",
  "main": "dist/server.js",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src tests",
    "lint:fix": "eslint src tests --fix",
    "format": "prettier --write src tests",
    "db:migrate": "prisma migrate dev",
    "db:seed": "tsx src/db/seeds/index.ts",
    "db:studio": "prisma studio",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "zod": "^3.22.4",
    "@prisma/client": "^5.4.1",
    "bcrypt": "^5.1.1",
    "jsonwebtoken": "^9.1.2",
    "pino": "^8.16.2",
    "pino-http": "^8.5.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.9.0",
    "@types/bcrypt": "^5.0.2",
    "@types/jsonwebtoken": "^9.0.5",
    "typescript": "^5.2.2",
    "tsx": "^4.1.0",
    "eslint": "^8.52.0",
    "@typescript-eslint/eslint-plugin": "^6.9.1",
    "@typescript-eslint/parser": "^6.9.1",
    "prettier": "^3.0.3",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.8",
    "ts-jest": "^29.1.1",
    "prisma": "^5.4.1"
  },
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=10.0.0"
  }
}
```

### Package.json for React/Next.js Frontend
```json
{
  "name": "blog-web",
  "version": "1.0.0",
  "description": "Blog platform web application",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint src",
    "lint:fix": "eslint src --fix",
    "format": "prettier --write src",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "type-check": "tsc --noEmit",
    "test:e2e": "playwright test"
  },
  "dependencies": {
    "next": "^14.0.3",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "zustand": "^4.4.3",
    "react-hook-form": "^7.48.0",
    "zod": "^3.22.4",
    "@hookform/resolvers": "^3.3.2",
    "@tanstack/react-query": "^5.28.0",
    "axios": "^1.6.1",
    "tailwindcss": "^3.3.5",
    "next-themes": "^0.2.1"
  },
  "devDependencies": {
    "@types/node": "^20.9.0",
    "@types/react": "^18.2.37",
    "typescript": "^5.2.2",
    "eslint": "^8.52.0",
    "eslint-config-next": "^14.0.3",
    "@typescript-eslint/eslint-plugin": "^6.9.1",
    "@typescript-eslint/parser": "^6.9.1",
    "prettier": "^3.0.3",
    "vitest": "^0.34.6",
    "@vitest/ui": "^0.34.6",
    "@testing-library/react": "^14.1.0",
    "@testing-library/jest-dom": "^6.1.5",
    "playwright": "^1.40.1",
    "@playwright/test": "^1.40.1",
    "tailwindcss": "^3.3.5",
    "postcss": "^8.4.31",
    "autoprefixer": "^10.4.16"
  },
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=10.0.0"
  }
}
```

### TypeScript Config (tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@services/*": ["./src/services/*"],
      "@utils/*": ["./src/utils/*"],
      "@types/*": ["./src/types/*"]
    }
  },
  "include": ["src", "tests"],
  "exclude": ["node_modules", "dist"]
}
```

---

## Phase 4-13: Detailed Code Generation

See specific framework reference files:
- `/references/fullstack-frameworks.md` — Implementation for each stack
- `/references/database-patterns.md` — Database layer generation
- `/references/error-handling-patterns.md` — Error handling by framework

---

## Testing Strategy

### Unit Test Template (Jest/Vitest)
```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { createPost } from '@services/postService';

describe('PostService', () => {
  describe('createPost', () => {
    it('should create a post with valid input', async () => {
      const input = {
        title: 'Test Post',
        content: 'Test content',
        userId: 'user-123'
      };

      const result = await createPost(input);

      expect(result).toMatchObject({
        title: input.title,
        content: input.content,
        userId: input.userId,
        id: expect.any(String),
        createdAt: expect.any(Date)
      });
    });

    it('should throw error with empty title', async () => {
      const input = {
        title: '',
        content: 'Test content',
        userId: 'user-123'
      };

      await expect(createPost(input)).rejects.toThrow(
        'Title is required'
      );
    });
  });
});
```

---

## Build Verification Checklist

- [ ] No TypeScript errors (`tsc --noEmit`)
- [ ] No ESLint warnings
- [ ] All tests passing
- [ ] Code coverage >= 80%
- [ ] Build succeeds (`npm run build`)
- [ ] No production console.logs
- [ ] No hardcoded secrets or API keys
- [ ] All environment variables documented
- [ ] Docker image builds successfully
