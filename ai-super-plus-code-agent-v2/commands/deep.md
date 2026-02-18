---
name: /deep
description: >
  Force maximum depth. All 14 phases, all agents engaged, full security audit, comprehensive
  testing, complete documentation. Use for critical projects, security-sensitive work,
  major features, and high-stakes implementations requiring absolute quality.
---

# Deep Command

## Purpose
Execute complete 14-phase pipeline with all agents engaged for maximum quality on critical work.

## When to Use
- Security-critical features
- Payment/financial systems
- Authentication systems
- Data protection features
- Major architectural changes
- New service creation
- High-visibility features
- Compliance-required work
- Zero-trust systems
- Patient/PII handling
- High-concurrency systems
- Mission-critical features

## When NOT to Use
- Simple bug fixes (use /quick)
- Documentation updates (use /document)
- Small improvements (use /quick)
- Time-sensitive hotfixes (use /quick)
- Low-risk changes (use /quick)

## Complete 14-Phase Pipeline

### Phase 1: Requirements & Architect (Architect Agent)
- **Stakeholder Interview**: Gather requirements
- **User Stories**: Document user scenarios
- **Acceptance Criteria**: Define success metrics
- **Constraints**: Identify technical/business constraints
- **Architecture Design**: High-level system design
- **Technology Selection**: Choose tech stack
- **Dependency Mapping**: Identify all dependencies
- **C4 Diagrams**: Create architecture diagrams
- **Risk Assessment**: Identify architectural risks
- **Review**: Comprehensive architecture review

### Phase 2: API Design (API Designer Agent)
- **REST Design**: RESTful API design
- **GraphQL/gRPC**: Alternative API option design
- **OpenAPI Spec**: Complete OpenAPI specification
- **Request/Response**: Full schema definitions
- **Error Handling**: Error code and response design
- **Rate Limiting**: Rate limit strategy
- **Versioning**: API versioning strategy
- **Authentication**: Auth integration design
- **Examples**: Comprehensive examples
- **Documentation**: API documentation

### Phase 3: Database Design (Database Builder Agent)
- **Schema Design**: Complete schema design
- **Normalization**: 3NF normalization verification
- **Relationships**: All relationships defined
- **Constraints**: All constraints defined
- **Indexes**: Comprehensive indexing strategy
- **Migrations**: Forward and backward migrations
- **ORM Models**: Generate ORM models
- **Seed Data**: Test data factories
- **Backup Strategy**: Backup and recovery plan
- **Performance**: Query optimization strategy

### Phase 4: Frontend Architecture (Frontend Builder Agent)
- **Component Tree**: Complete component hierarchy
- **State Management**: State architecture design
- **Routing**: Complete routing setup
- **Forms**: Form architecture design
- **Styling**: Design system implementation
- **Accessibility**: WCAG 2.1 AA compliance
- **Responsive**: Mobile/tablet/desktop design
- **Performance**: Frontend performance targets
- **Security**: Frontend security implementation
- **Internationalization**: i18n setup

### Phase 5: Backend Architecture (Backend Builder Agent)
- **Service Layer**: Service architecture design
- **Repository Layer**: Data access pattern design
- **Middleware**: Complete middleware stack
- **Validation**: Input validation strategy
- **Error Handling**: Comprehensive error handling
- **Logging**: Structured logging design
- **Caching**: Caching architecture
- **Job Queue**: Background job design
- **Rate Limiting**: Rate limiting implementation
- **API Contracts**: API contract verification

### Phase 6: Implementation (Implement Command)
- **Database**: Migrations and models
- **Backend**: Complete backend implementation
- **Frontend**: Complete frontend implementation
- **Integration**: Wire all components together
- **Configuration**: Environment configuration
- **Features**: All features implemented
- **Edge Cases**: Edge case handling
- **Error Scenarios**: Error path handling
- **Documentation**: Code comments and JSDoc
- **Type Safety**: Full TypeScript strict mode

### Phase 7: Testing (Test Engineer Agent)
- **Unit Tests**: 90%+ coverage on functions
- **Integration Tests**: Component integration tests
- **E2E Tests**: Critical user path tests
- **Performance Tests**: Load and stress testing
- **Security Tests**: OWASP vulnerability tests
- **Accessibility Tests**: WCAG compliance tests
- **Cross-Browser**: Chrome, Firefox, Safari, Edge
- **Mobile**: iOS and Android testing
- **Regression**: All regression tests pass
- **Fixtures**: Proper test data management

### Phase 8: Code Review (Code Reviewer Agent)
- **Architecture**: Architecture adherence review
- **Security**: Security vulnerabilities review
- **Performance**: Performance anti-patterns review
- **Quality**: Code quality standards review
- **Testing**: Test coverage adequacy review
- **Documentation**: Documentation completeness review
- **Standards**: SOLID principles review
- **Patterns**: Design patterns review
- **Type Safety**: TypeScript strict compliance review
- **Accessibility**: Accessibility compliance review

### Phase 9: Security Audit (Security Engineer Agent)
- **OWASP Top 10**: All 10 categories addressed
- **Vulnerability Scan**: Automated vulnerability scan
- **Secrets Detection**: No secrets in code
- **Dependency Audit**: All dependencies scanned
- **Authentication**: Secure authentication implementation
- **Authorization**: Proper authorization enforcement
- **Encryption**: Data at rest and transit encryption
- **Headers**: All security headers configured
- **Penetration**: Penetration testing simulation
- **Compliance**: Compliance requirements verification

### Phase 10: Performance Optimization (Performance Engineer Agent)
- **Profiling**: Complete performance profiling
- **Database**: Query optimization and indexing
- **Frontend**: Bundle size and rendering optimization
- **Caching**: Caching implementation
- **API**: Response time optimization
- **Benchmarking**: Before/after measurements
- **Core Web Vitals**: All metrics optimized
- **Memory**: Memory leak elimination
- **Monitoring**: Performance monitoring setup
- **Targets**: All targets met or exceeded

### Phase 11: DevOps & Deployment (DevOps Engineer Agent)
- **Dockerfile**: Production-ready Dockerfile
- **Kubernetes**: K8s manifest generation
- **CI/CD Pipeline**: Complete automation
- **Infrastructure**: IaC (Terraform/CloudFormation)
- **Monitoring**: Prometheus/Grafana setup
- **Logging**: ELK stack configuration
- **Alerting**: Alert rules and escalation
- **Backup**: Backup and recovery verification
- **Security**: Infrastructure security hardening
- **Runbooks**: Comprehensive runbooks

### Phase 12: Documentation (Documentation Writer Agent)
- **README**: Complete README documentation
- **API Docs**: OpenAPI documentation
- **Architecture Docs**: C4 diagrams and ADRs
- **Contributing Guide**: Development workflow guide
- **Code Comments**: Comprehensive JSDoc/docstrings
- **Changelog**: Complete change log
- **Runbooks**: Deployment and incident runbooks
- **Troubleshooting**: Comprehensive troubleshooting guide
- **FAQ**: Frequently asked questions
- **Examples**: Working code examples

### Phase 13: Final Review & QA
- **Code Review**: Final code review by tech lead
- **Security Review**: Final security sign-off
- **Performance Review**: Final performance verification
- **Documentation Review**: Documentation completeness
- **Testing**: All test suites pass
- **Type Checking**: Zero TypeScript errors
- **Linting**: Zero linting errors
- **Build**: Production build succeeds
- **Deployment**: Staging deployment succeeds
- **Monitoring**: All monitoring active

### Phase 14: Production Deployment & Monitoring
- **Pre-Deployment**: Final pre-deployment checklist
- **Deployment**: Production deployment
- **Verification**: Post-deployment verification
- **Monitoring**: Active monitoring of metrics
- **Alerts**: Alert triggers tested
- **Documentation**: Deployment documented
- **Communication**: Team communication
- **Rollback**: Rollback plan ready
- **Follow-up**: Post-deployment review
- **Lessons**: Lessons learned documented

## Quality Assurance Checkpoints

**Checkpoint 1: Requirements (End of Phase 1)**
- Architecture approved by tech lead
- Requirements clear and agreed
- Technical feasibility confirmed
- Risk assessment complete
- Success criteria defined

**Checkpoint 2: Design (End of Phase 2)**
- API design approved
- Database design approved
- Frontend architecture approved
- Backend architecture approved
- All designs reviewed and approved

**Checkpoint 3: Implementation (End of Phase 6)**
- All code written
- All tests written
- TypeScript strict compliance
- No linting errors
- Code formatted

**Checkpoint 4: Quality (End of Phase 8)**
- Code review completed
- All feedback addressed
- Architecture approved
- Standards met
- No critical issues

**Checkpoint 5: Security (End of Phase 9)**
- Security audit completed
- All vulnerabilities addressed
- Zero critical/high vulnerabilities
- Dependencies scanned
- Secrets removed

**Checkpoint 6: Performance (End of Phase 10)**
- Profiling completed
- Optimization implemented
- All targets met
- Benchmarks documented
- Monitoring setup

**Checkpoint 7: Infrastructure (End of Phase 11)**
- Docker images built
- K8s manifests ready
- CI/CD pipeline working
- IaC complete
- Monitoring active

**Checkpoint 8: Documentation (End of Phase 12)**
- All documentation complete
- Examples working
- Runbooks tested
- README clear
- API docs accurate

**Checkpoint 9: Final Review (End of Phase 13)**
- All issues resolved
- All tests passing
- Build succeeds
- Staging works
- Go/no-go decision

**Checkpoint 10: Production (End of Phase 14)**
- Deployment successful
- Monitoring active
- Alerts working
- Rollback ready
- Team trained

## Success Criteria

- Zero critical/high severity issues
- 95%+ test coverage
- All OWASP Top 10 addressed
- No known vulnerabilities
- 99.9% uptime in production
- <100ms API response time
- <2.5s LCP on frontend
- All documentation complete
- Team trained and confident
- Zero security incidents

## Output Expectations

Complete production-ready system with:
- Fully implemented features
- Comprehensive test suite (95%+ coverage)
- Complete documentation
- Production deployment infrastructure
- Security hardened and audited
- Performance optimized
- Monitoring and alerting active
- Team trained and ready
- Rollback procedures documented

## Success Indicators

- All 14 phases completed successfully
- All checkpoints approved
- Zero blockers or critical issues
- All tests passing
- All documentation complete
- Team confident in deployment
- Monitoring showing expected performance
- Zero security vulnerabilities
- Ready for production load
- Lessons learned documented
