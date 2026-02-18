# Agent Orchestration for Code Projects - Expert Reference

## Table of Contents

1. [Agent Spawning Strategy](#agent-spawning-strategy)
2. [Parallel Execution Patterns](#parallel-execution-patterns)
3. [Agent Communication Protocol](#agent-communication-protocol)
4. [Verification Chain](#verification-chain)
5. [Conflict Resolution](#conflict-resolution)
6. [Context Sharing](#context-sharing)

---

## Agent Spawning Strategy

### Phase-Based Agent Assignment

**CRITICAL**: Different agents for different code phases:

```
PROJECT PHASES AND AGENT ROLES:

Phase 1: Architecture & Planning (Opus)
  - Parse requirements
  - Design system architecture
  - Create module plan
  - Define interfaces/contracts
  Output: architecture.md, module-map.json

Phase 2: Schema & Configuration (Sonnet)
  - Design database schema
  - Create config files
  - Define environment variables
  Output: schema.sql, config/*.yml, .env.example

Phase 3: Core Models/Entities (Opus)
  - Generate domain models
  - Create type definitions
  - Implement validation
  Output: src/models/*, src/types/*

Phase 4: Repository Layer (Sonnet)
  - Generate database access layer
  - Implement query builders
  - Create ORM mappings
  Output: src/repositories/*

Phase 5: Service Layer (Opus)
  - Business logic implementation
  - Service coordination
  - Complex transformations
  Output: src/services/*

Phase 6: API Layer (Sonnet)
  - Route definitions
  - Request/response handlers
  - Middleware setup
  Output: src/routes/*, src/controllers/*

Phase 7: Tests - Unit (Sonnet)
  - Unit test generation
  - Mock/fixture creation
  Output: tests/unit/*

Phase 8: Tests - Integration (Opus)
  - API/database tests
  - Service orchestration tests
  Output: tests/integration/*

Phase 9: Tests - E2E (Sonnet)
  - UI/workflow tests
  - Complete feature tests
  Output: tests/e2e/*

Phase 10: Documentation (Haiku)
  - API documentation
  - Setup guides
  - Troubleshooting
  Output: docs/*

Phase 11: DevOps (Sonnet)
  - Docker/Kubernetes configs
  - CI/CD pipelines
  - Deployment scripts
  Output: docker/*, k8s/*, .github/workflows/*

Phase 12: Performance (Opus)
  - Query optimization
  - Caching strategy
  - Load testing
  Output: optimization-report.md, load-tests/

Phase 13: Security (Opus)
  - Security audit
  - Vulnerability fixes
  - Compliance checks
  Output: security-audit.md

Phase 14: Integration & Review (Opus)
  - Full integration
  - Code review
  - Final QA
  Output: final-report.md
```

### Model Selection by File Type

**CRITICAL**: Route models by file characteristics:

```
SECURITY-CRITICAL FILES (Opus):
  - Authentication modules
  - Authorization checks
  - Encryption/decryption
  - Payment processing
  - Sensitive data handling
  → Use: Claude Opus 4.6

COMPLEX BUSINESS LOGIC (Opus):
  - Service layers
  - State machines
  - Algorithm implementations
  - Complex transformations
  → Use: Claude Opus 4.6

STANDARD CRUD/BOILERPLATE (Sonnet):
  - Controllers/Routes
  - ORM models
  - Repositories
  - Utility functions
  - Standard patterns
  → Use: Claude 3.5 Sonnet

SIMPLE/MECHANICAL (Haiku):
  - Documentation generation
  - Config files
  - Tests (simple cases)
  - Linting/formatting
  - Log analysis
  → Use: Claude Haiku

SCANNING/ANALYSIS (Haiku):
  - Codebase structure analysis
  - Test result parsing
  - Log examination
  - Dependency tree analysis
  → Use: Claude Haiku
```

### Spawn Conditions

**HIGH**: Spawn agents based on readiness:

```
Agent spawning requires:
1. Dependencies completed (previous phase files exist)
2. Context files available (architecture, interfaces)
3. Parent agent confirms readiness
4. Input validation passed
5. Resource availability confirmed

Example: Can't spawn Service Layer agent until:
  ✓ Repository Layer files exist and are complete
  ✓ Model definitions are available
  ✓ Interface contracts are clear
  ✓ Architecture diagram shows service boundaries
  ✓ Parent agent gave completion signal
```

---

## Parallel Execution Patterns

### Independent Workstreams

**CRITICAL**: Maximize parallelism with dependency awareness:

```
DEPENDENCY DAG (can parallelize):

  Architecture Design
        ↓
  ┌─────┼─────┬──────┐
  ↓     ↓     ↓      ↓
Schema Core  Service API Tests
Models Repo  Layer   Layer (Unit)
  ↓     ↓    ↓      ↓    ↓
  └─────┴────┴──────┴────┘
        ↓
    Integration
      Tests
        ↓
    E2E Tests
        ↓
    Documentation
        ↓
    DevOps
        ↓
    Performance

PARALLEL GROUPS (can run simultaneously):
Group 1 (Parallel):
  - Schema Design (Sonnet)
  - Core Models (Opus)
  - Repository Layer (Sonnet)
  - All can start after Architecture

Group 2 (Parallel, after Group 1):
  - Service Layer (Opus)
  - API Layer (Sonnet)
  - Can run together after repos/models done

Group 3 (Parallel, after Group 2):
  - Unit Tests (Sonnet)
  - Integration Tests (Opus)
  - Can run in parallel if fixtures created

Sequential Required:
  - E2E Tests (after API complete)
  - Documentation (after code complete)
  - DevOps (after code + docs)
  - Performance (after deployment ready)
```

### Parallel Task Distribution

**HIGH**: Distribute work across agents:

```
Example: API Endpoint Generation

Sequential (must happen in order):
  1. Endpoint design (Opus)
     - Create request/response contracts
     - Define error cases
     Output: endpoint.contract.json

  2. Route handler (Sonnet)
     - Implement route based on contract
     - Use provided service interfaces
     Output: routes/users.js

  3. Validation (Sonnet)
     - Create input validators
     - Use contract from step 1
     Output: validators/userValidators.js

  4. Tests (Sonnet)
     - Generate tests based on handler
     - Use contract from step 1
     Output: tests/unit/routes/users.test.js

Parallel capable:
  - All endpoints with same contract format CAN be done by different agents
  - All tests for endpoints CAN be done in parallel after handlers complete
```

---

## Agent Communication Protocol

### Shared Context Format

**CRITICAL**: Standardized context exchange:

```json
{
  "project_id": "proj_abc123",
  "phase": 5,
  "phase_name": "Service Layer",
  "status": "in_progress",
  "timestamp": "2024-01-15T10:30:00Z",

  "architecture": {
    "services": [
      {
        "name": "UserService",
        "responsibilities": ["user management", "validation"],
        "dependencies": ["UserRepository", "EmailService"],
        "file": "src/services/UserService.ts"
      }
    ],
    "models": [
      {
        "name": "User",
        "fields": [
          { "name": "id", "type": "number", "required": true },
          { "name": "email", "type": "string", "required": true }
        ],
        "file": "src/models/User.ts"
      }
    ]
  },

  "completed_files": [
    "src/models/User.ts",
    "src/repositories/UserRepository.ts"
  ],

  "in_progress_files": [
    "src/services/UserService.ts"
  ],

  "blocked_on": [],
  "blocking": [],

  "interfaces": {
    "UserRepository": {
      "methods": [
        {
          "name": "findById",
          "params": [{ "name": "id", "type": "number" }],
          "returns": "Promise<User | null>",
          "file": "src/repositories/UserRepository.ts#L10"
        }
      ],
      "file": "src/repositories/UserRepository.ts"
    }
  },

  "errors": [],
  "warnings": [
    "UserService.updateUser has no type hints"
  ]
}
```

### File Manifest

**HIGH**: Track all generated files:

```json
{
  "manifest": {
    "version": "1.0",
    "generated_at": "2024-01-15T10:30:00Z",
    "total_files": 47,
    "files": [
      {
        "path": "src/models/User.ts",
        "phase": 3,
        "generated_by": "claude-opus-4-6",
        "status": "complete",
        "test_coverage": 95,
        "lines_of_code": 120,
        "dependencies": [],
        "hash": "sha256:abc..."
      },
      {
        "path": "src/repositories/UserRepository.ts",
        "phase": 4,
        "generated_by": "claude-3-5-sonnet",
        "status": "complete",
        "test_coverage": 88,
        "lines_of_code": 200,
        "dependencies": ["User.ts"],
        "hash": "sha256:def..."
      }
    ],
    "statistics": {
      "total_lines": 5420,
      "total_functions": 143,
      "average_coverage": 87,
      "quality_score": 8.6
    }
  }
}
```

### Agent Handoff Message

**MEDIUM**: Clear communication at phase transitions:

```
FROM: Opus Agent (Phase 3: Core Models)
TO: Sonnet Agent (Phase 4: Repository Layer)
TIMESTAMP: 2024-01-15T10:30:00Z

PHASE COMPLETE: Core Models Generated
STATUS: ✓ All models complete, tested, and documented

DELIVERABLES:
  ✓ src/models/User.ts (95% coverage)
  ✓ src/models/Product.ts (92% coverage)
  ✓ src/models/Order.ts (88% coverage)
  ✓ src/types/index.ts (all types defined)

CONTEXT FOR NEXT PHASE:
  - All models use strict TypeScript types
  - Validation rules in model constructors
  - See architecture.md for relationships
  - Interface contracts in interfaces/

DEPENDENCIES FOR YOU (Sonnet):
  - Implement repositories matching these interfaces:
    * findById(id: number): Promise<T>
    * create(data: CreateInput): Promise<T>
    * update(id: number, data: UpdateInput): Promise<T>
    * delete(id: number): Promise<boolean>
    * find(filter: Filter): Promise<T[]>

BLOCKERS: None
WARNINGS: None

NEXT STEPS:
  1. Generate repositories for each model
  2. Ensure all queries use indexes
  3. Create tests with 85%+ coverage
  4. Signal when complete

Ready for next phase!
```

---

## Verification Chain

### Generate → Lint → Test → Review → Approve

**CRITICAL**: Each file goes through verification:

```
FILE GENERATION FLOW:

┌─────────────────────────────────────────────┐
│ 1. GENERATE                                 │
│ - Agent writes code                         │
│ - Follows templates and patterns            │
│ - Includes comments and types              │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 2. LINT                                     │
│ - Run: eslint, prettier, tsc               │
│ - Check for: syntax errors, style issues   │
│ - Auto-fix: formatting, simple issues      │
│ - FAIL if: errors cannot be auto-fixed     │
└─────────────────────────────────────────────┘
                    ↓
            ┌───────┴────────┐
            ↓                ↓
      ✓ PASS            ✗ FAIL
        ↓                 ↓
┌──────────────┐  ┌─────────────────┐
│ 3. TEST      │  │ Agent fixes and │
│              │  │ re-generates    │
│ Run unit     │  └─────────────────┘
│ tests        │          ↓
│              │  Re-run lint
│ - Coverage   │  (loop until pass)
│   >= 80%     │
│ - All pass   │
└──────────────┘
      ↓
┌─────────────────────────────────────────────┐
│ 4. REVIEW (Automated)                       │
│ - Check quality rubric (section 6, 7, 8)   │
│ - Verify no duplicated code                │
│ - Check error handling complete            │
│ - Verify input validation present          │
│ - Check documentation present              │
└─────────────────────────────────────────────┘
      ↓
    ┌─┴─────────┐
    ↓           ↓
✓ PASS      ✗ NEEDS WORK
  ↓           ↓
  │      Agent improves:
  │      - Add missing validation
  │      - Improve docs
  │      - Fix antipatterns
  │           ↓
  │    Re-run full chain
  │
  ↓
┌─────────────────────────────────────────────┐
│ 5. APPROVE                                  │
│ - File meets all quality criteria           │
│ - Safe to integrate with other files        │
│ - Ready for next phase                      │
└─────────────────────────────────────────────┘
```

### Quality Gate Checks

**CRITICAL**: Automated verification:

```yaml
# quality-gates.yaml
gates:
  syntax:
    - command: "eslint {file}"
      fail_on_error: true
      auto_fix: true

  types:
    - command: "tsc --noEmit {file}"
      fail_on_error: true
      auto_fix: false

  tests:
    - command: "jest {file}.test.ts"
      fail_on_error: true
      coverage_minimum: 80
      auto_fix: false

  security:
    - command: "npm audit"
      fail_on_error: true
      auto_fix: false

  style:
    - command: "prettier --check {file}"
      fail_on_error: false
      auto_fix: true

  review:
    - check: "error_handling_complete"
      fail_on_missing: true
    - check: "input_validation_present"
      fail_on_missing: true
    - check: "documentation_present"
      fail_on_missing: true
    - check: "no_code_duplication"
      fail_on_missing: true

  quality_score:
    minimum: 7.0  # From quality-rubric
    dimensions:
      correctness: 8
      completeness: 8
      robustness: 7
      performance: 6
      security: 8
      maintainability: 7
      idiomatic: 7
      documentation: 6
```

---

## Conflict Resolution

### File Conflict Detection

**CRITICAL**: Detect and resolve edit conflicts:

```
CONFLICT SCENARIOS:

1. PARALLEL FILE GENERATION CONFLICT
   Problem: Two agents generate the same file simultaneously

   Detection:
   - File A: UserService.ts (Opus, started 10:00)
   - File B: UserService.ts (Sonnet, started 10:01)

   Resolution:
   - Abort File B immediately
   - Use File A (earlier timestamp)
   - Re-route File B agent to next task

2. DEPENDENCY CONFLICT
   Problem: File expects interface that changed

   Example:
   - UserRepository.ts: expects findById(id: number)
   - UserService.ts: calls findById(email: string)

   Detection: Type checker fails

   Resolution:
   1. Identify conflict in build output
   2. Sync both files to agreement
   3. Opus agent decides interface contract
   4. Sonnet agent updates implementation

3. ORDERING CONFLICT
   Problem: Agent generated file before dependency

   Example:
   - UserService.ts depends on UserRepository.ts
   - But UserRepository.ts not yet generated

   Detection: Import fails

   Resolution:
   1. Queue UserService.ts
   2. Generate UserRepository.ts first
   3. Then resume UserService.ts

4. CIRCULAR DEPENDENCY CONFLICT
   Problem: Module A imports B, B imports A

   Detection: Bundler fails, circular dependency warning

   Resolution:
   1. Identify circular path
   2. Opus agent redesigns to break cycle
   3. Introduce interface/abstraction
   4. Regenerate both modules
```

### Conflict Resolution Priority

**HIGH**: Strict resolution order:

```
CONFLICT RESOLUTION PRIORITY:

1. CRITICAL (must resolve)
   - Type errors: Opus makes final decision
   - Circular dependencies: Opus redesigns
   - Missing required interfaces: Opus defines

2. HIGH (must resolve before proceeding)
   - Test failures: Agent who wrote code fixes
   - Lint errors: Auto-fix or agent fixes
   - Missing required docs: Agent adds

3. MEDIUM (should resolve)
   - Code style inconsistency: Auto-fix
   - Duplicate code: Refactor to shared utility
   - Incomplete error handling: Agent improves

4. LOW (nice to have)
   - Performance optimization: Mark as TODO
   - Code comments: Add incrementally
   - Alternative approaches: Document in ADR
```

---

## Context Sharing

### What Each Agent Needs

**CRITICAL**: Right context for right agent:

```
OPUS AGENT (complex decisions) needs:
  ✓ Full architecture
  ✓ All completed code files
  ✓ Test results
  ✓ Performance requirements
  ✓ Security policies
  ✓ Business logic context
  ✗ Line-by-line test details
  ✗ Individual config values

  Context Size: ~200KB (full codebase reference)

SONNET AGENT (standard implementation) needs:
  ✓ Architecture for their module
  ✓ Interface contracts
  ✓ Completed dependencies
  ✓ Test requirements
  ✓ Code patterns/templates
  ✗ Full codebase (only relevant files)
  ✗ Security audit details

  Context Size: ~50KB (relevant modules)

HAIKU AGENT (simple tasks) needs:
  ✓ Task specification
  ✓ Input format/structure
  ✓ Output requirements
  ✓ Relevant examples
  ✗ Full codebase
  ✗ Architecture decisions
  ✗ Historical context

  Context Size: ~10KB (specific task inputs)
```

### Progressive Context Loading

**HIGH**: Load context incrementally:

```
CONTEXT LOAD STRATEGY:

Phase 1: Architecture (Small)
  - Load: requirements.md, architecture.md
  - Agent: Opus
  - Size: ~50KB

Phase 2: Schemas (Medium)
  - Load: architecture.md, schema.sql, models/
  - Agent: Sonnet, Opus
  - Size: ~100KB

Phase 3: Core Models (Medium)
  - Load: schema.sql, models/, types/
  - Agent: Opus
  - Size: ~100KB

Phase 4: Repositories (Medium)
  - Load: models/, repositories.template, tests/
  - Agent: Sonnet
  - Size: ~100KB

Phase 5: Services (Large)
  - Load: models/, repositories/, services.template, tests/
  - Agent: Opus
  - Size: ~150KB

Phase 6+: Selective Loading
  - Load: Only files being used
  - Agent: Sonnet
  - Size: ~50-100KB per agent

Key principle: Don't load context you won't use
```

### File Dependency Graph

**MEDIUM**: Understand dependencies for parallel work:

```
Example dependency graph for API project:

models/User.ts (0 dependencies)
    ↑
    └── repositories/UserRepository.ts
             ↑
             ├── services/UserService.ts
             │      ↑
             │      └── routes/users.ts
             │
             └── services/AuthService.ts (also uses User)
                    ↑
                    └── middleware/auth.ts
                           ↑
                           └── routes/all

Can parallelize:
  - User model + Product model (no dependency)
  - UserRepository + ProductRepository (no dependency)
  - UserService + AuthService (both use same repo)

Cannot parallelize:
  - UserRepository before User model
  - UserService before UserRepository
  - routes before services
```

---

## Summary: Orchestration Checklist

- [ ] Phase sequence defined and documented
- [ ] Agent model selected for each phase
- [ ] Dependency graph identified
- [ ] Parallel opportunities maximized
- [ ] Shared context format standardized
- [ ] File manifest created
- [ ] Handoff messages clear
- [ ] Verification chain automated
- [ ] Quality gates configured
- [ ] Conflict resolution process clear
- [ ] Context loading optimized
- [ ] Each agent has required context
- [ ] No circular dependencies
- [ ] Tests run after each phase
- [ ] Documentation generated incrementally
