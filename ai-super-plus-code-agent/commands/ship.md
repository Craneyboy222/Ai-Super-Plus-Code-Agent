# Command: ship

**Description**: Full pipeline execution (build → test → secure → deploy → docs)
**Complexity**: Highest - Executes all 14 phases
**Time**: 10-16 hours for full APP

---

## What This Command Does

Executes the complete 14-phase pipeline end-to-end:

```
/ship
  ├─ Phase 0: PROJECT INTAKE ✓
  ├─ Phase 1: ARCHITECTURE DESIGN ✓
  ├─ Phase 2: SPECIFICATION ✓
  ├─ Phase 3: SCAFFOLD ✓
  ├─ Phase 4: DATABASE LAYER ✓
  ├─ Phase 5: API LAYER ✓
  ├─ Phase 6: BUSINESS LOGIC ✓
  ├─ Phase 7: FRONTEND ✓
  ├─ Phase 8: INFRASTRUCTURE ✓
  ├─ Phase 9: TESTS ✓
  ├─ Phase 10: SECURITY AUDIT ✓
  ├─ Phase 11: QUALITY GATE ✓
  ├─ Phase 12: DOCUMENTATION ✓
  └─ Phase 13: SHIP ✓

Result: Production-ready system ready for deployment
```

---

## Usage

```
User: Ship a SaaS project with the following requirements:
- Multi-tenant blog platform
- User authentication with OAuth
- Rich text editor for posts
- Real-time comments
- Search functionality
- Admin dashboard

Agent (Architect):
  Analyzing requirements...
  Complexity: SYSTEM
  Estimated time: 16 hours
  Estimated cost: ~200k tokens

Agent (Fullstack Builder):
  Beginning full pipeline...

[Progress bar: ████████░░ 40%]
Phase 0-3: Complete
Phase 4-7: In progress...
Phase 8: Infrastructure generation
Phase 9: Test suite generation
Phase 10: Security audit
Phase 11: Quality gates
Phase 12: Documentation
Phase 13: Shipping...

Final Output:
✓ 150+ files generated
✓ 600+ tests (86% coverage)
✓ Security audit: PASSED
✓ Performance: OPTIMIZED
✓ Documentation: COMPLETE
✓ Ready to deploy in: AWS / GCP / Azure

Deployment instructions: See DEPLOYMENT.md
Get started: npm install && npm run dev
```

---

## Pre-Ship Requirements

Before running `/ship`, ensure you have:

```
Requirements Checklist:
☐ Clear project requirements documented
☐ Tech stack preferences specified (or use defaults)
☐ Target deployment platform identified
☐ Performance requirements defined
☐ Scalability expectations communicated
☐ Security compliance needs identified
☐ Budget and timeline understood
☐ Team size and expertise communicated
```

---

## What Gets Delivered

### 1. Complete Codebase
```
├── Backend (API + Business Logic)
│   ├── Routes and handlers
│   ├── Database models and migrations
│   ├── Service layer
│   ├── Error handling
│   ├── Validation
│   ├── Authentication
│   ├── Authorization
│   └── 200+ test cases
├── Frontend (React/Next.js)
│   ├── Pages and components
│   ├── State management
│   ├── Forms and validation
│   ├── API client integration
│   ├── Responsive design
│   ├── Accessibility
│   └── 300+ test cases
├── Database
│   ├── Schema with migrations
│   ├── Indexes and constraints
│   ├── Seed data
│   └── Backup strategy
└── Infrastructure
    ├── Docker configuration
    ├── CI/CD pipeline
    ├── Kubernetes manifests
    ├── Environment configs
    ├── Monitoring setup
    └── Deployment scripts
```

### 2. Comprehensive Test Suite
```
├── Unit Tests (300+)
│   ├── Service logic
│   ├── Component behavior
│   ├── Utility functions
│   └── Edge cases
├── Integration Tests (150+)
│   ├── API endpoints
│   ├── Database operations
│   ├── Service interactions
│   └── Middleware stack
├── E2E Tests (50+)
│   ├── User authentication
│   ├── Core workflows
│   ├── Complex scenarios
│   └── Error recovery
├── Security Tests (100+)
│   ├── XSS prevention
│   ├── SQL injection prevention
│   ├── CSRF protection
│   ├── Authorization checks
│   ├── Rate limiting
│   └── Input validation
└── Performance Tests (20+)
    ├── Load testing
    ├── Stress testing
    ├── API response times
    └── Database query optimization
```

**Coverage**: 80-90% code coverage
**Execution Time**: < 30 seconds
**All Passing**: ✓

### 3. Security Audit Report
```
OWASP Top 10 Compliance:
☑ A01: Access Control - PASSED
☑ A02: Cryptographic Failures - PASSED
☑ A03: Injection - PASSED
☑ A04: Insecure Design - PASSED
☑ A05: Security Misconfiguration - PASSED
☑ A06: Vulnerable & Outdated Components - PASSED
☑ A07: Authentication Failures - PASSED
☑ A08: Data Integrity Failures - PASSED
☑ A09: Logging & Monitoring Failures - PASSED
☑ A10: SSRF - PASSED

Security Issues Found: 0
Critical Vulnerabilities: 0
High Vulnerabilities: 0
Recommendations: 5
```

### 4. Complete Documentation
```
├── README.md
│   ├── Project description
│   ├── Tech stack
│   ├── Setup instructions
│   ├── Development guide
│   └── Contributing guide
├── ARCHITECTURE.md
│   ├── System design
│   ├── Component architecture
│   ├── Data flow diagrams
│   ├── Technology decisions (ADRs)
│   └── Scalability plan
├── API.md
│   ├── Auto-generated API docs
│   ├── Example requests/responses
│   ├── Authentication guide
│   ├── Rate limits
│   └── Error codes
├── DEPLOYMENT.md
│   ├── Prerequisites
│   ├── Step-by-step deployment
│   ├── Environment setup
│   ├── Database setup
│   └── Rollback procedures
├── CONTRIBUTING.md
│   ├── Code style guide
│   ├── Pull request process
│   ├── Testing requirements
│   ├── Commit message format
│   └── Code review process
└── TROUBLESHOOTING.md
    ├── Common issues
    ├── Solutions
    ├── Debugging tips
    └── Support channels
```

### 5. Infrastructure & Deployment
```
├── Dockerfile (production-optimized)
├── docker-compose.yml (development)
├── .github/workflows/
│   ├── ci.yml (lint, test, build)
│   └── deploy.yml (automated deployment)
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── secrets.yaml
├── terraform/ (IaC)
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── .env.example (all required variables)
```

### 6. Quality Metrics
```
Build Quality:
├─ TypeScript Errors: 0
├─ ESLint Warnings: 0
├─ Build Time: < 5 minutes
├─ Bundle Size: < 1MB
└─ Docker Image: < 200MB

Code Quality:
├─ Code Coverage: 85%
├─ Cyclomatic Complexity: < 5 (avg)
├─ Maintainability Index: > 70
├─ Technical Debt: Minimal
└─ Code Review Score: A

Performance:
├─ API Response Time (p95): < 200ms
├─ Frontend TTI: < 2s
├─ Database Query Time (avg): < 50ms
├─ Memory Usage: Stable
└─ CPU Usage: Baseline normal

Security:
├─ Vulnerability Scan: CLEAN
├─ Dependency Audit: CLEAN
├─ Secrets Detected: 0
├─ Security Headers: ✓
└─ OWASP Compliance: 100%
```

---

## Post-Ship Checklist

### Immediate (Day 1)
```
☐ Verify all systems running
☐ Check health endpoints
☐ Monitor error logs
☐ Review user feedback
☐ Verify backups working
☐ Confirm monitoring active
☐ Team training complete
```

### First Week
```
☐ Load test with realistic traffic
☐ Monitor for anomalies
☐ Performance analysis
☐ Security baseline established
☐ Cost tracking started
☐ Team familiar with system
☐ Documentation corrections
```

### First Month
```
☐ Full scalability test
☐ Disaster recovery test
☐ Dependency security updates
☐ Code optimization review
☐ Team retrospective
☐ Roadmap for features
```

---

## Rollback Plan

If critical issues found:

```
IMMEDIATE ACTION:
├─ 1. Click rollback button (< 1 minute)
├─ 2. Verify previous version running
├─ 3. Run smoke tests
├─ 4. Alert all stakeholders
└─ 5. Investigation begins

Investigation (First 4 hours):
├─ Root cause analysis
├─ Identify fix
├─ Prepare hotfix
├─ Code review hotfix
└─ Deploy hotfix

Post-Issue:
├─ Post-mortem (24 hours)
├─ Process improvement
├─ Documentation update
└─ Team debrief
```

---

## Support After Ship

### Monitoring Dashboard
```
Real-time metrics:
├─ Uptime: 99.9%
├─ Error Rate: < 0.1%
├─ Response Time: 150ms (avg)
├─ Active Users: 2,341
├─ Database Size: 42GB
├─ Disk Usage: 45%
├─ CPU Usage: 35%
├─ Memory Usage: 62%
└─ Network: Stable
```

### Alerting
```
Critical Alerts:
├─ Service Down → SMS + Email
├─ Error Rate > 1% → Slack
├─ Response Time > 1s → Slack
├─ Disk Usage > 90% → Email
├─ Database Connection Pool Full → SMS
└─ Security Event → SMS + Email
```

### Support Channels
```
Priority 1 (Down): Phone
Priority 2 (Degraded): Slack + Email
Priority 3 (Minor): Email + Ticket
Priority 4 (Feature): Ticket
```

---

## Success Looks Like

```
After 1 week:
✓ System stable and reliable
✓ Users successfully using platform
✓ No critical bugs found
✓ Performance metrics normal
✓ Team confident in codebase
✓ Documentation accurate

After 1 month:
✓ 99.9% uptime maintained
✓ Handling peak traffic smoothly
✓ Zero critical security issues
✓ Team can make code changes
✓ Scaling plan validated
✓ Cost within budget

After 1 quarter:
✓ Hundreds of active users
✓ Seamless operations
✓ Technical debt minimal
✓ Team productivity high
✓ Business metrics met
✓ Ready for next phase of growth
```

---

**Command Status**: READY TO EXECUTE
**Estimated Completion**: 10-16 hours
**Output Quality**: PRODUCTION GRADE
