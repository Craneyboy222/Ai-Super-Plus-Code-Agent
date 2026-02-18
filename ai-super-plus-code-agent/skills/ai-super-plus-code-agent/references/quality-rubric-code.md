# Quality Rubric for Code - Expert Reference

## Quality Dimensions

### 1. Correctness (Does it work?)

**10: Perfect Correctness**
- All edge cases handled
- No known bugs
- Extensive test coverage
- Handles both happy and error paths
- Cross-browser/platform tested

**8: Production-Ready**
- Core functionality correct
- Common edge cases handled
- 80%+ test coverage
- Clear error handling
- No critical bugs

**6: Acceptable**
- Main flow works
- Some edge cases untested
- 60%+ test coverage
- Basic error handling
- Minor bugs acceptable

**4: Problematic**
- Works for basic cases
- Fails on edge cases
- <50% test coverage
- Incomplete error handling
- Known bugs not fixed

**2: Broken**
- Fails on common cases
- No error handling
- Untested
- Multiple critical bugs

**1: Non-Functional**
- Doesn't work at all

---

### 2. Completeness (Is it finished?)

**10: Fully Complete**
- All requirements implemented
- All edge cases handled
- Documentation complete
- Tests pass
- Ready for production

**8: Nearly Complete**
- All major features implemented
- Most edge cases handled
- Documentation substantial
- Tests mostly passing
- Minor gaps only

**6: Substantially Complete**
- Core features implemented
- Some features missing
- Basic documentation
- 70%+ of tests passing
- Clear TODOs documented

**4: Partially Complete**
- Major features missing
- Incomplete implementation
- Minimal documentation
- 50%+ of tests passing
- Many TODOs

**2: Barely Started**
- Few features implemented
- Very incomplete
- No documentation
- Tests failing
- Too many gaps

**1: Empty/Stub**
- No implementation

---

### 3. Robustness (Does it handle failures gracefully?)

**10: Extremely Robust**
- Comprehensive error handling
- Circuit breakers for external calls
- Graceful degradation
- Resource limits enforced
- Recovery mechanisms
- Chaos-tested

**8: Very Robust**
- Good error handling
- Most external calls protected
- Timeouts implemented
- Resource limits in place
- Clear error messages

**6: Acceptably Robust**
- Basic error handling
- Some external protection
- Timeouts on critical paths
- Error messages present
- Basic resource checks

**4: Fragile**
- Minimal error handling
- Unprotected external calls
- No timeouts
- Poor error messages
- Crashes easily

**2: Very Fragile**
- Almost no error handling
- Fails on invalid input
- Crashes on external errors
- No recovery

**1: Fails Immediately**
- Broken on first error

---

### 4. Performance (Is it fast enough?)

**10: Optimally Fast**
- Consistently < target latency (p99)
- Throughput exceeds requirements
- Memory efficient
- No N+1 queries
- Uses appropriate data structures
- Profiled and optimized

**8: Fast**
- Meets performance targets
- 95% < target latency
- Reasonable memory usage
- Some query optimization
- Efficient algorithms

**6: Adequate**
- Within acceptable bounds
- 80% < target latency
- Memory acceptable
- Basic optimization done
- No major inefficiencies

**4: Slow**
- Below target performance
- 70% < target latency
- Memory concerns
- Unoptimized queries
- Inefficient algorithms

**2: Very Slow**
- Significantly underperforms
- Hangs on moderate load
- Memory leaks
- Multiple N+1 issues

**1: Unusable**
- Timeouts constantly

---

### 5. Security (Is it safe?)

**10: Highly Secure**
- No OWASP Top 10 vulnerabilities
- Input validation strict
- SQL injection impossible
- XSS prevented
- CSRF protected
- Secrets managed properly
- Encryption for sensitive data
- Regularly audited

**8: Secure**
- No critical vulnerabilities
- Input validation present
- Parameterized queries
- XSS prevention
- Authentication required
- Secrets in environment
- HTTPS enforced

**6: Acceptably Secure**
- No major vulnerabilities
- Basic input validation
- SQL prepared statements
- Basic XSS prevention
- Authentication basic
- Secrets stored safely

**4: Insecure**
- Vulnerable to common attacks
- Minimal validation
- SQL injection possible
- XSS vulnerabilities
- Weak authentication
- Hardcoded secrets

**2: Very Insecure**
- Multiple OWASP Top 10 issues
- No validation
- Obvious vulnerabilities

**1: Completely Insecure**
- Trivial to exploit

---

### 6. Maintainability (Is it easy to understand and modify?)

**10: Highly Maintainable**
- Clear, self-documenting code
- Consistent naming
- DRY principle followed
- Single responsibility
- Well-abstracted
- Easy to test
- Easy to extend

**8: Maintainable**
- Clear code
- Consistent style
- Limited duplication
- Clear responsibilities
- Testable
- Documented

**6: Adequately Maintainable**
- Understandable with effort
- Some duplication
- Adequate documentation
- Testable with work
- Some coupling

**4: Hard to Maintain**
- Confusing structure
- Significant duplication
- Poor documentation
- Difficult to test
- High coupling
- Tight dependencies

**2: Very Hard to Maintain**
- Spaghetti code
- No documentation
- Untestable
- Magic numbers
- Unclear intent

**1: Unmaintainable**
- Completely opaque

---

### 7. Idiomatic (Does it follow language/framework conventions?)

**10: Perfectly Idiomatic**
- Native patterns throughout
- Framework conventions followed
- No language antipatterns
- Idiomatic error handling
- Proper concurrency patterns
- Library usage correct

**8: Idiomatic**
- Follows language conventions
- Framework patterns used
- Few style violations
- Natural error handling
- Appropriate use of language features

**6: Adequately Idiomatic**
- Mostly follows conventions
- Some non-idiomatic patterns
- Understandable approach
- Works despite style issues

**4: Non-Idiomatic**
- Fights the language/framework
- Many antipatterns
- Wrong abstractions
- Poor convention adherence

**2: Very Non-Idiomatic**
- Looks like another language
- Major antipatterns
- Misunderstood framework

**1: Unrecognizable**
- Wrong paradigm entirely

---

### 8. Documentation (Is it well documented?)

**10: Excellently Documented**
- API documentation complete (JSDoc/docstrings)
- Architecture documented
- Complex algorithms explained
- Examples provided
- Setup instructions clear
- Troubleshooting guide
- Contributing guide

**8: Well Documented**
- API documentation present
- Key concepts explained
- Examples for common use cases
- Setup instructions
- README comprehensive

**6: Adequately Documented**
- API documented
- Basic explanations
- README covers basics
- Some examples

**4: Poorly Documented**
- Minimal documentation
- README vague
- No code comments
- Hard to understand

**2: Barely Documented**
- No meaningful documentation
- Code unclear

**1: Undocumented**
- Zero documentation

---

## File-Level Quality Gate

A file is considered COMPLETE only if:

**CRITICAL (ALL MUST BE TRUE):**
- [ ] Correctness >= 8 (no critical bugs)
- [ ] Completeness >= 8 (all features implemented)
- [ ] All tests passing
- [ ] No lint errors
- [ ] No security vulnerabilities
- [ ] Error handling for all branches

**HIGH (ALL MUST BE TRUE):**
- [ ] Code is idiomatic (>= 7)
- [ ] Maintainability >= 7
- [ ] Documentation present (>= 6)
- [ ] Performance acceptable (>= 6 if performance-critical)
- [ ] Test coverage >= 80%

**MEDIUM:**
- [ ] Self-documenting code
- [ ] No hardcoded values
- [ ] Consistent naming
- [ ] DRY principle (no >2 line duplication)

---

## Function-Level Quality

### Required for Every Function:

**Parameter Validation:**
```javascript
// ✓ CORRECT: Validate inputs
function calculateDiscount(subtotal, percentage) {
  if (typeof subtotal !== 'number' || subtotal < 0) {
    throw new TypeError('subtotal must be non-negative number');
  }
  if (typeof percentage !== 'number' || percentage < 0 || percentage > 100) {
    throw new TypeError('percentage must be 0-100');
  }
  return subtotal * (1 - percentage / 100);
}

// ✗ WRONG: No validation
function calculateDiscount(subtotal, percentage) {
  return subtotal * (1 - percentage / 100);
}
```

**Error Handling:**
```javascript
// ✓ CORRECT: Handle all error cases
async function fetchUser(id) {
  if (!id) {
    throw new ValidationError('User ID required');
  }

  try {
    const response = await fetch(`/api/users/${id}`);

    if (!response.ok) {
      throw new ApiError(`API returned ${response.status}`);
    }

    return response.json();
  } catch (err) {
    if (err instanceof TypeError) {
      throw new NetworkError('Network request failed', { cause: err });
    }
    throw err;  // Re-throw known errors
  }
}

// ✗ WRONG: Silent failures
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();  // No error handling
}
```

**Return Types:**
```typescript
// ✓ CORRECT: Clear return type
function getUserName(user: User): string {
  return user.firstName + ' ' + user.lastName;
}

// Return type includes null when possible
function findUserById(id: number): User | null {
  return database.users.find(u => u.id === id) || null;
}

// ✗ WRONG: Unclear return
function getUserName(user) {
  return user.firstName + ' ' + user.lastName;  // Could be undefined
}
```

**Documentation:**
```javascript
/**
 * Calculate discount on order subtotal
 *
 * @param {number} subtotal - Order subtotal in dollars (>= 0)
 * @param {number} percentage - Discount percentage (0-100)
 * @returns {number} Final price after discount
 * @throws {TypeError} If parameters invalid
 *
 * @example
 * const price = calculateDiscount(100, 10);  // 90
 */
function calculateDiscount(subtotal, percentage) {
  // Implementation
}
```

---

## Architecture-Level Quality

### Separation of Concerns

**CRITICAL: Each module has single responsibility**

```
✓ CORRECT:
- UserRepository: Database access only
- UserService: Business logic only
- UserController: Request/response handling only
- UserValidator: Input validation only

✗ WRONG:
- UserController: Does everything
  - HTTP handling
  - Business logic
  - Database access
  - Validation
```

### Dependency Direction

**CRITICAL: Flow is unidirectional (acyclic)**

```
✓ CORRECT: Dependency graph is acyclic
Controller → Service → Repository → Database
   ↓          ↓        ↓
 HTTP       Business  Data
 Logic      Logic     Access

✗ WRONG: Circular dependencies
UserService → OrderService
OrderService → UserService  (circular!)
```

### Interface Design

**HIGH: Clear contracts at boundaries**

```python
# ✓ CORRECT: Clear interface
class UserRepository:
    def find_by_id(self, id: int) -> Optional[User]:
        """Find user by ID, return None if not found"""

    def create(self, data: CreateUserInput) -> User:
        """Create user, raise DuplicateError if email exists"""

# ✗ WRONG: Unclear interface
class UserRepository:
    def get_user(self, params):  # What params?
        """Get user"""  # What does this do?
```

---

## Scoring Examples

### Example 1: Login Function

```javascript
// SCORE BREAKDOWN:
// Correctness: 7/10
//   - Happy path works
//   - Missing: timeout handling, too many attempts
//
// Completeness: 6/10
//   - Basic functionality done
//   - Missing: 2FA, account lockout, audit logging
//
// Robustness: 4/10
//   - No timeout
//   - Unprotected against brute force
//   - No circuit breaker for DB
//
// Performance: 8/10
//   - Query optimized with index
//   - Direct DB call (good for auth)
//
// Security: 6/10
//   - Uses bcrypt (good)
//   - Missing: rate limiting, audit logs
//   - SQL injection protected
//
// Maintainability: 7/10
//   - Clear logic
//   - Decent variable names
//   - Could extract password validation
//
// Idiomatic: 8/10
//   - Follows Express patterns
//   - Async/await correct
//
// Documentation: 5/10
//   - Function documented
//   - Missing: error cases documented

async function login(req, res) {
  try {
    const { email, password } = req.body;

    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const match = await bcrypt.compare(password, user.passwordHash);
    if (!match) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);
    res.json({ token });
  } catch (err) {
    res.status(500).json({ error: 'Login failed' });
  }
}

// OVERALL: 6.5/10 - Functional but needs hardening
```

### Example 2: Well-Designed Module

```python
# SCORE BREAKDOWN:
# Correctness: 9/10
#   - All cases handled
#   - Comprehensive tests
#
# Completeness: 9/10
#   - All features implemented
#   - Edge cases covered
#
# Robustness: 9/10
#   - Timeout handling
#   - Graceful degradation
#   - Error recovery
#
# Performance: 9/10
#   - Query optimized
#   - Caching used
#   - No N+1
#
# Security: 10/10
#   - Input validation
#   - No vulnerabilities
#   - Audit logging
#
# Maintainability: 9/10
#   - Clear structure
#   - Well-named
#   - Easy to extend
#
# Idiomatic: 9/10
#   - Pythonic
#   - Standard patterns
#
# Documentation: 9/10
#   - Comprehensive docs
#   - Examples provided

class UserService:
    """User business logic with comprehensive error handling"""

    def __init__(self, repository: UserRepository, cache: Cache):
        self.repository = repository
        self.cache = cache

    async def get_user(self, user_id: int) -> Optional[User]:
        """
        Fetch user by ID with caching

        Args:
            user_id: User identifier (positive integer)

        Returns:
            User object if found, None otherwise

        Raises:
            ValidationError: If user_id is invalid
            RepositoryError: If database error occurs
        """
        if not isinstance(user_id, int) or user_id <= 0:
            raise ValidationError('user_id must be positive integer')

        # Try cache first
        cached = await self.cache.get(f'user:{user_id}')
        if cached:
            return User.from_dict(cached)

        try:
            user = await self.repository.find_by_id(user_id)
            if user:
                await self.cache.set(f'user:{user_id}', user.to_dict(), ttl=3600)
            return user
        except DatabaseError as err:
            logger.error('Database error fetching user', extra={'user_id': user_id, 'error': str(err)})
            # Try cache as fallback
            cached_fallback = await self.cache.get(f'user:{user_id}:fallback')
            if cached_fallback:
                return User.from_dict(cached_fallback)
            raise RepositoryError('Failed to fetch user') from err

# OVERALL: 9.2/10 - Production-ready code
```

---

## Summary: Quality Checklist

### Before Marking File Complete:

- [ ] Correctness >= 8: No critical bugs, handles edge cases
- [ ] Completeness >= 8: All features implemented
- [ ] Robustness >= 7: Error handling for all branches
- [ ] Performance >= 6: Acceptable for use case
- [ ] Security >= 8: No vulnerabilities
- [ ] Maintainability >= 7: Clear and understandable
- [ ] Idiomatic >= 7: Follows conventions
- [ ] Documentation >= 6: API documented
- [ ] Tests: 80%+ coverage, all passing
- [ ] No lint errors
- [ ] All parameters validated
- [ ] Error handling comprehensive
- [ ] Return types clear
- [ ] No hardcoded values
- [ ] No code duplication (>2 lines)
