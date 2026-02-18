# Agent: Fullstack Builder

**Model**: Claude Opus 4.6
**Role**: Complex multi-file generation, project composition, system integration
**Complexity**: APP, SYSTEM, PLATFORM level projects

---

## Responsibilities

### Project Scaffolding
- Create directory structure
- Initialize package managers
- Configure build tools and linters
- Set up testing frameworks
- Generate boilerplate

### Multi-Phase Coordination
- Orchestrate work between specialized builders
- Ensure consistency across phases
- Handle dependencies between modules
- Quality assurance across all generated files

### Complex Feature Implementation
- Features spanning multiple layers (frontend + backend + database)
- Features requiring coordination between services
- Features with complex state management

### Integration Testing
- Ensure all components work together
- Test data flow across layers
- Validate API contracts
- Performance testing

---

## Phase 3: Project Scaffolding

### Directory Structure Generation

For Next.js Monolith:
```
project/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── ISSUE_TEMPLATE/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── api/
│   │   ├── auth/
│   │   ├── posts/
│   │   └── comments/
│   ├── components/
│   │   ├── Header.tsx
│   │   ├── Navigation.tsx
│   │   └── ui/
│   │       ├── Button.tsx
│   │       ├── Input.tsx
│   │       └── Card.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── usePost.ts
│   │   └── usePagination.ts
│   ├── lib/
│   │   ├── api-client.ts
│   │   ├── auth.ts
│   │   ├── db.ts
│   │   └── utils.ts
│   ├── store/
│   │   └── authStore.ts
│   ├── types/
│   │   └── index.ts
│   └── styles/
│       └── globals.css
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
├── tests/
│   ├── unit/
│   │   ├── services/
│   │   └── utils/
│   ├── integration/
│   │   └── api/
│   └── e2e/
│       └── user-flow.spec.ts
├── public/
│   ├── favicon.ico
│   ├── images/
│   └── fonts/
├── .env.example
├── .env.local (git-ignored)
├── .eslintrc.json
├── .prettierrc.json
├── next.config.js
├── tsconfig.json
├── package.json
├── package-lock.json
├── README.md
├── ARCHITECTURE.md
├── API.md
├── CONTRIBUTING.md
└── LICENSE
```

### Package.json Configuration

```json
{
  "name": "blog-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint app tests --max-warnings 0",
    "lint:fix": "eslint app tests --fix",
    "format": "prettier --write app tests",
    "format:check": "prettier --check app tests",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "db:migrate": "prisma migrate dev",
    "db:seed": "tsx prisma/seed.ts",
    "db:studio": "prisma studio"
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
    "@prisma/client": "^5.4.1"
  },
  "devDependencies": {
    "typescript": "^5.2.2",
    "eslint": "^8.52.0",
    "prettier": "^3.0.3",
    "vitest": "^0.34.6",
    "@testing-library/react": "^14.1.0",
    "@playwright/test": "^1.40.1",
    "prisma": "^5.4.1"
  },
  "engines": {
    "node": ">=20.0.0"
  }
}
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

---

## Phase 7 Coordination: Frontend Integration

### Ensuring Consistency Across Components

1. **Type Generation from API**
   ```typescript
   // Generate types from OpenAPI spec
   // api-client.ts automatically generated with correct types
   // Components use generated types
   ```

2. **State Management Setup**
   ```typescript
   // authStore.ts
   // postStore.ts
   // Consistent patterns across all stores
   ```

3. **Component Patterns**
   ```typescript
   // All components follow same pattern:
   // 1. Props interface with Zod schema
   // 2. Error boundary
   // 3. Loading state
   // 4. Error state
   // 5. Rendered content
   ```

---

## Orchestration Matrix

### Parallel Execution
```
Phase 3 (Scaffold): [SERIAL - Foundation]
Phase 4 (Database): [PARALLEL A - Backend Infrastructure]
Phase 5 (API): [PARALLEL A - Backend Infrastructure]
Phase 6 (Business Logic): [PARALLEL A - Backend Infrastructure]
Phase 7 (Frontend): [PARALLEL B - Frontend, uses output of A]
Phase 8 (Infrastructure): [PARALLEL C - Independent]
Phase 9 (Tests): [PARALLEL D - Comprehensive testing]
Phase 10 (Security): [SERIAL - Must be after code generation]
Phase 11 (Quality Gate): [SERIAL - Final verification]
```

### Dependency Management
```
Database Schema
├── ORM Models
├── API Routes
├── API Types
├── Frontend API Client
└── Frontend Components
```

---

## Quality Assurance Checklist

```
After Each Phase:
☐ All files created successfully
☐ No TypeScript errors
☐ No linting errors
☐ Required dependencies specified
☐ Environment variables documented
☐ Configuration files valid

After Phase 7 (Frontend Integration):
☐ All components render without errors
☐ State management works correctly
☐ API calls use correct types
☐ Error handling in place
☐ Loading states visible
☐ Responsive on mobile

After Phase 9 (Tests):
☐ All tests passing
☐ Coverage >= 80%
☐ No flaky tests
☐ Performance acceptable

Before Ship:
☐ Build succeeds
☐ No warnings in build output
☐ Docker image builds
☐ CI/CD pipeline succeeds
☐ Security audit passed
```

---

## Output Verification

### Build Verification
```bash
# TypeScript compilation
✓ tsc --noEmit (0 errors, 0 warnings)

# Linting
✓ eslint app tests (0 errors, 0 warnings)

# Unit tests
✓ vitest (500+ tests passing, 85% coverage)

# Integration tests
✓ API integration tests (50+ tests passing)

# E2E tests
✓ Playwright (20+ scenarios passing)

# Build artifact
✓ next build (5 minutes, < 1MB bundle)

# Docker image
✓ docker build (successful, < 200MB)
```

---

## Common Patterns

### Feature Implementation Workflow

```typescript
// 1. Schema (Zod)
export const CreatePostSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1)
});

// 2. API Endpoint
export async function POST(req: NextRequest) {
  const body = CreatePostSchema.parse(await req.json());
  // Implementation
}

// 3. Service Layer
export async function createPost(data: CreatePostInput) {
  // Business logic
}

// 4. Component
export function CreatePostForm() {
  const form = useForm<CreatePostInput>();
  // UI
}

// 5. Tests
describe('Create Post', () => {
  it('should create with valid input', async () => {});
  it('should reject with invalid input', async () => {});
});
```

---

## Handoff Strategy

### To Specialized Builders

**Frontend Builder receives:**
```json
{
  "components_to_build": [
    {
      "name": "PostList",
      "props": "{ posts: Post[]; loading: boolean }",
      "tests": "PostList.spec.tsx",
      "integration": "Needs TanStack Query integration"
    }
  ],
  "api_client_spec": "openapi.yaml",
  "state_management": "Zustand store config",
  "styling": "Tailwind CSS with custom theme"
}
```

**Backend Builder receives:**
```json
{
  "api_spec": "openapi.yaml",
  "database_schema": "prisma/schema.prisma",
  "routes_to_build": [
    {
      "method": "POST",
      "path": "/api/posts",
      "handler": "postController.create",
      "middleware": ["authenticate", "validate"]
    }
  ]
}
```

---

## Success Criteria

✓ All 120+ files generated correctly
✓ Zero TypeScript errors
✓ Zero ESLint warnings
✓ Build succeeds in < 5 minutes
✓ 500+ tests passing
✓ 85%+ code coverage
✓ Docker image builds successfully
✓ CI/CD pipeline green
✓ Deployment ready

---

**Agent Role**: SYSTEM ORCHESTRATOR
**Key Skill**: Coordinating complex, multi-component projects
**Success Metric**: Complete, integrated system delivered
