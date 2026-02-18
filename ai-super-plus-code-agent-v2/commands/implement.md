---
name: /implement
description: >
  Generate complete implementation code for a feature or module. Takes feature description,
  generates all files in dependency order with types, tests, documentation, and integration
  points. Produces fully working, tested, documented feature.
---

# Implement Command

## Purpose
Generate complete, production-ready implementation for a feature or module with all necessary files, tests, and documentation.

## When to Use
- Implementing a new feature with full stack
- Adding a module/service to existing project
- Generating CRUD operations for a resource
- Building a component with all dependencies

## Execution Steps

### 1. Requirement Analysis
- **Parse feature description**: Extract requirements from text
- **Identify scope**: Frontend, backend, database, API changes
- **List dependencies**: What existing code is needed
- **Resource identification**: What entities are involved
- **Operations**: What CRUD operations are needed
- **Relationships**: Entity relationships and foreign keys
- **Business logic**: Validation, calculations, workflows

### 2. Architecture Planning
- **API Design**: Endpoints, request/response schemas
- **Database Schema**: Tables, columns, relationships, indexes
- **Component Structure**: Frontend components needed
- **Service Layer**: Business logic organization
- **Data Flow**: How data moves through layers
- **Error Handling**: Expected errors and responses
- **Generate Plan**: Show planned files and structure

### 3. Generate Files in Dependency Order
- **Database First**: Migrations, schemas, models
- **API Layer**: OpenAPI specs, request/response types
- **Service Layer**: Business logic, validation
- **Repository Layer**: Database access, queries
- **Controller/Handler**: Route handlers, request processing
- **Frontend Components**: React components, forms, pages
- **Tests**: Unit, integration, E2E tests
- **Documentation**: JSDoc, README, API docs
- **Integration**: Wiring everything together

### 4. Database Layer Generation
- **Migration File**: SQL for creating/updating schema
- **ORM Model**: TypeORM/Sequelize/Mongoose model
- **Indexes**: Performance indexes on query columns
- **Seed Data**: Factory for test data
- **Relationships**: Foreign key setup, cascades
- **Validation**: Constraints and check constraints

### 5. API Layer Generation
- **OpenAPI Schema**: Request/response types
- **Types**: TypeScript interfaces for models
- **Request Validation**: Schema validation
- **Error Types**: Custom error classes
- **Constants**: Status codes, error messages
- **Enums**: Restricted value types

### 6. Service Layer Generation
- **Service Class**: Business logic implementation
- **Repository Injection**: Dependency injection setup
- **Methods**: CRUD methods and custom operations
- **Validation**: Input validation before DB access
- **Error Handling**: Try-catch with proper error responses
- **Logging**: Structured logging of operations
- **Transactions**: Multi-step operations in transactions

### 7. Repository Layer Generation
- **Repository Class**: Database access abstraction
- **Query Methods**: find, findOne, create, update, delete
- **Advanced Queries**: Filtering, sorting, pagination
- **Batch Operations**: Bulk insert/update/delete
- **Optimization**: Eager loading, select specificity
- **Caching**: Cache layer for expensive queries

### 8. Controller/Handler Generation
- **Route Handlers**: GET, POST, PUT, DELETE handlers
- **Request Processing**: Extract params, validate, call service
- **Response Formatting**: Format service responses
- **Error Handling**: Catch and format errors
- **Status Codes**: Appropriate HTTP status codes
- **Logging**: Request/response logging

### 9. Frontend Components Generation
- **Page Component**: Main feature page
- **Form Component**: Input form with validation
- **List Component**: Display list with pagination
- **Detail Component**: Display single item details
- **Hooks**: Custom hooks for API calls
- **State Management**: Store/context setup
- **Styling**: Responsive CSS/Tailwind classes

### 10. Test Generation
- **Unit Tests**: Service logic tests
- **Repository Tests**: Database query tests
- **Controller Tests**: Route handler tests
- **Component Tests**: React component tests
- **Integration Tests**: Feature end-to-end tests
- **Fixtures**: Mock data factories
- **Coverage**: Aim for 80%+ coverage

### 11. Documentation
- **JSDoc**: Document all public functions
- **API Documentation**: Update OpenAPI spec
- **README**: Usage examples
- **Type Documentation**: Document complex types
- **Comments**: Explain non-obvious logic
- **Examples**: Working code examples

### 12. Integration & Verification
- **Wire Routing**: Add routes to router configuration
- **Inject Dependencies**: Wire services and repositories
- **Migrate Database**: Run migrations
- **Compile TypeScript**: Verify no TS errors
- **Run Tests**: All tests pass
- **Type Check**: Full TypeScript strict check
- **Lint Check**: ESLint with no errors
- **Build**: Production build succeeds

## Quality Criteria

- All code follows project conventions
- 100% TypeScript strict mode compliance
- 80%+ test coverage minimum
- No ESLint or Prettier errors
- No circular dependencies
- All endpoints have tests
- All service methods have tests
- Request validation catches invalid input
- Error handling covers all failure paths
- Database schema is normalized
- API follows REST conventions
- Components are properly memoized
- No N+1 query problems
- Code is well-documented with JSDoc

## Output Expectations

- Complete feature file structure
- All required files generated
- Database migrations ready to run
- API endpoints fully implemented
- Frontend components rendering correctly
- Tests passing with good coverage
- No TypeScript errors
- No linting errors
- Type-safe throughout
- Proper error handling
- Clean integration with existing code
- Comprehensive documentation
- Ready to merge and deploy

## Success Indicators

- `npm test` shows all tests passing
- `npm run build` completes without errors
- No TypeScript errors with `npx tsc --noEmit`
- No ESLint errors with `npm run lint`
- Feature works end-to-end in development
- API endpoints respond with correct data
- Forms validate and submit successfully
- Error cases handled gracefully
- Pagination and filtering work correctly
- All code is documented
- Feature integrates with existing system
