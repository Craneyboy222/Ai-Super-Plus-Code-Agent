---
name: /refactor
description: >
  Code refactoring with safety. Identifies refactoring targets, creates tests first,
  refactors incrementally, verifies no behavior changes. Ensures code improvement without
  introducing bugs.
---

# Refactor Command

## Purpose
Safely refactor code to improve structure, readability, and maintainability while maintaining functionality.

## When to Use
- Code review identifies refactoring opportunity
- Complexity too high in a component
- Code duplication needs consolidation
- Outdated patterns need modernization
- Performance optimization opportunity
- Moving to new library version
- Technical debt reduction

## Execution Steps

### 1. Identify Refactoring Target
- **Analyze Code**: Understand current structure
- **Metrics**: Complexity, duplication, test coverage
- **Problems**: What specifically needs improvement
- **Scope**: Define boundaries of refactoring
- **Impact Analysis**: What else might be affected
- **Risk Assessment**: How risky is this change
- **Benefit Analysis**: How much will this improve
- **Priority**: High/medium/low priority refactoring

### 2. Plan Refactoring Strategy
- **End State**: What will code look like after
- **Incremental Steps**: Break into small steps
- **Intermediate States**: Code works after each step
- **Test Strategy**: How to verify no regression
- **Dependencies**: What needs refactoring first
- **Rollback Plan**: How to rollback if needed
- **Review**: Plan review before starting
- **Timeline**: Estimate duration

### 3. Create Tests First (Red Phase)
- **Analyze Code**: Understand current behavior
- **Test Current**: Create tests that pass now
- **Edge Cases**: Include boundary conditions
- **Error Cases**: Include error handling
- **Integration**: Include integration points
- **Coverage**: Aim for 80%+ coverage
- **Parametrized**: Test multiple inputs/scenarios
- **Documentation**: Tests document behavior

### 4. Refactor Code (Green Phase)
- **Small Steps**: Make small incremental changes
- **Run Tests**: After each change, run tests
- **Verify Green**: All tests pass after each change
- **No Changes Yet**: Don't change tests, only code
- **Type Safety**: Maintain TypeScript strictness
- **Code Review**: Review refactored code
- **Quality**: Maintain or improve code quality
- **Performance**: No performance regression

### 5. Verify No Behavior Change
- **Run Full Suite**: Complete test suite passes
- **Code Coverage**: Coverage maintained or improved
- **Integration Test**: Test with dependent code
- **E2E Test**: User workflows still work
- **Performance**: No performance regression
- **Type Check**: No new TypeScript errors
- **Lint**: No linting errors
- **Manual Testing**: Manually verify key flows

### 6. Type Safety & Strictness
- **No Any Types**: Maintain strict TypeScript
- **Explicit Types**: All values properly typed
- **Generics**: Proper generic type usage
- **Type Guards**: Proper type narrowing
- **Return Types**: Explicit return types
- **Parameter Types**: All parameters typed
- **Null Checks**: Proper null/undefined handling
- **Casting**: Minimize type assertions

### 7. Code Style & Quality
- **Formatting**: Run Prettier
- **Linting**: Fix ESLint warnings
- **Naming**: Clear, consistent naming
- **Comments**: Update comments to match code
- **Documentation**: Update JSDoc/docstrings
- **Duplication**: Further DRY improvements
- **Complexity**: Reduce cyclomatic complexity
- **Nesting**: Reduce nesting depth

### 8. Performance Verification
- **Benchmark**: Measure before/after if applicable
- **Profiling**: Check profiling if performance-related
- **Bundle Size**: Verify no bundle size increase
- **Memory**: Check memory usage
- **Speed**: No performance regression
- **Rendering**: Frontend rendering unchanged
- **Queries**: Database queries unchanged
- **Caching**: Caching behavior unchanged

### 9. Incremental Rollout
- **Feature Branch**: Code in feature branch
- **Code Review**: Peer review before merge
- **Merge Main**: Merge to main/develop
- **Staging Deploy**: Deploy to staging for testing
- **Monitor**: Watch metrics during deploy
- **Production Deploy**: Deploy to production
- **Monitor Metrics**: Watch for issues
- **Rollback Plan**: Ready to rollback if needed

### 10. Monitoring & Validation
- **Error Rates**: Monitor error rates
- **Performance**: Monitor performance metrics
- **User Reports**: Watch for user issues
- **Logging**: Monitor application logs
- **Alerts**: Set up alerts for regressions
- **A/B Testing**: Compare before/after metrics
- **Duration**: Monitor for reasonable period
- **Documentation**: Document completion

## Refactoring Patterns

**Extract Method**
- **Identify**: Long method with multiple concerns
- **Extract**: Pull out cohesive piece into new method
- **Test**: Add tests for new method
- **Call**: Call new method from original
- **Verify**: Tests still pass
- **Repeat**: Extract until appropriate size

**Extract Class**
- **Identify**: Class with too many responsibilities
- **Extract**: Create new class with related methods
- **Move**: Move methods and fields to new class
- **Dependencies**: Inject dependency in original
- **Test**: Add tests for new class
- **Verify**: All tests pass

**Consolidate Duplicates**
- **Identify**: Duplicate code blocks
- **Extract**: Create shared method/component
- **Replace**: Replace duplicates with calls
- **Test**: Verify all use cases still work
- **Generalize**: Make shared code flexible enough

**Modernize Patterns**
- **Identify**: Outdated patterns (callbacks → promises)
- **Analyze**: Impact of changing pattern
- **Refactor**: Update to modern pattern
- **Test**: Verify behavior unchanged
- **Deprecate**: Deprecate old pattern
- **Migrate**: Update remaining code

## Quality Criteria

- All existing tests pass
- No new test failures
- No TypeScript errors
- No ESLint warnings
- No performance regression
- No bundle size increase
- Code is cleaner/simpler
- Behavior unchanged
- Better maintainability
- Better readability

## Output Expectations

- Refactored code in feature branch
- All tests passing
- Code review approved
- Pull request merged
- Staging/production deployed
- No monitoring alerts
- Documentation updated
- Refactoring change documented
- Performance metrics stable
- Ready for next refactoring

## Success Indicators

- All tests pass after refactoring
- Code is simpler and cleaner
- No behavior changes
- No performance regression
- Type safety maintained
- Code review approved
- No issues reported post-deploy
- Easier to maintain going forward
- Team agrees it was improvement
