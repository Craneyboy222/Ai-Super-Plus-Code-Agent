---
name: /document
description: >
  Generate comprehensive documentation. README, API docs (OpenAPI), architecture docs (C4),
  contributing guide, code comments, changelog. Ensures documentation is discoverable,
  complete, and always in sync with code.
---

# Document Command

## Purpose
Generate comprehensive documentation for project, API, architecture, and contribution guidelines.

## When to Use
- Creating new project documentation
- Documenting API endpoints
- Architecture changes requiring documentation
- Contributing guide for new contributors
- Release notes/changelog
- Onboarding new team members
- Pre-deployment documentation
- Compliance documentation

## Execution Steps

### 1. Analyze Codebase for Documentation
- **Scan Files**: Identify all source files
- **Extract JSDoc**: Pull JSDoc comments
- **Find Functions**: List all exported functions
- **Find Components**: List all components
- **Find Routes**: List all API routes
- **Database**: Identify database schema
- **Config**: Identify configuration
- **Tests**: Understand test structure

### 2. Generate README Documentation
- **Project Title**: Clear, descriptive title
- **Description**: What does project do (1-2 sentences)
- **Key Features**: 5-10 main features
- **Tech Stack**: Technologies and versions
- **Status**: Production/Beta/Alpha status
- **Live Link**: Link to deployed app if available
- **Getting Started**: Quick setup instructions
- **Installation**: Step-by-step setup
- **Configuration**: Environment variable setup
- **Running**: How to start dev server
- **Testing**: How to run tests
- **Building**: How to build for production
- **Troubleshooting**: Common issues/solutions
- **Contributing**: Contributing guidelines link
- **License**: License information
- **Authors**: Maintainers and credits

### 3. Generate API Documentation

**OpenAPI Specification**
- **API Overview**: Base URL, authentication
- **Endpoints**: All endpoints with methods, paths
- **Parameters**: Query, path, body parameters
- **Request Schemas**: Request body schemas
- **Response Schemas**: Response body schemas
- **Error Codes**: Possible error codes and meanings
- **Rate Limits**: Request rate limits
- **Examples**: CURL, JavaScript, Python examples
- **Authentication**: How to authenticate

**API Documentation Portal**
- **Getting Started**: 5-minute quick start
- **Authentication**: How to get API credentials
- **First Request**: Walkthrough of first API call
- **Error Handling**: How to handle errors
- **Pagination**: Pagination pattern explanation
- **Filtering**: How to filter results
- **Sorting**: How to sort results
- **Rate Limiting**: Rate limits and retry strategy
- **SDKs**: Available SDKs and links
- **Common Tasks**: Guides for common operations
- **Troubleshooting**: Common issues and solutions

### 4. Generate Architecture Documentation

**C4 Model Diagrams**
- **Context Diagram**: System and external systems
- **Container Diagram**: High-level application structure
- **Component Diagram**: Components within containers
- **Code Diagram**: Class diagrams for critical code

**Architecture Decision Records (ADRs)**
- **Template**: Standard ADR template
- **Status**: Proposed/Accepted/Deprecated
- **Context**: Why decision was needed
- **Decision**: What was decided
- **Consequences**: Positive and negative outcomes
- **Alternatives**: Other options considered
- **Examples**: Decision examples and rationale

**Design Documentation**
- **System Design**: High-level system design
- **Database Design**: Schema and relationships
- **Data Flow**: How data flows through system
- **Integration**: External service integrations
- **Security**: Security architecture and approach
- **Scalability**: Horizontal/vertical scaling approach
- **Performance**: Performance considerations
- **Resilience**: Failure handling and recovery

### 5. Generate Contributing Guide
- **Code Style**: Linting and formatting rules
- **Git Workflow**: Branch naming and PR process
- **Commit Messages**: Format and guidelines
- **Testing**: Testing expectations
- **Documentation**: Documentation requirements
- **Code Review**: Code review process
- **Development Setup**: Detailed development environment setup
- **Running Tests**: How to run tests locally
- **Building**: How to build locally
- **Common Tasks**: Common development tasks
- **Reporting Issues**: How to report bugs
- **Suggesting Features**: How to suggest features
- **Code of Conduct**: Community guidelines

### 6. Generate Code Comments & JSDoc

**JSDoc Generation**
- **Functions**: Purpose, params, returns, example
- **Classes**: Class purpose, properties, methods
- **Exports**: Document what is exported
- **Types**: TypeScript type documentation
- **Constants**: Constant documentation
- **Examples**: Usage examples for complex APIs
- **Deprecation**: Mark deprecated code
- **Links**: Cross-references between docs

**Inline Comments**
- **Complex Logic**: Explain non-obvious algorithms
- **Business Logic**: Explain business rules
- **Workarounds**: Document why workarounds exist
- **Gotchas**: Highlight common mistakes
- **Performance**: Document performance-critical code
- **Security**: Document security-sensitive code

### 7. Generate Changelog

**Format**
- **Versions**: Semantic versioning (MAJOR.MINOR.PATCH)
- **Release Date**: Date of each release
- **Categories**: Added, Changed, Deprecated, Removed, Fixed, Security
- **Items**: Clear, user-facing descriptions
- **Links**: Links to PRs, issues, commits
- **Contributors**: Acknowledge contributors

**Content**
- **New Features**: User-facing features
- **Breaking Changes**: Changes requiring code updates
- **Deprecations**: Features being deprecated
- **Bug Fixes**: Bugs fixed
- **Performance**: Performance improvements
- **Security**: Security fixes and updates

### 8. Generate Contributing Guide

**Development Setup**
- **Prerequisites**: Required software versions
- **Installation**: Step-by-step setup
- **Configuration**: Local configuration
- **Running**: How to start development server
- **Testing**: How to run tests
- **Verification**: How to verify setup works

**Workflow**
- **Branch Naming**: convention (feature/x, fix/x)
- **Commit Messages**: Format (type: description)
- **Pull Requests**: PR template and process
- **Code Review**: How PRs are reviewed
- **Approval**: Who can approve PRs
- **Merging**: When and how to merge
- **CI/CD**: What checks run automatically

**Standards**
- **Code Style**: Project code style rules
- **Testing**: Testing coverage expectations
- **Documentation**: Documentation expectations
- **Performance**: Performance benchmarks
- **Security**: Security review checklist
- **Accessibility**: Accessibility requirements

### 9. Generate Deployment Documentation

**Deployment Guide**
- **Prerequisites**: Pre-deployment checklist
- **Steps**: Step-by-step deployment process
- **Verification**: Post-deployment verification
- **Rollback**: Rollback procedures
- **Monitoring**: What to monitor
- **Communication**: Who to notify

**Runbooks**
- **Incident Response**: Incident response procedures
- **Troubleshooting**: Common issues and fixes
- **Scaling**: How to scale application
- **Backups**: Backup and recovery procedures
- **Maintenance**: Regular maintenance tasks
- **Updates**: How to update dependencies

### 10. Generate Support & FAQ

**Getting Help**
- **Documentation**: Link to documentation
- **Issues**: Link to issue tracker
- **Discussions**: Link to discussions
- **Email**: Support email
- **Slack**: Community Slack workspace
- **Response Time**: Expected response time

**FAQ**
- **Common Issues**: Frequently asked questions
- **Troubleshooting**: Troubleshooting steps
- **Best Practices**: Best practices guide
- **Examples**: Working code examples
- **Performance**: Performance tips
- **Security**: Security best practices

### 11. Documentation Review
- **Accuracy**: Documentation matches actual code
- **Completeness**: All major aspects documented
- **Clarity**: Clear and understandable
- **Examples**: Sufficient working examples
- **Links**: Proper linking between docs
- **Searchability**: Easy to find information
- **Format**: Consistent formatting
- **Maintenance**: Updatable and maintainable

### 12. Publish Documentation
- **Generate**: Generate final documentation
- **Structure**: Organize documentation logically
- **Navigation**: Easy navigation between docs
- **Search**: Documentation searchability
- **Hosting**: Choose documentation hosting
- **Deploy**: Deploy documentation
- **Promote**: Share documentation with team
- **Maintain**: Keep documentation updated

## Quality Criteria

- README provides complete project overview
- API documentation 100% accurate
- Architecture documentation reflects reality
- Contributing guide enables new PRs
- Code has comprehensive JSDoc
- Changelog updated with each release
- Examples all work and are current
- All links are valid
- Documentation is searchable
- New contributors can onboard in <1 day

## Output Expectations

```
/docs
  README.md (main documentation)
  CONTRIBUTING.md (contribution guidelines)
  CHANGELOG.md (version history)
  /api
    openapi.yaml (API specification)
    getting-started.md
    authentication.md
  /architecture
    README.md
    C4_context.md
    C4_container.md
    /adr
      001-technology-selection.md
  /guides
    development-setup.md
    code-style.md
    testing.md
  /runbooks
    deployment.md
    incident-response.md
    troubleshooting.md
  FAQ.md
  SUPPORT.md
```

## Success Indicators

- README clear enough for new user to understand
- API docs match actual endpoints
- Architecture docs reflect current design
- Contributing guide enables rapid contributions
- All public functions have JSDoc
- Changelog updated with every release
- Examples all work and are practical
- New developer can set up in <30 minutes
- Minimal documentation questions from users
- Search finds answers in <30 seconds
