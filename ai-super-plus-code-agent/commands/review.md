---
name: /review
description: >
  Full code review. Checks architecture adherence, security, performance, naming, documentation,
  test coverage. Produces detailed report with findings, recommendations, and prioritized
  improvements.
---

# Review Command

## Purpose
Perform comprehensive code review across architecture, security, performance, quality, and best practices.

## When to Use
- Code review before merging PRs
- Quality assessment of new features
- Security audit of production code
- Performance review of slow components
- Technical debt assessment
- Onboarding new developers to codebase
- Pre-deployment quality verification

## Execution Steps

### 1. Code Discovery & Analysis
- **Scan Repository**: Identify all source files
- **Parse Code**: Extract AST and structure
- **Identify Modules**: Module and component boundaries
- **Map Dependencies**: Import and dependency analysis
- **Calculate Metrics**: Complexity, duplication, coverage
- **Identify Patterns**: Design patterns and anti-patterns

### 2. Architecture Review
- **Layering Check**: Verify separation of concerns
- **Dependency Direction**: Check dependencies flow correctly
- **Circular Dependencies**: Detect circular imports
- **Module Coupling**: Assess inter-module coupling
- **Cohesion**: Check related code is grouped
- **Abstraction Levels**: Verify consistent abstraction
- **Code Organization**: Logical structure assessment

### 3. Security Audit
- **Input Validation**: Server-side validation present
- **SQL Injection**: Parameterized queries used
- **XSS Prevention**: Output encoding applied
- **Authentication**: Proper auth on sensitive endpoints
- **Authorization**: RBAC properly enforced
- **Secrets Detection**: No hardcoded secrets
- **Dependency Vulnerabilities**: npm audit check
- **Encryption**: Sensitive data encrypted
- **OWASP Coverage**: All Top 10 addressed
- **CSP Headers**: Content security policy configured

### 4. Performance Review
- **N+1 Queries**: Database query optimization
- **Memory Leaks**: Potential memory leak patterns
- **Inefficient Loops**: Algorithm complexity review
- **Bundle Size**: JavaScript bundle size analysis
- **Render Performance**: React/Vue rendering optimization
- **Caching**: Cache usage assessment
- **Database Indexes**: Index presence and usage
- **API Response Times**: Endpoint performance
- **Database Connections**: Connection pooling
- **Network**: Unnecessary requests, compression

### 5. Code Quality Check
- **Naming Conventions**: Consistent naming
- **Code Structure**: Proper formatting and structure
- **Method Length**: Functions should be <50 lines
- **Cyclomatic Complexity**: Complexity <10 per function
- **Parameter Count**: <4 parameters per function
- **Nesting Depth**: <3 levels maximum
- **DRY Principle**: No repeated code blocks
- **SOLID Principles**: Single Responsibility, etc
- **Documentation**: Comments and JSDoc present
- **Comments Quality**: Meaningful comments only

### 6. Testing Coverage Review
- **Unit Test Coverage**: 80%+ minimum
- **Integration Tests**: Key flows tested
- **E2E Tests**: Critical user paths tested
- **Edge Cases**: Boundary conditions covered
- **Error Paths**: Exception handling tested
- **Mock Quality**: Realistic mocks
- **Test Isolation**: Proper setup/teardown
- **Flaky Tests**: No non-deterministic tests
- **Test Speed**: Tests run efficiently
- **Coverage Gaps**: Untested code identified

### 7. Type Safety Check
- **TypeScript Strict**: No 'any' types
- **Type Completeness**: All values typed
- **Generic Usage**: Proper generic type usage
- **Union Types**: Discriminated unions used
- **Type Guards**: Proper type narrowing
- **Return Types**: Explicit return types
- **Parameter Types**: All parameters typed
- **No Assertions**: No 'as' assertions unnecessarily

### 8. Best Practices Review
- **Framework Usage**: Correct framework patterns
- **Design Patterns**: Appropriate patterns used
- **Configuration**: Externalized configuration
- **Error Handling**: Comprehensive error handling
- **Logging**: Structured logging present
- **Monitoring**: Metrics and alerts setup
- **Documentation**: README and docs complete
- **Version Control**: Good commit messages
- **Dependencies**: Updated and minimal

### 9. Generate Findings Report
- **Critical Issues**: Must fix (security, data loss)
- **Major Issues**: Should fix (architecture, bugs)
- **Minor Issues**: Nice to fix (style, docs)
- **Code Quality Score**: 0-100 score
- **Severity Distribution**: Pie chart of issues
- **Component Scores**: Individual component scores
- **Trend**: Improvement or regression over time

### 10. Create Recommendations
- **Refactoring Opportunities**: Code to refactor
- **Modernization**: Outdated patterns to update
- **Performance Improvements**: Specific optimizations
- **Security Hardening**: Vulnerability fixes
- **Testing Gaps**: Missing tests to add
- **Documentation**: Documentation to improve
- **Technical Debt**: Debt to address
- **Next Steps**: Prioritized action items

## Quality Criteria

- All security vulnerabilities identified
- Architecture issues found and documented
- Performance bottlenecks highlighted
- Code quality issues comprehensive
- Testing gaps clearly identified
- Recommendations actionable and specific
- Score reflects actual code quality
- Issues properly prioritized
- No false positives
- Review can be completed in reasonable time

## Output Expectations

```
CODE_REVIEW_REPORT.md
├── Executive Summary (1 paragraph)
├── Overall Score (0-100)
├── Review Date & Duration
├── Files Reviewed (count and list)
├── Critical Issues (security, data loss)
│   └── Details with location
├── Major Issues (architecture, bugs)
│   └── Details with location
├── Minor Issues (style, docs)
│   └── Details with location
├── Scoring Breakdown
│   ├── Architecture (0-100)
│   ├── Security (0-100)
│   ├── Performance (0-100)
│   ├── Code Quality (0-100)
│   ├── Testing (0-100)
│   └── Documentation (0-100)
├── Metrics Summary
│   ├── Cyclomatic Complexity
│   ├── Test Coverage
│   ├── Code Duplication
│   └── Lines of Code
├── Architecture Diagram
├── Dependency Graph
├── Recommendations (by priority)
├── Refactoring Opportunities
├── Performance Optimization Suggestions
├── Security Hardening Plan
├── Testing Gaps
├── Technical Debt Assessment
├── Components Reviewed
│   ├── Component Name (score)
│   └── Top Issues
└── Next Steps
```

## Success Indicators

- Review identifies 90%+ of actual issues
- All security vulnerabilities found
- Critical performance bottlenecks identified
- Architecture problems documented
- No critical issues missed
- Recommendations are specific and actionable
- Score accurately reflects code quality
- Report is comprehensive but concise
- False positive rate <5%
- Review findings align with best practices
