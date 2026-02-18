# Command: build

**Description**: Build a complete project from specification (all 14 phases)

**Usage**: `/build`

---

## Process

### 1. Gather Requirements
- Ask user for detailed requirements
- Clarify scope, timeline, tech preferences
- Identify constraints and non-functional requirements

### 2. Route to Architect
**Agent**: Architect (Claude Opus 4.6)

```
Tasks:
├── Analyze requirements
├── Select architecture pattern
├── Select tech stack
├── Design data model
├── Design API surface
├── Create build order
└── Generate Architecture Decision Record (ADR)
```

### 3. Phase Execution (Parallel where possible)
```
Timeline Estimation:
├── Phases 0-2 (Design): 1-2 hours
├── Phase 3 (Scaffold): 30 minutes
├── Phases 4-7 (Core generation): 3-4 hours
├── Phase 8 (Infrastructure): 1 hour
├── Phase 9 (Tests): 2-3 hours
├── Phase 10 (Security): 1-2 hours
├── Phase 11 (Quality Gate): 1 hour
├── Phase 12 (Documentation): 1-2 hours
└── Phase 13 (Ship): 30 minutes

Total: 10-16 hours for complete APP
```

### 4. Deliverables
```
Outputs:
├── Project scaffolding (directories, configs)
├── Database migrations
├── Backend API (routes, handlers, middleware)
├── Business logic layer
├── Frontend components and pages
├── Test suites (unit, integration, e2e)
├── Infrastructure configs (Docker, K8s, CI/CD)
├── Complete documentation
├── Security audit report
└── Deployment guide
```

### 5. Quality Gates
```
Before Ship:
☐ All code reviewed
☐ Tests passing (80%+ coverage)
☐ No TypeScript errors
☐ No ESLint warnings
☐ Security audit passed
☐ Performance targets met
☐ Documentation complete
☐ Build succeeds
```

---

## Example Execution

```
User: Build me a SaaS blog platform with user auth, posts, comments.

Agent (Architect):
  ✓ Phase 0: Complexity = APP
  ✓ Phase 1: Selected Monolith + Next.js + Express + PostgreSQL
  ✓ Phase 2: Specification complete (OpenAPI)
  → Routing to Fullstack Builder

Agent (Fullstack Builder):
  ✓ Phase 3: Project scaffolded
  → Parallel execution:
    - Database Builder: Phases 4 (Database)
    - Backend Builder: Phases 5-6 (API, Business Logic)
    - Frontend Builder: Phase 7 (Frontend)
    - Test Engineer: Phase 9 (Tests)
    - DevOps Engineer: Phase 8 (Infrastructure)

Agent (Security Engineer):
  ✓ Phase 10: Security audit complete

Agent (Code Reviewer):
  ✓ Phase 11: Quality gates passed

Agent (Documentation Writer):
  ✓ Phase 12: README, API docs, architecture docs

Agent (Fullstack Builder):
  ✓ Phase 13: Ready to ship!

User receives:
├── Working repository
├── All source files
├── Tests with 82% coverage
├── Docker configuration
├── CI/CD pipeline
├── Complete documentation
└── Deployment instructions
```

---

## Critical Rules Applied

1. **ZERO TODOs**: Every file is 100% complete
2. **ZERO BUGS**: Every code path tested
3. **PRODUCTION GRADE**: Deploy as-is
4. **FULL STACK**: Database through deployment
5. **SECURITY BY DEFAULT**: All OWASP checks
6. **TYPE SAFE**: TypeScript, no `any`
7. **TESTED**: 80%+ coverage
8. **DOCUMENTED**: Every public API documented
9. **DEPLOYABLE**: Docker + CI/CD ready
10. **MAINTAINABLE**: Clean architecture

---

## Typical Project Output

```
blog-platform/
├── .github/workflows/
│   ├── ci.yml
│   └── deploy.yml
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   └── types/
│   ├── tests/
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── models/
│   │   ├── middleware/
│   │   └── types/
│   ├── tests/
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   ├── package.json
│   └── tsconfig.json
├── docker-compose.yml
├── Dockerfile
├── .env.example
├── README.md
├── ARCHITECTURE.md
├── API.md
└── CONTRIBUTING.md

Lines of Code: 8,000-15,000 (depending on features)
Time to Deployment-Ready: 10-16 hours
Code Coverage: 82-95%
Test Count: 400-600 tests
```
