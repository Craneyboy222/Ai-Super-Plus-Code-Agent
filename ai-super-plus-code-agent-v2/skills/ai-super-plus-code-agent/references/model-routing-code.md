# Model Routing for Code Generation - Expert Reference

## Table of Contents

1. [Task Type Routing](#task-type-routing)
2. [File Type Routing](#file-type-routing)
3. [Phase Routing](#phase-routing)
4. [Cost Optimization](#cost-optimization)
5. [Quality Thresholds](#quality-thresholds)

---

## Task Type Routing

### Architecture = Opus

**CRITICAL**: Complex architectural decisions require best model:

```
TASK: Design system architecture for SaaS platform
DESCRIPTION:
  - Multiple microservices
  - Complex state management
  - Distributed tracing needed
  - Event-driven architecture

COMPLEXITY INDICATORS:
  ✓ Multiple components interact in complex ways
  ✓ Trade-offs between approaches
  ✓ Domain-specific knowledge needed
  ✓ Long-term maintenance considerations
  ✓ Performance/scalability critical

ROUTING: Claude Opus 4.6
REASON: Needs superior reasoning for architectural patterns, trade-off analysis,
        and long-term maintainability decisions

EXPECTED OUTPUT:
  - architecture.md (comprehensive)
  - Component interaction diagrams
  - Technology selection justification
  - Scalability analysis
  - Deployment strategy
```

### CRUD = Sonnet

**HIGH**: Straightforward CRUD operations use faster model:

```
TASK: Generate user CRUD endpoints
DESCRIPTION:
  - Standard REST API endpoints
  - Create, Read, Update, Delete
  - Basic validation
  - Database queries

COMPLEXITY INDICATORS:
  ✓ Well-established patterns
  ✓ No novel trade-offs
  ✓ Clear requirements
  ✓ Straightforward implementation
  ✗ No complex state management
  ✗ No novel algorithms

ROUTING: Claude 3.5 Sonnet
REASON: Proven patterns, straightforward implementation, speed important

EXPECTED LATENCY: ~2-5 seconds per endpoint
EXPECTED OUTPUT:
  - Route handlers
  - Controllers
  - Database queries
  - Basic tests
```

### Scanning = Haiku

**HIGH**: Parsing and analysis use most efficient model:

```
TASK: Analyze test results and generate report
DESCRIPTION:
  - Parse JSON test output
  - Summarize test results
  - Generate report markdown
  - Identify failures

COMPLEXITY INDICATORS:
  ✓ Mechanical processing
  ✓ No reasoning required
  ✓ Pattern matching only
  ✓ Deterministic output
  ✗ No creative decisions
  ✗ No complex reasoning

ROUTING: Claude Haiku
REASON: Simple parsing/formatting, cost-critical, speed excellent

EXPECTED LATENCY: <1 second
EXPECTED COST: $0.01-0.05
EXPECTED OUTPUT:
  - Test report
  - Failure summary
  - Statistics
```

---

## File Type Routing

### Security-Critical Files → Opus

**CRITICAL**: Authentication, encryption, payment → Opus only:

```
SECURITY-CRITICAL FILES:
  ✓ src/auth/authentication.ts        (Opus)
  ✓ src/auth/authorization.ts         (Opus)
  ✓ src/security/encryption.ts        (Opus)
  ✓ src/payment/processor.ts          (Opus)
  ✓ src/security/secrets.ts           (Opus)
  ✓ src/security/rate-limiter.ts      (Opus)

REASONING:
  - Security vulnerabilities are catastrophic
  - Need deep security knowledge
  - Need threat modeling
  - Cryptography requires expertise
  - PCI-DSS compliance critical
  - Authorization bugs are critical

QUALITY REQUIREMENTS:
  - Correctness: 10/10
  - Security: 10/10
  - No shortcuts allowed
  - Code review required
  - Penetration testing
```

### Complex Business Logic → Opus

**CRITICAL**: State machines, complex workflows → Opus:

```
COMPLEX BUSINESS LOGIC FILES:
  ✓ src/services/order-fulfillment.ts (Opus)
  ✓ src/services/payment-workflow.ts   (Opus)
  ✓ src/state-machines/checkout.ts    (Opus)
  ✓ src/services/pricing-engine.ts    (Opus)

REASONING:
  - Multiple branches and edge cases
  - Complex state transitions
  - Error scenarios critical
  - Business impact high
  - Changes are frequent
  - Bugs are expensive

EXAMPLE: Order Fulfillment State Machine
  States: Pending → Confirmed → Preparing → Shipped → Delivered
  Transitions: Multiple per state
  Error handling: Partial shipment, cancellation, refunds
  Complexity: High decision tree

ROUTING: Opus
REASON: Needs to reason about all possible states and transitions
```

### Standard CRUD/ORM → Sonnet

**HIGH**: Standard database models and CRUD → Sonnet:

```
STANDARD FILES:
  ✓ src/models/User.ts                (Sonnet)
  ✓ src/models/Product.ts             (Sonnet)
  ✓ src/repositories/UserRepo.ts      (Sonnet)
  ✓ src/repositories/ProductRepo.ts   (Sonnet)
  ✓ src/controllers/users.ts          (Sonnet)
  ✓ src/routes/products.ts            (Sonnet)

REASONING:
  - Established ORM patterns
  - Straightforward queries
  - Standard CRUD operations
  - Predictable structure
  - Speed matters for volume

CHARACTERISTICS:
  - High volume of files
  - Simple to generate
  - Easily testable
  - Quick to implement
  - Low error rate with templates
```

### Configuration Files → Haiku

**MEDIUM**: Config generation, simple formatting → Haiku:

```
CONFIGURATION FILES:
  ✓ .env.example                      (Haiku)
  ✓ docker-compose.yml                (Haiku)
  ✓ webpack.config.js                 (Haiku)
  ✓ tsconfig.json                     (Haiku)
  ✓ .github/workflows/ci.yml          (Haiku)
  ✓ nginx.conf                        (Haiku)

REASONING:
  - Mechanical generation
  - Templates available
  - No reasoning needed
  - Cost critical
  - High volume

COST COMPARISON:
  Config via Opus: ~20 seconds, $0.50
  Config via Haiku: ~3 seconds, $0.05
  Savings: 90% cost, 85% faster
```

### Tests (Simple) → Sonnet

**MEDIUM**: Unit tests for straightforward functions → Sonnet:

```
TEST FILES FOR SIMPLE FUNCTIONS:
  ✓ tests/unit/utils/formatters.test.ts     (Sonnet)
  ✓ tests/unit/models/User.test.ts          (Sonnet)
  ✓ tests/unit/repositories/User.test.ts    (Sonnet)
  ✓ tests/unit/helpers/validation.test.ts   (Sonnet)

REASONING:
  - Clear test patterns
  - Straightforward assertions
  - Mocking is standard
  - Good template coverage
  - High volume needed

CHARACTERISTICS:
  - Predictable test structure
  - Cover obvious cases
  - Standard mocking patterns
  - Quick to implement
```

### Tests (Complex) → Opus

**HIGH**: Integration and E2E tests → Opus:

```
TEST FILES FOR COMPLEX LOGIC:
  ✓ tests/integration/order-service.test.ts    (Opus)
  ✓ tests/integration/payment-flow.test.ts     (Opus)
  ✓ tests/e2e/checkout.test.ts                 (Opus)
  ✓ tests/integration/authorization.test.ts    (Opus)

REASONING:
  - Need to understand complex interactions
  - Edge cases critical
  - Security/authorization testing
  - Multiple components involved
  - Bugs are expensive

CHARACTERISTICS:
  - Complex setup/teardown
  - Multiple assertion paths
  - Error scenario coverage
  - Edge cases critical
  - Need deep domain knowledge
```

### Documentation → Haiku

**MEDIUM**: Generated documentation → Haiku:

```
DOCUMENTATION FILES:
  ✓ docs/API.md          (Haiku - from OpenAPI spec)
  ✓ docs/SETUP.md        (Haiku - from comments)
  ✓ CONTRIBUTING.md      (Haiku - template-based)
  ✓ CHANGELOG.md         (Haiku - from git log)
  ✓ docs/TROUBLESHOOTING.md (Haiku - Q&A format)

REASONING:
  - Generated from code/templates
  - No creative input needed
  - Fast turnaround important
  - Cost critical for large docs
  - Haiku sufficient for documentation

COST SAVINGS:
  20-page API doc via Opus: $1.00, 60 seconds
  20-page API doc via Haiku: $0.10, 10 seconds
  Savings: 90% cost, 83% faster
```

---

## Phase Routing

### Phase-by-Phase Model Selection

**CRITICAL**: Different models for different phases:

```
PHASE 1: Architecture & Planning
  Primary: Opus
  Secondary: Haiku (for documentation)
  Files: architecture.md, design decisions
  Duration: 30-60 minutes
  Cost: ~$2

PHASE 2: Schema Design
  Primary: Sonnet
  Secondary: Haiku (for migrations)
  Files: schema.sql, migrations
  Duration: 15-30 minutes
  Cost: ~$0.50

PHASE 3: Core Models
  Primary: Opus (complex validation)
  Secondary: Sonnet (simple models)
  Files: type definitions, entities
  Duration: 20-40 minutes
  Cost: ~$1

PHASE 4: Repository Layer
  Primary: Sonnet
  Secondary: Haiku (tests for simple repos)
  Files: database access, ORM models
  Duration: 20-30 minutes
  Cost: ~$0.75

PHASE 5: Service Layer
  Primary: Opus (complex logic)
  Secondary: Sonnet (simple services)
  Files: business logic, orchestration
  Duration: 40-60 minutes
  Cost: ~$2

PHASE 6: API Layer
  Primary: Sonnet
  Secondary: Haiku (config)
  Files: routes, controllers, middleware
  Duration: 30-45 minutes
  Cost: ~$1

PHASES 7-8: Unit & Integration Tests
  Primary: Sonnet (unit), Opus (integration)
  Secondary: Haiku (test fixtures)
  Files: test suites
  Duration: 45-90 minutes
  Cost: ~$2

PHASE 9: E2E Tests
  Primary: Opus
  Files: end-to-end test scenarios
  Duration: 30-45 minutes
  Cost: ~$1.50

PHASE 10: Documentation
  Primary: Haiku
  Secondary: Opus (complex docs)
  Files: API docs, guides, troubleshooting
  Duration: 20-30 minutes
  Cost: ~$0.50

PHASE 11: DevOps
  Primary: Sonnet
  Secondary: Haiku (config generation)
  Files: Docker, K8s, CI/CD
  Duration: 30-45 minutes
  Cost: ~$0.75

PHASE 12: Performance
  Primary: Opus
  Files: optimization strategies, load tests
  Duration: 30-60 minutes
  Cost: ~$1.50

PHASE 13: Security
  Primary: Opus
  Files: security audit, vulnerability fixes
  Duration: 45-90 minutes
  Cost: ~$2

PHASE 14: Integration & Review
  Primary: Opus
  Files: final integration, validation
  Duration: 60-120 minutes
  Cost: ~$2.50

TOTAL PROJECT:
  Estimated time: ~8-14 hours
  Estimated cost: ~$17-20
  Model breakdown:
    - Opus: 60% of work, 70% of cost
    - Sonnet: 30% of work, 25% of cost
    - Haiku: 10% of work, 5% of cost
```

---

## Cost Optimization

### Cost-Aware Routing

**HIGH**: Balance quality and cost:

```
COST VS QUALITY MATRIX:

Perfect Quality (Opus):
  Cost per 1000 tokens: $15 (input), $45 (output)
  Speed: ~30 sec per request
  Use for: Architecture, security, complex logic, integration tests

Good Quality (Sonnet):
  Cost per 1000 tokens: $3 (input), $12 (output)
  Speed: ~5 sec per request
  Use for: CRUD, standard implementations, tests

Fast & Cheap (Haiku):
  Cost per 1000 tokens: $0.25 (input), $1.25 (output)
  Speed: <2 sec per request
  Use for: Config, parsing, documentation

COST EXAMPLES:

1000-line project:

Full Opus:
  Est. cost: ~$40-60
  Risk: Over-engineered, slow

Optimized (Opus + Sonnet + Haiku):
  Est. cost: ~$15-20
  Risk: Lower, appropriate for task
  Quality: Still excellent

Cheapest (Haiku for everything):
  Est. cost: ~$5-10
  Risk: Very high, poor quality
  Quality: Unacceptable
```

### Token Budgeting

**HIGH**: Allocate tokens by phase importance:

```
10,000 token budget:

Architecture (Critical):
  Tokens: 3,000
  Model: Opus
  Files: architecture.md, decision documents
  Reasoning: Foundation is critical

Core Logic (High):
  Tokens: 4,000
  Model: Opus
  Files: Services, authorization, state machines
  Reasoning: Business logic complexity

Standard Implementation (Medium):
  Tokens: 2,000
  Model: Sonnet
  Files: CRUD, repositories, routes
  Reasoning: Proven patterns, quick generation

Tests & Docs (Low):
  Tokens: 1,000
  Model: Haiku/Sonnet
  Files: Tests, documentation, config
  Reasoning: Can regenerate if needed
```

---

## Quality Thresholds

### When to Upgrade Model

**CRITICAL**: Triggers for using better model:

```
AUTOMATIC UPGRADE TO OPUS:

Trigger 1: Test Failures
  - If Sonnet-generated code fails tests: Use Opus
  - If >2 tests fail: Definitely Opus
  - Example: Repository query returns wrong data

Trigger 2: Security Issues
  - Any security concern: Upgrade to Opus
  - Missing validation: Opus
  - Missing error handling: Opus

Trigger 3: Complexity Discovery
  - If task becomes more complex: Upgrade to Opus
  - If edge cases appear: Opus
  - If state management needed: Opus

Trigger 4: Quality Score Below Threshold
  - If quality score < 7: Use Opus
  - If multiple dimensions < 6: Opus
  - Code review fails: Opus

DECISION FLOW:

  Initial: Use Sonnet
     ↓
  [Test/Quality Check]
     ↙          ↘
  ✓ Pass    ✗ Fail/Low Quality
     ↓              ↓
  Accept      Upgrade to Opus
     ↓              ↓
  Continue     [Regenerate]
                    ↓
              [Test/Quality Check]
                    ↓
                  ✓ Pass
```

### Quality Gates per Model

**HIGH**: Minimum acceptable quality by model:

```
SONNET QUALITY GATES:
  - Correctness: >= 8
  - Completeness: >= 8
  - Security: >= 8
  - Tests: >= 80% coverage
  - Lint: 0 errors

  If any gate fails: Regenerate with Opus

HAIKU QUALITY GATES:
  - Correctness: >= 7 (lower for simple tasks)
  - Lint: 0 errors
  - Format: Valid syntax

  If fails: Regenerate with Sonnet

OPUS EXPECTED:
  - Correctness: >= 9
  - Completeness: >= 9
  - Security: >= 9
  - Tests: >= 90% coverage

  If falls short: Regenerate/iterate
```

### Cost/Quality Trade-off

**MEDIUM**: Know when to spend more:

```
ALWAYS USE OPUS (security > cost):
  ✓ Authentication/authorization
  ✓ Payment processing
  ✓ Encryption
  ✓ Data validation (sensitive data)
  ✓ State machines
  ✓ Integration tests
  ✓ Deployment code

ALWAYS USE SONNET (good balance):
  ✓ CRUD operations
  ✓ API routes
  ✓ ORM models
  ✓ Unit tests
  ✓ Standard repositories
  ✓ Service orchestration

ALWAYS USE HAIKU (speed/cost matters):
  ✓ Documentation
  ✓ Configuration files
  ✓ Data parsing
  ✓ Report generation
  ✓ Simple utilities

FLEXIBLE (depends on complexity):
  ? Service layers (Opus if complex, Sonnet if simple)
  ? Tests (Opus for integration, Sonnet for unit)
  ? DevOps (Sonnet normally, Haiku for templates)
```

---

## Summary: Routing Decision Tree

```
START: I need code generated

1. Is it security-critical?
   YES → Opus (no exceptions)
   NO → Continue

2. Is it complex business logic?
   YES → Opus
   NO → Continue

3. Is it integration/E2E test?
   YES → Opus
   NO → Continue

4. Is it CRUD or standard pattern?
   YES → Sonnet
   NO → Continue

5. Is it configuration, docs, or simple parsing?
   YES → Haiku
   NO → Continue

6. Is it high-impact file?
   YES → Opus
   NO → Sonnet (default)

FINAL DECISION:
  - Opus: Security, complex logic, high-impact, integration tests
  - Sonnet: CRUD, standard implementation, unit tests
  - Haiku: Config, docs, parsing, utilities
```

---

## Routing Checklist

- [ ] Each file type has assigned model
- [ ] Security-critical → Opus only
- [ ] Complex logic → Opus
- [ ] Standard CRUD → Sonnet
- [ ] Config/docs → Haiku
- [ ] Quality gates defined per model
- [ ] Cost tracking implemented
- [ ] Upgrade triggers documented
- [ ] Phase routing finalized
- [ ] Token budget allocated
- [ ] Fallback strategy clear (regenerate with Opus on failure)
