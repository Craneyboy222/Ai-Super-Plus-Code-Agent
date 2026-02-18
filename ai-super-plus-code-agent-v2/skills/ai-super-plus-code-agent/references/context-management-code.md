# Context Management for Code Generation - Expert Reference

## Table of Contents

1. [What to Include](#what-to-include)
2. [What to Exclude](#what-to-exclude)
3. [Progressive Loading](#progressive-loading)
4. [File Dependency Graph Traversal](#file-dependency-graph-traversal)
5. [Token Budget Allocation](#token-budget-allocation)

---

## What to Include in Context

### Interfaces and Type Definitions

**CRITICAL**: Include all types the agent needs to understand:

```typescript
// INCLUDE IN CONTEXT:

// 1. Domain models and their relationships
interface User {
  id: number;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}

// 2. Request/Response types
interface CreateUserInput {
  email: string;
  name: string;
  password: string;
}

interface CreateUserResponse {
  success: boolean;
  user?: User;
  error?: string;
}

// 3. Error types
class ValidationError extends Error {
  code = 'VALIDATION_ERROR';
}

class DuplicateError extends Error {
  code = 'DUPLICATE_ERROR';
}

// 4. Configuration interfaces
interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
  maxConnections: number;
}

// 5. Service/Repository interfaces
interface UserRepository {
  findById(id: number): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(data: CreateUserInput): Promise<User>;
  update(id: number, data: Partial<User>): Promise<User>;
}

// WHY INCLUDE:
// - Agent needs to know what types they're working with
// - Type information prevents bugs
// - Dependencies between types clear
// - Contract is explicit
```

### Related Completed Files

**HIGH**: Include files the new code depends on:

```
SCENARIO: Generating UserService

Include in context:
  ✓ User.ts (the model)
  ✓ UserRepository.ts (dependency)
  ✓ EmailService.ts (if used)
  ✓ ValidationUtils.ts (if used)
  ✓ Error definitions (AppError.ts)

DON'T include:
  ✗ ProductService.ts (different domain)
  ✗ PaymentService.ts (different domain)
  ✗ ConfigService.ts (not used)
  ✗ OrderRepository.ts (not used)

FILE RELEVANCE CHECK:
  Ask: "Does UserService import/use this file?"
  If YES → Include
  If NO → Exclude
```

### Architecture and Design Documents

**HIGH**: High-level context shapes decisions:

```
INCLUDE:
  ✓ architecture.md
    - System components
    - Module responsibilities
    - Data flow
    - Integration points

  ✓ design-decisions.md
    - Why certain patterns chosen
    - Technology selections
    - Constraints and assumptions

  ✓ api-contracts.md
    - Request/response formats
    - Error codes
    - Status codes

Example architecture snippet:
```
  System has three layers:
    1. Controllers (HTTP interface)
    2. Services (Business logic)
    3. Repositories (Data access)

  UserService is in Services layer.
  It uses:
    - UserRepository for data access
    - EmailService for notifications
    - ValidationUtils for input checking

  It should NOT use:
    - HTTP frameworks (that's Controllers)
    - Payment processing (that's OrderService)
```

This context prevents agent from:
  - Putting HTTP handling in service (architectural violation)
  - Adding payment logic (wrong responsibility)
  - Over-coupling to database (layering violation)
```

### Test Examples and Patterns

**MEDIUM**: Show expected test structure:

```javascript
// INCLUDE: Example test structure

// Example unit test
describe('UserService', () => {
  let service;
  let mockRepo;

  beforeEach(() => {
    mockRepo = {
      findById: jest.fn(),
      create: jest.fn()
    };
    service = new UserService(mockRepo);
  });

  it('returns user if found', async () => {
    mockRepo.findById.mockResolvedValue({ id: 1, name: 'John' });
    const user = await service.getUser(1);
    expect(user.name).toBe('John');
  });

  it('throws if user not found', async () => {
    mockRepo.findById.mockResolvedValue(null);
    await expect(service.getUser(1)).rejects.toThrow('User not found');
  });
});

// WHY INCLUDE:
// - Agent sees expected test structure
// - Tests use mocking (not real DB)
// - Error cases tested
// - Agent can follow same pattern
```

### Performance Requirements

**MEDIUM**: Context shapes optimization decisions:

```
INCLUDE in context:

System Performance Requirements:
  - API p99 latency: < 500ms
  - Database query max time: 50ms
  - Cache TTL: 3600 seconds
  - Max concurrent connections: 20

User Service SLA:
  - getUser: < 100ms (frequent operation)
  - createUser: < 500ms (validation heavy)
  - bulkFetch: < 1s for 1000 users

WHY INCLUDE:
  - Agent knows to optimize getUser (frequent, tight SLA)
  - Agent knows createUser can be slower (higher tolerance)
  - Agent knows to use caching/batching
  - Agent knows connection pool size
```

---

## What to Exclude from Context

### Test Files (When Generating Implementation)

**CRITICAL**: Don't include tests when generating implementation:

```
SCENARIO: Generating UserRepository implementation

EXCLUDE:
  ✗ UserRepository.test.ts
  ✗ UserRepository.integration.test.ts

WHY EXCLUDE:
  - Test reveals implementation details
  - Agent might just implement to pass tests (test-driven is OK)
  - But if tests included upfront, agent focuses on tests not design
  - Separate steps: Design → Implement → Test

CORRECT FLOW:
  1. Design UserRepository interface (include)
  2. Generate implementation (no tests)
  3. Run against tests separately
```

### Unrelated Modules

**HIGH**: Don't include files in different domains:

```
SCENARIO: Generating UserService

EXCLUDE:
  ✗ OrderService.ts
  ✗ PaymentProcessor.ts
  ✗ InventoryService.ts

WHY EXCLUDE:
  - Context bloat (irrelevant information)
  - Agent might create unwanted dependencies
  - Different responsibilities
  - Increases token usage unnecessarily

TOKEN SAVINGS:
  Include only UserService domain: ~30KB
  Include everything: ~200KB
  Savings: 85% context reduction, faster response
```

### Internal Implementation Details

**MEDIUM**: Exclude if not needed:

```
EXCLUDE from context when generating code:
  ✗ Implementation of private methods
  ✗ Internal helper functions (if not used by caller)
  ✗ Database connection setup (if generic)
  ✗ Logging implementation details

INCLUDE public contracts:
  ✓ Public method signatures
  ✓ Error types thrown
  ✓ Return types
  ✓ Parameter types

EXAMPLE:

EXCLUDE (implementation detail):
  private validateEmail(email: string): boolean {
    // 50 lines of regex and checks
  }

INCLUDE (public contract):
  public validateEmail(email: string): void {
    // Throws ValidationError if invalid
  }
```

### Old/Deprecated Code

**HIGH**: Never include deprecated code:

```
SCENARIO: Agent sees old code in context

BAD CONTEXT:
  // DEPRECATED: Use UserServiceV2 instead
  export class UserService {
    // Old implementation
  }

  export class UserServiceV2 {
    // New implementation
  }

Agent might:
  - Use deprecated version
  - Copy deprecated patterns
  - Get confused by options

CORRECT APPROACH:
  - Remove deprecated code before sharing context
  - Or clearly mark deprecated with big warnings
  - Or only include non-deprecated code in context
```

---

## Progressive Loading

### Phase 1: Specs and Interfaces

**CRITICAL**: Load minimal context first:

```
CONTEXT LOAD SEQUENCE FOR NEW PROJECT:

PHASE 1: Specifications (5KB)
  Load:
    ✓ requirements.md
    ✓ Type definitions (interfaces only)
    ✓ Error types
  Agent: Opus (Architecture design)
  Output: architecture.md, interface definitions

PHASE 2: Schema and Models (50KB)
  Load:
    ✓ architecture.md (from Phase 1)
    ✓ schema.sql
    ✓ Model interfaces (User, Product, etc.)
  Agent: Opus/Sonnet
  Output: ORM models, type definitions

PHASE 3: Data Access Layer (100KB)
  Load:
    ✓ Models (from Phase 2)
    ✓ Repository interfaces
    ✓ Query patterns
    ✓ Test examples
  Agent: Sonnet
  Output: Repository implementations

PHASE 4: Business Logic (100KB)
  Load:
    ✓ Models (Phase 2)
    ✓ Repositories (Phase 3)
    ✓ Service interfaces
    ✓ Business rules
    ✓ Error handling patterns
  Agent: Opus
  Output: Service implementations

PHASE 5: API Layer (80KB)
  Load:
    ✓ Services (Phase 4)
    ✓ Route interface
    ✓ Request/response schemas
    ✓ Error mapping
    ✓ Middleware patterns
  Agent: Sonnet
  Output: Route handlers, controllers

KEY PRINCIPLE:
  Load context incrementally, not all at once
  Each phase only loads what it needs
```

### Dependency-Driven Loading

**HIGH**: Load based on file dependencies:

```
ALGORITHM: Load minimum context for a file

Input: Target file to generate (e.g., UserService.ts)

1. IDENTIFY DEPENDENCIES
   - Find all imports in target file spec
   - Extract dependency list

   Target: UserService
   Dependencies:
     - User model
     - UserRepository
     - EmailService
     - ValidationUtils
     - AppError

2. VERIFY DEPENDENCIES EXIST
   - Check if all dependencies are completed
   - If not, queue dependency generation first

   Status:
     ✓ User model (completed)
     ✓ UserRepository (completed)
     ✗ EmailService (not yet)
     ✗ ValidationUtils (not yet)

   Action: Generate EmailService and ValidationUtils first

3. LOAD DEPENDENCIES INTO CONTEXT
   - Include all dependency files
   - Include their interfaces
   - DO NOT include unrelated files

   Load for UserService:
     - User.ts
     - UserRepository.ts
     - EmailService.ts
     - ValidationUtils.ts
     - AppError.ts
     - (do NOT load OrderService, ProductService, etc.)

4. ADD SPECIFICATIONS
   - Include UserService interface/specification
   - Include performance requirements
   - Include error handling requirements

5. GENERATE CODE
   - Agent has exactly what it needs
   - Context is minimal and focused
   - Token usage optimized
```

---

## File Dependency Graph Traversal

### Build Dependency Map

**CRITICAL**: Understand file relationships:

```
ALGORITHM: Build dependency graph

Step 1: Scan all imports

models/User.ts:
  imports: (none)

repositories/UserRepository.ts:
  imports: User from models/

services/UserService.ts:
  imports: User, UserRepository, EmailService

services/EmailService.ts:
  imports: (none)

routes/users.ts:
  imports: UserService

Step 2: Create dependency graph

  User ← (no dependencies)
    ↑
    ├── UserRepository
    │      ↑
    │      └── UserService
    │            ↑
    │            ├── routes/users
    │            └── services (depends on this)
    │
    └── EmailService (no dependencies)
           ↑
           └── UserService

Step 3: Identify generation order

Level 1 (no dependencies): User, EmailService
Level 2 (depends on Level 1): UserRepository
Level 3 (depends on Level 2): UserService
Level 4 (depends on Level 3): routes/users

Generate in order: User → UserRepository → UserService → routes

Step 4: Load context progressively

When generating UserService:
  Context load:
    ✓ User (from Level 1)
    ✓ UserRepository (from Level 2)
    ✓ EmailService (from Level 1)
    ✗ routes (depends on this, don't include)
    ✗ ProductService (unrelated)
```

### Circular Dependency Detection

**HIGH**: Detect and break cycles:

```
DETECTION: Find circular dependencies

Example cycle:
  ServiceA imports ServiceB
  ServiceB imports ServiceA

Algorithm:
  1. Build dependency graph
  2. Use depth-first search (DFS)
  3. Mark nodes as visiting/visited
  4. If we revisit a visiting node: cycle detected

Example:

AuthService imports:
  - UserRepository ✓
  - PermissionService

PermissionService imports:
  - AuthService ← CYCLE DETECTED!

RESOLUTION:

Option 1: Extract common interface
  AuthorizationChecker interface (minimal)
    - CheckPermission(user, resource)
  AuthService implements it
  PermissionService depends on interface (not class)

Option 2: Move dependency to different layer
  AuthService and PermissionService both depend on:
    - PermissionValidator (new)

Option 3: Merge modules
  If tightly coupled, merge into single AuthorizationService
```

---

## Token Budget Allocation

### Calculate Context Budget

**CRITICAL**: Know your token limits:

```
TOKEN BUDGET MATH:

Model: Claude 3.5 Sonnet
  Input window: 200,000 tokens
  Reserved for output: 4,000 tokens
  Available for context + prompt: 196,000 tokens

Typical token counts:
  - 1 page of text: ~200-300 tokens
  - 1 function (10 lines): ~30-50 tokens
  - 1 TypeScript interface: ~20-40 tokens
  - Full file (100 lines): ~200-300 tokens

BUDGET ALLOCATION:

Total: 196,000 tokens

1. System prompt (rules, patterns): 5,000 tokens (2.5%)
2. Architecture context: 15,000 tokens (7.5%)
3. Type definitions: 10,000 tokens (5%)
4. Dependency files: 50,000 tokens (25%)
5. Examples and patterns: 10,000 tokens (5%)
6. Performance requirements: 5,000 tokens (2.5%)
7. User prompt (task): 20,000 tokens (10%)
8. Reserved for output: 81,000 tokens (41%)

REMAINING: ~81,000 tokens for generated code output
```

### Budget Limits by Phase

**HIGH**: Different phases need different budgets:

```
PHASE 1: Architecture Design
  Available: 150,000 tokens
  Context: 10,000 tokens (6.5%)
  Output: 140,000 tokens (93.5%)

  Use: Opus (long reasoning)

PHASE 2-4: Code Generation
  Available: 100,000 tokens
  Context: 30,000 tokens (30%)
  Output: 70,000 tokens (70%)

  Use: Opus (complex) or Sonnet (standard)

PHASE 5-6: Standard Implementation
  Available: 60,000 tokens
  Context: 20,000 tokens (33%)
  Output: 40,000 tokens (67%)

  Use: Sonnet

PHASE 7-10: Tests and Docs
  Available: 40,000 tokens
  Context: 15,000 tokens (37%)
  Output: 25,000 tokens (63%)

  Use: Sonnet or Haiku
```

### Optimize Context to Fit Budget

**MEDIUM**: Reduce context when hitting limits:

```
CONTEXT REDUCTION STRATEGY:

If context exceeds budget:

1. REMOVE UNRELATED FILES
   Current: 45KB (includes Order, Payment services)
   Target: 35KB
   Action: Remove Order, Payment (unrelated to User service)
   Result: 20KB savings

2. SUMMARIZE INSTEAD OF FULL CODE
   Replace 20KB of UserRepository implementation
   With: "UserRepository has methods: findById, create, update, delete"
   Result: 10KB savings

3. REFERENCE INSTEAD OF INCLUDE
   Instead of including 5KB of error definitions
   Say: "See error-types.ts for error definitions"
   Result: 5KB savings

4. REMOVE TESTS
   Exclude test files (can be run separately)
   Result: 15KB savings

5. LIMIT EXAMPLES
   Include 1-2 examples instead of 5-6
   Result: 5KB savings

TOTAL SAVINGS: 55KB → Well under budget

DECISION TREE:
  Context too large?
    ↓
  Is it a test file?
    YES → Remove
  Is it unrelated to task?
    YES → Remove
  Is it full implementation?
    YES → Replace with interface/summary
  Is it examples (5+)?
    YES → Reduce to 1-2
```

---

## Summary: Context Management Checklist

- [ ] Include: Interfaces, type definitions, completed dependencies
- [ ] Include: Architecture and design documents
- [ ] Include: Test patterns and examples
- [ ] Include: Performance requirements
- [ ] Exclude: Test implementations (when generating code)
- [ ] Exclude: Unrelated modules
- [ ] Exclude: Internal implementation details
- [ ] Exclude: Deprecated code
- [ ] Load context progressively by phase
- [ ] Load dependencies based on import graph
- [ ] Detect and break circular dependencies
- [ ] Calculate token budget before context loading
- [ ] Allocate tokens by phase criticality
- [ ] Optimize context to fit token limits
- [ ] Document context strategy in project

---

## Context Template

Use this template when preparing context for agent:

```
# CONTEXT FOR [Agent Name]: [Task Description]

## 1. ARCHITECTURE (Include if relevant)
[High-level system design, component relationships]

## 2. INTERFACES & TYPES (Always include)
[Type definitions for code being generated]

## 3. DEPENDENCIES (Load based on imports)
[Files this code depends on]

## 4. PATTERNS & EXAMPLES (Include 1-2 good examples)
[How similar code is implemented]

## 5. REQUIREMENTS (Clear and specific)
[Functional, performance, security requirements]

## 6. CONSTRAINTS
[What this code CANNOT do]

## 7. ERROR HANDLING
[Expected error conditions and types]

## 8. PERFORMANCE TARGETS
[SLA for this code]

## 9. TESTING APPROACH
[How this code will be tested]

## TOKEN BUDGET
- Context size: X KB
- Model capacity: Y tokens
- Reserved for output: Z tokens
- Remaining: (Y-Z) tokens available
```
