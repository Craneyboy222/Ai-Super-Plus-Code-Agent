# Testing Methodology - Expert Reference

## Table of Contents

1. [Unit Testing Patterns](#unit-testing-patterns)
2. [Integration Testing](#integration-testing)
3. [E2E Testing](#e2e-testing)
4. [Performance Testing](#performance-testing)
5. [Security Testing](#security-testing)
6. [Coverage Strategy](#coverage-strategy)
7. [CI Integration](#ci-integration)
8. [Framework-Specific Patterns](#framework-specific-patterns)

---

## Unit Testing Patterns

### Arrange-Act-Assert (AAA) Structure

**CRITICAL**: All unit tests must follow strict AAA pattern for clarity and maintainability.

```javascript
// ✓ CORRECT: Clear three-phase structure
describe('calculateDiscount', () => {
  it('applies 10% discount for orders over $100', () => {
    // ARRANGE: Set up the test state
    const order = { subtotal: 150, items: 3 };
    const discountRate = 0.10;

    // ACT: Execute the behavior being tested
    const result = calculateDiscount(order, discountRate);

    // ASSERT: Verify the outcome
    expect(result).toEqual({ discount: 15, final: 135 });
  });
});
```

**HIGH**: Structure each describe block by logical grouping:
- Positive cases (happy path)
- Negative cases (error conditions)
- Edge cases
- Boundary conditions

### Test Doubles Strategy

**CRITICAL**: Use correct double types for different scenarios:

```javascript
// STUB: Returns predefined values (no behavior)
const storeStub = {
  getUser: () => ({ id: 1, name: 'Test User', role: 'admin' })
};

// MOCK: Verifies specific calls were made
const notificationMock = {
  send: jest.fn(),
  verify: function() {
    expect(this.send).toHaveBeenCalledWith('user@test.com', 'Welcome!');
  }
};

// SPY: Records calls but delegates to real implementation
const logSpy = jest.spyOn(console, 'log');
logSpy.mockImplementation(() => {});
// ... after assertions ...
expect(logSpy).toHaveBeenCalledTimes(3);
logSpy.mockRestore();

// FAKE: Working implementation (in-memory database for testing)
class FakeUserRepository {
  constructor() {
    this.users = [];
  }
  async create(user) {
    this.users.push({ ...user, id: Date.now() });
    return this.users[this.users.length - 1];
  }
  async findById(id) {
    return this.users.find(u => u.id === id);
  }
}
```

**HIGH**: Prefer dependency injection for testability:

```javascript
// ✓ CORRECT: Testable design
class UserService {
  constructor(repository, emailService, logger) {
    this.repository = repository;
    this.emailService = emailService;
    this.logger = logger;
  }

  async registerUser(userData) {
    this.logger.info('Registering user', { email: userData.email });
    const user = await this.repository.create(userData);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// ✗ WRONG: Tightly coupled
class UserService {
  async registerUser(userData) {
    const user = await UserRepository.create(userData); // Can't mock
    await EmailService.sendWelcome(user.email);       // Can't mock
    return user;
  }
}
```

### Fixture Factories

**CRITICAL**: Use factory functions for consistent test data generation:

```javascript
// ✓ CORRECT: Flexible factories
const userFactory = {
  build: (overrides = {}) => ({
    id: Math.random(),
    email: 'user@test.com',
    name: 'Test User',
    role: 'user',
    createdAt: new Date(),
    ...overrides
  }),
  admin: (overrides = {}) =>
    userFactory.build({ role: 'admin', ...overrides }),
  inactive: (overrides = {}) =>
    userFactory.build({ status: 'inactive', ...overrides })
};

// Usage
it('promotes regular user to admin', () => {
  const user = userFactory.build({ role: 'user' });
  promoteToAdmin(user);
  expect(user.role).toBe('admin');
});

it('removes inactive users', () => {
  const inactive = userFactory.inactive({ email: 'old@test.com' });
  removeInactiveUsers([inactive]);
  expect(database.users).not.toContainEqual(inactive);
});
```

**HIGH**: Factory builder pattern for complex objects:

```javascript
class OrderBuilder {
  constructor() {
    this.order = {
      id: generateId(),
      items: [],
      subtotal: 0,
      tax: 0,
      total: 0,
      status: 'draft',
      createdAt: new Date()
    };
  }

  withItem(product, quantity = 1) {
    const lineItem = { product, quantity, price: product.price * quantity };
    this.order.items.push(lineItem);
    this.order.subtotal += lineItem.price;
    this.order.total = this.order.subtotal * 1.08; // 8% tax
    return this;
  }

  withDiscount(percentage) {
    this.order.discount = this.order.subtotal * (percentage / 100);
    this.order.total = this.order.subtotal - this.order.discount;
    return this;
  }

  build() {
    return { ...this.order };
  }
}

// Fluent API usage
const order = new OrderBuilder()
  .withItem(shoes, 2)
  .withItem(socks, 5)
  .withDiscount(10)
  .build();
```

---

## Integration Testing

### API Testing Patterns

**CRITICAL**: Test API contracts, not implementation details:

```javascript
// ✓ CORRECT: Contract-focused
describe('POST /users', () => {
  it('creates user and returns 201 with Location header', async () => {
    const response = await supertest(app)
      .post('/users')
      .send({ email: 'new@test.com', name: 'New User' })
      .expect(201)
      .expect('Content-Type', /json/)
      .expect('Location', /\/users\/\d+/);

    expect(response.body).toHaveProperty('id');
    expect(response.body.email).toBe('new@test.com');
  });

  it('returns 400 for missing required fields', async () => {
    await supertest(app)
      .post('/users')
      .send({ name: 'No Email' })
      .expect(400)
      .expect(response => {
        expect(response.body.errors).toContainEqual({
          field: 'email',
          message: expect.any(String)
        });
      });
  });
});

// ✗ WRONG: Implementation-focused
describe('User creation', () => {
  it('calls createUserInDatabase function', async () => {
    const spy = jest.spyOn(database, 'createUserInDatabase');
    // ... test just checks function was called
  });
});
```

**HIGH**: Test error conditions and edge cases:

```javascript
describe('POST /users/batch', () => {
  it('returns 207 Multi-Status with partial success', async () => {
    const response = await supertest(app)
      .post('/users/batch')
      .send([
        { email: 'valid@test.com' },        // Valid
        { email: 'invalid-email' },         // Invalid
        { email: 'duplicate@test.com' }     // Duplicate
      ])
      .expect(207);

    expect(response.body.results).toEqual([
      { status: 201, data: { id: expect.any(Number), email: 'valid@test.com' } },
      { status: 400, error: 'Invalid email format' },
      { status: 409, error: 'Email already exists' }
    ]);
  });
});
```

### Database Testing

**CRITICAL**: Use transactions for test isolation:

```javascript
// ✓ CORRECT: Isolated tests with rollback
describe('User Repository', () => {
  let transaction;

  beforeEach(async () => {
    transaction = await db.beginTransaction();
  });

  afterEach(async () => {
    await transaction.rollback();
  });

  it('creates user with auto-generated ID', async () => {
    const user = await userRepository.create(
      { email: 'test@test.com' },
      { transaction }
    );
    expect(user.id).toBeDefined();
    expect(user.createdAt).toBeInstanceOf(Date);
  });

  it('enforces unique email constraint', async () => {
    await userRepository.create(
      { email: 'unique@test.com' },
      { transaction }
    );

    await expect(
      userRepository.create(
        { email: 'unique@test.com' },
        { transaction }
      )
    ).rejects.toThrow('Unique constraint violation');
  });
});
```

**HIGH**: Test query efficiency in integration tests:

```javascript
it('uses index for email lookups', async () => {
  const queryPlan = await db.explain(
    'SELECT * FROM users WHERE email = $1',
    ['test@test.com']
  );

  expect(queryPlan.nodes[0].node_type).toBe('Index Scan');
  expect(queryPlan.execution_time_ms).toBeLessThan(5);
});
```

### Service-to-Service Testing

**MEDIUM**: Mock external services with timeout assertions:

```javascript
describe('PaymentService', () => {
  let paymentGateway;

  beforeEach(() => {
    paymentGateway = {
      charge: jest.fn().mockImplementation(async (amount) => {
        // Simulate network delay
        await new Promise(resolve => setTimeout(resolve, 100));
        return { transactionId: 'tx_123', status: 'success' };
      })
    };
    service = new PaymentService(paymentGateway, { timeout: 5000 });
  });

  it('charges credit card and returns transaction ID', async () => {
    const result = await service.processPayment(100);
    expect(result.transactionId).toBe('tx_123');
  });

  it('handles timeout from external service', async () => {
    paymentGateway.charge.mockImplementation(() =>
      new Promise(resolve => setTimeout(() => resolve(), 6000))
    );

    await expect(service.processPayment(100))
      .rejects.toThrow('Payment gateway timeout');
  });
});
```

---

## E2E Testing

### Playwright Patterns

**CRITICAL**: Use page objects and avoid test interdependence:

```javascript
// Page Object Model
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('input[type="email"]');
    this.passwordInput = page.locator('input[type="password"]');
    this.submitButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator('[role="alert"]');
  }

  async goto() {
    await this.page.goto('/login');
    await this.page.waitForLoadState('networkidle');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectErrorMessage(message) {
    await expect(this.errorMessage).toContainText(message);
  }
}

// Test using page object
describe('Authentication Flow', () => {
  let page, loginPage;

  beforeEach(async ({ browser }) => {
    page = await browser.newPage();
    loginPage = new LoginPage(page);
  });

  afterEach(async () => {
    await page.close();
  });

  it('logs in user with valid credentials', async () => {
    await loginPage.goto();
    await loginPage.login('user@test.com', 'password123');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome, User')).toBeVisible();
  });

  it('shows error for invalid credentials', async () => {
    await loginPage.goto();
    await loginPage.login('user@test.com', 'wrong');

    await loginPage.expectErrorMessage('Invalid credentials');
  });
});
```

**HIGH**: Visual regression testing:

```javascript
it('renders dashboard with correct styling', async ({ page }) => {
  await page.goto('/dashboard');
  await page.waitForLoadState('networkidle');

  expect(await page.screenshot()).toMatchSnapshot('dashboard-light');
});

it('matches dark mode styling', async ({ page }) => {
  await page.goto('/dashboard');
  await page.locator('button[aria-label="Toggle dark mode"]').click();
  await page.waitForTimeout(300); // Animation

  expect(await page.screenshot()).toMatchSnapshot('dashboard-dark');
});
```

### Test Data Management

**CRITICAL**: Use API-driven setup, not UI-driven:

```javascript
// ✓ CORRECT: Fast setup via API
export async function setupTestUser(context) {
  const response = await context.request.post('/api/users', {
    data: { email: 'test@test.com', name: 'Test User' }
  });
  return response.json();
}

// ✗ WRONG: Slow UI navigation
function setupTestUser(page) {
  // Navigates UI, fills forms - slow and fragile
  page.goto('/signup');
  page.fill('input[name="email"]', 'test@test.com');
  // ... tedious UI interaction
}

// Usage in tests
beforeEach(async ({ page, context }) => {
  testUser = await setupTestUser(context);
  await page.goto('/');
  await page.evaluate((userId) => {
    localStorage.setItem('userId', userId);
  }, testUser.id);
});
```

**HIGH**: Parameterized tests for multiple scenarios:

```javascript
describe('Product Filtering', () => {
  const testCases = [
    { filter: 'price:0-50', expectedCount: 5 },
    { filter: 'price:50-100', expectedCount: 8 },
    { filter: 'category:electronics', expectedCount: 12 },
    { filter: 'inStock:true', expectedCount: 15 }
  ];

  testCases.forEach(({ filter, expectedCount }) => {
    it(`shows ${expectedCount} products for "${filter}"`, async ({ page }) => {
      await page.goto(`/products?${filter}`);
      const items = await page.locator('[data-test="product-item"]').count();
      expect(items).toBe(expectedCount);
    });
  });
});
```

---

## Performance Testing

### k6 Load Testing Patterns

**CRITICAL**: Establish baselines before optimization:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend, Counter, Gauge } from 'k6/metrics';

const apiDuration = new Trend('api_duration');
const apiErrors = new Counter('api_errors');
const activeUsers = new Gauge('active_users');

export const options = {
  stages: [
    { duration: '30s', target: 10 },    // Ramp up
    { duration: '2m', target: 100 },    // Stay at load
    { duration: '30s', target: 0 }      // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<500'],   // 99th percentile under 500ms
    http_req_failed: ['rate<0.1'],      // Error rate under 10%
    'api_errors': ['rate<0.05']
  }
};

export default function () {
  activeUsers.set(__VU);

  const response = http.get('https://api.example.com/products');
  const duration = response.timings.duration;
  apiDuration.add(duration);

  if (!response.ok) {
    apiErrors.add(1);
  }

  check(response, {
    'status 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'has product data': (r) => r.json('products') !== null
  });

  sleep(1);
}
```

**HIGH**: Spike testing for unexpected load:

```javascript
export const spikeTest = {
  stages: [
    { duration: '10s', target: 50 },
    { duration: '1s', target: 500 },    // Sudden spike
    { duration: '10s', target: 500 },
    { duration: '5s', target: 0 }
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'],  // Relaxed during spike
    http_req_failed: ['rate<0.25']
  }
};
```

### Baseline Establishment

**CRITICAL**: Document baseline metrics for regression detection:

```bash
# Baseline: Average API response time
# Command: k6 run tests/baseline.js

# Expected Results (Production-like environment):
# - p50: 150ms
# - p95: 350ms
# - p99: 500ms
# - Error rate: < 0.1%
# - Throughput: 2000 req/s with 100 concurrent users
```

---

## Security Testing

### OWASP ZAP Integration

**CRITICAL**: Automate security scanning in CI/CD:

```yaml
# GitHub Actions
- name: OWASP ZAP Scan
  uses: zaproxy/action-full-scan@v0
  with:
    target: 'https://staging.example.com'
    rules_file_name: '.zap/custom-rules.tsv'
    cmd_options: '-a'

- name: Publish ZAP Report
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: zap-report
    path: report_html.html
```

**HIGH**: Input fuzzing for API endpoints:

```javascript
describe('Input Fuzzing', () => {
  const fuzzTargets = [
    '/api/users',
    '/api/products',
    '/api/orders'
  ];

  const fuzzPayloads = [
    '<script>alert("xss")</script>',
    '"; DROP TABLE users; --',
    '../../../etc/passwd',
    '${7*7}',  // Template injection
    'null',
    undefined,
    { nested: { deep: { object: {} } } }
  ];

  fuzzTargets.forEach(endpoint => {
    fuzzPayloads.forEach((payload, i) => {
      it(`resists fuzzing on ${endpoint} with payload ${i}`, async () => {
        const response = await supertest(app)
          .post(endpoint)
          .send({ userInput: payload })
          .expect(400);

        expect(response.body.error).not.toContain(payload);
      });
    });
  });
});
```

### Authentication Testing

**MEDIUM**: Test JWT expiration and refresh:

```javascript
describe('JWT Authentication', () => {
  it('rejects expired tokens', async () => {
    const expiredToken = jwt.sign(
      { userId: 123 },
      secret,
      { expiresIn: '-1h' }
    );

    await supertest(app)
      .get('/api/profile')
      .set('Authorization', `Bearer ${expiredToken}`)
      .expect(401);
  });

  it('refreshes token with valid refresh token', async () => {
    const refreshToken = generateRefreshToken();
    await tokenStore.save(userId, refreshToken);

    const response = await supertest(app)
      .post('/api/refresh')
      .send({ refreshToken })
      .expect(200);

    expect(response.body.accessToken).toBeDefined();
  });
});
```

---

## Coverage Strategy

### Minimum Coverage Requirements

**CRITICAL**: Enforce baseline coverage thresholds:

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/**/*.test.js',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/critical/': {
      branches: 100,
      functions: 100,
      lines: 100
    }
  ]
};
```

### Critical Path Coverage

**CRITICAL**: 100% coverage for security-critical code:

```javascript
// Paths requiring 100% coverage:
// - Authentication/authorization
// - Payment processing
// - Data encryption/decryption
// - Permission checks
// - Error handlers

// Example critical path
describe('PermissionChecker - CRITICAL PATH (100% coverage)', () => {
  it('allows admin to access any resource', () => {
    expect(canAccess({ role: 'admin' }, 'sensitive_data')).toBe(true);
  });

  it('allows user to access own data', () => {
    expect(canAccess(
      { role: 'user', id: 123 },
      'user_123_profile'
    )).toBe(true);
  });

  it('denies user accessing others data', () => {
    expect(canAccess(
      { role: 'user', id: 123 },
      'user_456_profile'
    )).toBe(false);
  });

  it('denies guest any access', () => {
    expect(canAccess({ role: 'guest' }, 'any_data')).toBe(false);
  });
});
```

### What NOT to Test

**HIGH**: Don't test framework internals:

```javascript
// ✗ DON'T: Test framework behavior
it('React.useState returns an array', () => {
  const [state, setState] = useState(0);
  expect(Array.isArray([state, setState])).toBe(true);
});

// ✓ DO: Test your business logic
it('increments counter when button clicked', () => {
  render(<Counter initialValue={0} />);
  fireEvent.click(screen.getByRole('button'));
  expect(screen.getByText('1')).toBeInTheDocument();
});

// ✗ DON'T: Test library behavior
it('lodash.map works correctly', () => {
  expect(_.map([1, 2, 3], x => x * 2)).toEqual([2, 4, 6]);
});

// ✓ DO: Test your domain logic
it('applies discount to order items', () => {
  const items = [{ price: 100 }, { price: 50 }];
  const discounted = applyDiscount(items, 0.10);
  expect(discounted[0].price).toBe(90);
});
```

---

## CI Integration

### Parallel Test Execution

**CRITICAL**: Distribute tests across CI workers:

```yaml
# GitHub Actions: Parallel matrix testing
strategy:
  matrix:
    test-suite: [unit, integration, e2e]
    shard: [1, 2, 3, 4]

steps:
  - name: Run tests (shard ${{ matrix.shard }}/4)
    run: |
      npm test -- \
        --testPathPattern=${{ matrix.test-suite }} \
        --shard=${{ matrix.shard }}/4
```

### Flaky Test Management

**HIGH**: Detect and quarantine flaky tests:

```javascript
// jest.config.js
module.exports = {
  testRunner: 'jest-circus/runner',
  retryTimes: 2,  // Retry failed tests twice
};

// Mark flaky tests for quarantine
describe('Payment Processing', () => {
  it.flaky('handles concurrent payment requests', async () => {
    // Test that sometimes fails due to timing
    const results = await Promise.all([
      processPayment(100),
      processPayment(100)
    ]);
    expect(results).toHaveLength(2);
  });
});
```

**MEDIUM**: Monitor test duration trends:

```bash
# Extract and track test performance
npm test -- --json --outputFile=test-results.json

# Create metrics for regression detection
jq '.testResults[] | {name: .name, duration: .perfStats.duration}' \
  test-results.json >> test-metrics.log
```

### Test Reporting

**HIGH**: Generate comprehensive reports:

```yaml
- name: Generate Test Report
  if: always()
  run: |
    npm test -- \
      --coverage \
      --testResultsProcessor=jest-junit \
      --outputFile=junit.xml

- name: Publish Results
  uses: dorny/test-reporter@v1
  if: always()
  with:
    name: Test Results
    path: junit.xml
    reporter: 'jest-junit'
    fail-on-error: true
```

---

## Framework-Specific Patterns

### Jest (JavaScript/Node.js)

**CRITICAL**: Setup test utilities:

```javascript
// jest.setup.js
beforeEach(() => {
  // Clear mocks between tests
  jest.clearAllMocks();
});

afterEach(() => {
  jest.restoreAllMocks();
});

// Global test utilities
global.testData = {
  user: { id: 1, email: 'test@test.com' },
  product: { id: 101, name: 'Widget', price: 29.99 }
};
```

**HIGH**: Custom matchers for domain logic:

```javascript
expect.extend({
  toBeValidEmail(received) {
    const pass = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(received);
    return {
      pass,
      message: () => `expected ${received} to be a valid email`
    };
  },

  toBeWithinRange(received, min, max) {
    const pass = received >= min && received <= max;
    return {
      pass,
      message: () => `expected ${received} to be within ${min}...${max}`
    };
  }
});

// Usage
expect('user@test.com').toBeValidEmail();
expect(responseTime).toBeWithinRange(100, 500);
```

### Vitest (Modern JavaScript)

**HIGH**: Faster iteration with Vitest:

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html']
    }
  }
});
```

### Pytest (Python)

**CRITICAL**: Fixtures for test setup:

```python
import pytest

@pytest.fixture
def db_session(db):
    """Provide transactional test database."""
    yield db
    db.rollback()

@pytest.fixture
def authenticated_client(client):
    """Provide authenticated API client."""
    client.post('/login', {
        'email': 'test@test.com',
        'password': 'password123'
    })
    return client

def test_user_profile(authenticated_client, db_session):
    response = authenticated_client.get('/api/profile')
    assert response.status_code == 200
    assert response.json()['email'] == 'test@test.com'
```

**HIGH**: Parametrized testing in Python:

```python
@pytest.mark.parametrize('input,expected', [
    ('test@test.com', True),
    ('invalid-email', False),
    ('', False),
    ('user@domain', False)
])
def test_email_validation(input, expected):
    assert is_valid_email(input) == expected
```

### Go Testing

**CRITICAL**: Table-driven tests in Go:

```go
func TestCalculateDiscount(t *testing.T) {
  tests := []struct {
    name        string
    subtotal    float64
    discountPct float64
    expected    float64
  }{
    {"no discount", 100, 0, 100},
    {"10% discount", 100, 10, 90},
    {"50% discount", 100, 50, 50},
  }

  for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
      result := CalculateDiscount(tt.subtotal, tt.discountPct)
      if result != tt.expected {
        t.Errorf("expected %v, got %v", tt.expected, result)
      }
    })
  }
}
```

**HIGH**: Benchmarking in Go:

```go
func BenchmarkUserLookup(b *testing.B) {
  repo := setupRepository()
  b.ReportAllocs()

  for i := 0; i < b.N; i++ {
    repo.GetByID(123)
  }
}

// Run with: go test -bench=. -benchmem
```

---

## Summary: Testing Checklist

- [ ] All units follow AAA pattern
- [ ] Test doubles (stub, mock, spy, fake) used appropriately
- [ ] Factories used for test data
- [ ] API tests verify contracts, not implementation
- [ ] Database tests use transactions for isolation
- [ ] E2E uses page objects, not raw selectors
- [ ] E2E test data setup via API, not UI
- [ ] Performance baselines documented
- [ ] Security tests (fuzzing, auth) included
- [ ] Coverage >= 80%, critical paths 100%
- [ ] No framework internals tested
- [ ] CI runs tests in parallel
- [ ] Flaky tests identified and quarantined
- [ ] Test reports generated and tracked
- [ ] Framework-specific best practices applied
