# Prompt Patterns for Code Generation - Expert Reference

## Table of Contents

1. [Specification-First Pattern](#specification-first-pattern)
2. [Test-Driven Pattern](#test-driven-pattern)
3. [Defensive Coding Pattern](#defensive-coding-pattern)
4. [API Surface Pattern](#api-surface-pattern)
5. [Incremental Complexity Pattern](#incremental-complexity-pattern)
6. [Code Review Self-Prompt](#code-review-self-prompt)
7. [Architecture Decision Pattern](#architecture-decision-pattern)

---

## Specification-First Pattern

### Define Interface → Implement → Test

**CRITICAL**: Start with contracts, not implementations:

```
PATTERN: Define interface first, then implement

Step 1: DEFINE INTERFACE (Specification)
  Prompt to model:
  "Define the TypeScript interface for UserService with:
   - getAllUsers(): Promise<User[]>
   - getUserById(id: number): Promise<User | null>
   - createUser(data: CreateUserInput): Promise<User>
   - updateUser(id: number, data: UpdateUserInput): Promise<User>
   - deleteUser(id: number): Promise<boolean>

   Include JSDoc comments with parameter descriptions, return types,
   and any thrown errors. This is the contract."

  Output: UserService interface

Step 2: IMPLEMENT (Based on contract)
  Prompt to model:
  "Implement the UserService interface from [previous output].
   Use the provided UserRepository interface.
   Include error handling for all edge cases.
   Validate all inputs. Log important operations."

  Output: UserService implementation

Step 3: TEST (Verify contract)
  Prompt to model:
  "Write tests for UserService that verify:
   - Each method returns expected types
   - Errors thrown match interface documentation
   - Invalid inputs rejected
   - Edge cases handled
   Use the interface contract as test specification."

  Output: UserService.test.ts

EXAMPLE PROMPT:

"I need a user authentication service. Define the TypeScript interface:

interface AuthService {
  login(email: string, password: string): Promise<AuthToken>;
  logout(userId: number): Promise<void>;
  refresh(token: string): Promise<AuthToken>;
  validateToken(token: string): Promise<ValidatedToken>;
}

Where:
- AuthToken = { accessToken: string; refreshToken: string }
- ValidatedToken = { userId: number; expiresAt: Date }
- Throw UnauthorizedError if credentials invalid
- Throw TokenExpiredError if token expired

Include comprehensive JSDoc."
```

### Benefits

```
✓ Clear contracts before implementation
✓ Easier to test (test the interface, not internals)
✓ Better code review (interface review first)
✓ Parallel work (implement and test in parallel)
✓ Prevents implementation errors (contract drives implementation)
```

---

## Test-Driven Pattern

### Write Tests First → Implement to Pass

**CRITICAL**: Tests drive implementation:

```
PATTERN: TDD - tests before code

Step 1: WRITE TESTS
  Prompt to model:
  "Write comprehensive Jest tests for a calculateDiscount function that:
   - Takes subtotal (number) and percentage (number)
   - Returns final price (number)
   - Throws ValidationError if subtotal < 0
   - Throws ValidationError if percentage not 0-100
   - Handles edge case: 0% discount = subtotal
   - Handles edge case: 100% discount = 0
   - Handles decimal values correctly
   Use table-driven tests for multiple cases."

  Output: calculateDiscount.test.ts

Step 2: IMPLEMENT TO PASS TESTS
  Prompt to model:
  "Implement calculateDiscount function to pass all these tests: [paste test output]

   Ensure:
   - All tests pass
   - No extra code (only what tests require)
   - Clear implementation"

  Output: calculateDiscount.ts

VALIDATION:
  Run: npm test
  If all tests pass: Done
  If tests fail: Regenerate implementation

EXAMPLE TDD CYCLE:

Test 1: it('applies discount correctly', () => {
  const result = calculateDiscount(100, 10);
  expect(result).toBe(90);  // FAILS - function doesn't exist
});

Implementation 1:
export function calculateDiscount(subtotal, percentage) {
  return subtotal * (1 - percentage / 100);
}
// Test passes now

Test 2: it('throws error for invalid subtotal', () => {
  expect(() => calculateDiscount(-10, 10)).toThrow(ValidationError);
});
// FAILS - no validation

Implementation 2:
export function calculateDiscount(subtotal, percentage) {
  if (subtotal < 0) throw new ValidationError('Subtotal must be >= 0');
  if (percentage < 0 || percentage > 100) throw new ValidationError('Percentage must be 0-100');
  return subtotal * (1 - percentage / 100);
}
// All tests pass
```

### Why TDD Works for AI Code Generation

```
✓ Clear success criteria (tests pass = done)
✓ Prevents over-engineering (only implement what tests need)
✓ Catches bugs early (tests find issues immediately)
✓ Documents behavior (tests are executable spec)
✓ Regression prevention (tests prevent future breaks)
✓ Confidence increase (if tests pass, code works)
```

---

## Defensive Coding Pattern

### Validate → Handle → Recover

**CRITICAL**: Assume inputs are hostile:

```
PATTERN: Defensive coding approach

PROMPT TEMPLATE:
"Generate a function that [description].

Requirements:
1. VALIDATE INPUTS: Check all parameters
   - Correct types
   - Required fields present
   - Values within acceptable ranges
   - Throw ValidationError if invalid

2. HANDLE ERRORS: Manage all failure cases
   - External service failures
   - Database errors
   - Network timeouts
   - Invalid state
   - Throw specific error types

3. RECOVER GRACEFULLY: Provide fallbacks
   - Use cached data if service fails
   - Provide default values where sensible
   - Log all errors with context
   - Return meaningful error messages

4. DOCUMENT: Include JSDoc
   - Parameter validation rules
   - Possible error conditions
   - Example usage

Include tests for:
- Happy path
- Invalid inputs
- Error conditions
- Edge cases"

EXAMPLE: Safe API Call Function

Prompt:
"Create a function that fetches user data from API with defensive coding:
  function getUser(userId: unknown): Promise<User>

Defensive requirements:
1. Validate userId: must be positive integer
2. Handle: Network timeout (max 5s), API error, invalid response
3. Recover: Cache user after success, use cache on error
4. Errors: Custom error types, meaningful messages
5. Tests: All failure modes tested"

Output would include:
- Input validation (userId type check)
- Timeout handling (AbortController)
- Error handling (try/catch for each failure)
- Fallback (cache lookup)
- Specific error types
- Comprehensive tests
```

---

## API Surface Pattern

### Design Consumer Experience → Implement Internals

**CRITICAL**: Think from consumer perspective first:

```
PATTERN: Design API surface before implementation

Step 1: DESIGN API (Consumer perspective)
  Prompt to model:
  "Design a TypeScript API for [feature] from the consumer perspective.
   The consumer will write code like:

   const service = new UserService(repository);
   const users = await service.getActiveUsers();
   const user = await service.getUserById(123);
   if (!user) { /* handle not found */ }
   await service.updateUser(123, { email: 'new@test.com' });

   Define:
   1. Class/module interface
   2. Method signatures
   3. Return types (including null/error cases)
   4. Error conditions and error types
   5. JSDoc with examples

   Make the API intuitive for consumers."

Step 2: IMPLEMENT INTERNALS
  Prompt to model:
  "Implement the API from Step 1 with:
   - Clear parameter validation
   - Comprehensive error handling
   - Efficient database queries
   - Transaction support where appropriate
   - Logging for debugging"

EXAMPLE: Repository API Design

Consumer perspective:
  const users = await repo.findByStatus('active');  // Intuitive

Internal implementation:
  private executeQuery(sql, params) { /* ... */ }
  private validateStatus(status) { /* ... */ }
  private mapRowToUser(row) { /* ... */ }

API Design Prompt:
"Design a ProductRepository API where consumers can:
1. Find products by category: repo.findByCategory('electronics')
2. Search products: repo.search({ query: 'wireless', minPrice: 10 })
3. Paginate results: repo.findAll({ limit: 20, offset: 40 })
4. Get single product: repo.getById(123)
5. Create: repo.create({ name, price })
6. Update: repo.update(123, { price: 29.99 })
7. Delete: repo.delete(123)

Make it clean, intuitive, chainable where possible.
Include error cases (not found, validation, db errors)."
```

---

## Incremental Complexity Pattern

### Start Simple → Add Features → Optimize

**CRITICAL**: Build in layers, not all at once:

```
PATTERN: Incremental development

Step 1: BASIC (Minimal viable implementation)
  Prompt to model:
  "Create a basic [component] that:
   - Does the core functionality only
   - Happy path only (no error handling yet)
   - No optimization
   - No edge cases

   Example: UserService that just gets/creates users"

  Output: Basic UserService

Step 2: COMPLETE (Add features)
  Prompt to model:
  "Enhance the UserService with:
   - Error handling for all operations
   - Input validation
   - Edge case handling (duplicate email, not found, etc.)
   - Proper error types"

  Output: Feature-complete UserService

Step 3: OPTIMIZE (Performance and polish)
  Prompt to model:
  "Optimize UserService:
   - Add caching for frequently accessed users
   - Query optimization (indexes used)
   - Connection pooling
   - Logging and monitoring
   - Performance tests"

  Output: Production-ready UserService

TIMELINE EXAMPLE:

Phase 1 (10 min): Basic
  function createUser(email, name) {
    return db.insert({ email, name });
  }

Phase 2 (10 min): Add error handling
  function createUser(email, name) {
    if (!email) throw new ValidationError('Email required');
    if (!name) throw new ValidationError('Name required');
    try {
      return db.insert({ email, name });
    } catch (err) {
      if (err.code === 'UNIQUE_VIOLATION') {
        throw new DuplicateError('Email already exists');
      }
      throw new DatabaseError('Failed to create user');
    }
  }

Phase 3 (10 min): Optimize
  class UserService {
    constructor(repo, cache) { }

    async createUser(email, name) {
      // Validate
      if (!email) throw new ValidationError('Email required');

      // Check cache first (not found)
      const cached = await cache.get(`user:${email}`);
      if (cached?.exists === false) {
        throw new DuplicateError('Email already exists');
      }

      // Create with transaction
      try {
        const user = await this.repo.create({ email, name });
        await cache.set(`user:${user.id}`, user);
        return user;
      } catch (err) {
        // Error handling
      }
    }
  }
```

---

## Code Review Self-Prompt

### Argue Code Is Wrong → Find and Fix Issues

**CRITICAL**: Self-critical approach prevents bugs:

```
PATTERN: Self-review before submission

Step 1: GENERATE CODE
  Model generates code normally

Step 2: SELF-REVIEW PROMPT
  Prompt model with generated code:
  "Review this code for bugs and improvements:

   [paste generated code]

   Check:
   1. CORRECTNESS: Does it handle all cases?
      - Happy path works?
      - Error cases handled?
      - Edge cases covered?
   2. SECURITY: Any vulnerabilities?
      - Input validation present?
      - No SQL injection?
      - No XSS risks?
   3. ROBUSTNESS: What breaks this?
      - Timeout scenarios
      - Null/undefined values
      - Concurrent access
      - Resource exhaustion
   4. PERFORMANCE: Any inefficiencies?
      - N+1 queries?
      - Unnecessary computations?
      - Memory leaks?
   5. STYLE: Violations?
      - Naming conventions
      - Code duplication
      - Complexity

   List ALL issues found, even minor ones."

Step 3: GENERATE FIXES
  Prompt model:
  "Fix all issues from the review.
   Generate improved code that:
   - Handles all cases
   - Secure
   - Efficient
   - Clean code"

EXAMPLE SELF-REVIEW:

Generated code:
  async function getUser(id) {
    const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);
    return user[0];
  }

Self-review prompt produces:
  Issues found:
  1. SQL injection: String interpolation (CRITICAL)
  2. No validation: id not checked (HIGH)
  3. No error handling: Query failure not handled (HIGH)
  4. No null check: Returns undefined if not found (MEDIUM)

Fixed code:
  async function getUser(id: number): Promise<User | null> {
    if (!Number.isInteger(id) || id <= 0) {
      throw new ValidationError('ID must be positive integer');
    }

    try {
      const result = await db.query(
        'SELECT * FROM users WHERE id = $1',
        [id]  // Parameterized
      );

      return result.rows[0] || null;
    } catch (err) {
      logger.error('Database error', { id, error: err.message });
      throw new DatabaseError('Failed to fetch user');
    }
  }
```

---

## Architecture Decision Pattern

### Options → Trade-offs → ADR

**CRITICAL**: Document architectural choices:

```
PATTERN: Architecture decision record (ADR)

PROMPT TEMPLATE:
"Make an architecture decision for [problem].

Provide:
1. CONTEXT: Why is this decision needed?
   - What problem are we solving?
   - What constraints exist?
   - What are success criteria?

2. OPTIONS: What are the choices?
   Option A: [description]
   - Pros: [list]
   - Cons: [list]
   - Cost estimate: [estimate]
   - Risk level: [low/medium/high]

   Option B: [description]
   - Pros: [list]
   - Cons: [list]
   - Cost estimate: [estimate]
   - Risk level: [low/medium/high]

   Option C: [description]
   - Pros: [list]
   - Cons: [list]
   - Cost estimate: [estimate]
   - Risk level: [low/medium/high]

3. DECISION: Which option and why?
   - Selected: [Option X]
   - Justification: [detailed reasoning]
   - Trade-offs accepted: [what we're giving up]
   - Contingencies: [if it doesn't work, we do X]

4. IMPLEMENTATION: How to execute?
   - Steps to implement
   - Timeline
   - Success metrics"

EXAMPLE: Database Choice

"Decide on primary database for SaaS with 1M users.
Constraints: Real-time, transactional, scalable to 100M users.

Options:
A) PostgreSQL
   - Pros: Powerful, ACID, extensible, open source
   - Cons: Single-machine scaling limit, complex replication
   - Cost: Low
   - Risk: Medium (needs sharding at scale)

B) MongoDB
   - Pros: Horizontal scaling, flexible schema, real-time
   - Cons: Higher memory, no ACID, learning curve
   - Cost: Medium
   - Risk: Medium (eventual consistency)

C) CockroachDB
   - Pros: SQL, distributed, auto-scaling, ACID
   - Cons: Newer, smaller community, vendor lock-in
   - Cost: High
   - Risk: Low

Decision: PostgreSQL initially, migrate to CockroachDB if sharding needed

Rationale:
- Proven, stable, excellent for transactional workloads
- Lower operational complexity at current scale
- Familiar to team
- Contingency: CockroachDB migration path clear

Implementation:
- Week 1: Setup PostgreSQL cluster
- Week 2: Schema design and optimization
- Week 3: Application integration
- Ongoing: Monitor for sharding triggers"
```

---

## Summary: Prompt Pattern Checklist

- [ ] Specification-first used for new modules
- [ ] TDD used for critical functions
- [ ] Defensive coding for all user input
- [ ] API surface designed before implementation
- [ ] Complexity increased incrementally
- [ ] Self-review performed on generated code
- [ ] Architecture decisions documented
- [ ] Prompts include both positive and negative cases
- [ ] Error conditions explicitly specified
- [ ] Edge cases identified and handled
- [ ] Tests written before or alongside code
- [ ] Security assumptions documented
- [ ] Performance requirements explicit
