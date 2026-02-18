---
name: /test
description: >
  Run and generate tests. Can generate missing tests, run existing test suite, report
  coverage, identify untested code paths, and ensure test quality with proper isolation
  and fixtures.
---

# Test Command

## Purpose
Generate comprehensive test suites and ensure all code has adequate test coverage with quality tests.

## When to Use
- Running tests to verify code quality
- Generating tests for untested code
- Checking test coverage metrics
- Fixing test failures
- Ensuring feature implementation has tests
- Pre-deployment test verification

## Execution Steps

### 1. Analyze Test Coverage
- **Run existing tests**: Execute test suite with coverage
- **Parse coverage report**: Extract line/branch coverage per file
- **Identify gaps**: Files with <80% coverage
- **List uncovered paths**: Specific code paths missing tests
- **Analyze priorities**: Critical path coverage vs edge cases
- **Generate coverage report**: HTML/JSON coverage visualization

### 2. Test Gap Identification
- **Untested Functions**: Functions with zero test calls
- **Untested Branches**: if/else paths with no tests
- **Untested Error Paths**: Error handling not tested
- **Missing Edge Cases**: Boundary conditions not tested
- **Mock Dependencies**: Hard-to-test dependencies identified
- **Integration Gaps**: Component interactions not tested
- **E2E Gaps**: User workflow paths not tested

### 3. Test Generation Strategy
- **Priority Functions**: Test most-used and critical functions first
- **Unit Tests**: Individual function behavior
- **Integration Tests**: Component interaction
- **E2E Tests**: User workflows
- **Edge Cases**: Boundary and special conditions
- **Error Paths**: Exception and error handling
- **Performance Tests**: Timing-sensitive code

### 4. Generate Unit Tests
- **Function Setup**: Import and describe block
- **Happy Path**: Normal input producing expected output
- **Edge Cases**: Boundary conditions (empty, null, max values)
- **Error Conditions**: Invalid input handling
- **Type Checking**: Type mismatches caught
- **Side Effects**: Function side effects verified
- **Mocking**: Dependencies properly mocked
- **Assertions**: Clear, specific assertions

### 5. Generate Integration Tests
- **Component Interaction**: Multiple units working together
- **Service Integration**: Service calling repository
- **Controller Integration**: Controller calling service
- **Database Integration**: ORM operations
- **Transaction Testing**: Multi-step operations
- **Error Propagation**: Errors properly bubbled up
- **State Management**: State changes flow correctly

### 6. Generate E2E Tests
- **Critical Paths**: Must-work user workflows
- **CRUD Operations**: Create, read, update, delete flows
- **Form Submission**: Form validation and submission
- **Navigation**: Page transitions and routing
- **Error Handling**: Error messages display correctly
- **Performance**: Page loads within acceptable time
- **Accessibility**: Keyboard navigation works
- **Cross-browser**: Works on Chrome, Firefox, Safari

### 7. Generate Test Fixtures
- **Mock Data Factories**: Reusable test data generation
- **Faker Integration**: Random realistic data
- **Default Overrides**: Override specific fields
- **Relationships**: Proper foreign key setup
- **Batch Creation**: Efficient data setup
- **Unique Constraints**: Handle unique field requirements
- **Cleanup**: Proper teardown of fixtures

### 8. Implement Test Isolation
- **Setup/Teardown**: BeforeEach/AfterEach cleanup
- **Database Transactions**: Rollback after each test
- **Mock Reset**: Mocks reset between tests
- **Timer Cleanup**: Clear timers/intervals
- **Event Cleanup**: Remove event listeners
- **No Shared State**: Each test independent
- **Parallel Safety**: Tests can run in parallel

### 9. Add Mocking Strategy
- **Module Mocks**: jest.mock() for dependencies
- **Partial Mocks**: jest.spyOn() for selective mocking
- **Mock Implementation**: Provide mock return values
- **Mock Verification**: Verify mock calls
- **Mock Restore**: Restore original after test
- **Spy Tracking**: Track function calls and arguments

### 10. Run Tests & Report
- **Execute Tests**: npm test with coverage
- **Parse Results**: Extract pass/fail status
- **Generate Report**: Coverage report and summary
- **List Failures**: Failed tests with error messages
- **Coverage Summary**: Coverage per file and overall
- **Trend Analysis**: Coverage improvement over time
- **Recommendations**: Areas needing test work

### 11. Fix Test Failures
- **Analyze Failures**: Understand failure cause
- **Identify Issues**: Code bug or test bug
- **Fix Code**: Correct implementation bugs
- **Fix Tests**: Correct test expectations if needed
- **Re-run Tests**: Verify fixes work
- **Prevent Regression**: Ensure fix doesn't break other tests

### 12. Verify Test Quality
- **No Flaky Tests**: Run tests 10x, all pass
- **Meaningful Assertions**: Each assert verifies behavior
- **Good Names**: Test names describe what is tested
- **Documentation**: Test purpose is clear
- **Performance**: Tests run quickly (<1s unit, <5s integration)
- **Independence**: Tests don't affect each other

## Quality Criteria

- 80%+ overall line coverage
- 90%+ coverage on critical paths
- All error paths tested
- All branches covered
- No flaky tests
- All tests pass consistently
- Tests run in <30 seconds for unit tests
- Tests run in <2 minutes for integration
- Clear test names describing behavior
- Proper test isolation and cleanup
- No warnings from test framework
- Mock data realistic and maintainable

## Output Expectations

- Generated test files for untested code
- All tests passing
- Coverage report showing improvement
- No test failures or warnings
- Test output readable and informative
- Coverage gaps identified
- Recommendations for additional testing
- Performance metrics for test suite
- Mock factories for common data
- Proper test isolation and cleanup
- Well-documented test code
- Ready to add to CI/CD pipeline

## Success Indicators

- `npm test` passes with all tests green
- Coverage meets 80%+ minimum
- Critical paths have 90%+ coverage
- `npm test -- --coverage` shows improvement
- All error paths are tested
- No flaky test failures over 10 runs
- Tests run in <30 seconds
- Coverage report is clear and actionable
- New tests follow existing patterns
- Test names clearly describe behavior
