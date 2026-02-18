---
name: /quick
description: >
  Quick mode bypass. Skips full pipeline for simple tasks. Direct execution with natural
  quality. Use for small changes, quick fixes, and low-risk modifications without full
  14-phase pipeline overhead.
---

# Quick Command

## Purpose
Execute simple tasks directly without full pipeline overhead, maintaining quality while moving fast.

## When to Use
- Simple bug fixes (<50 lines)
- Documentation updates
- Configuration changes
- Dependency updates
- Small feature additions (<100 lines)
- Refactoring small sections
- Test additions for small code
- Performance tweaks (simple)
- Style/formatting fixes
- Comments and documentation

## When NOT to Use
- Major features (use /implement)
- Architecture changes (use /deep)
- Security-critical changes (use /secure)
- Database schema changes (use /deep)
- API contract changes (use /deep)
- Performance-critical optimization (use /optimize)
- Complex refactoring (use /refactor)
- New library integration (use /deep)

## Execution Steps

### 1. Quick Assessment
- **Scope**: Confirm task is small (<100 lines)
- **Risk**: Assess risk as low
- **Testing**: Simple testing sufficient
- **Documentation**: Minimal docs needed
- **Approval**: Quick review sufficient
- **Complexity**: No complex interactions

### 2. Direct Implementation
- **Code**: Implement changes directly
- **Quality**: Maintain code quality standards
- **Testing**: Add minimal tests
- **Type Safety**: Maintain TypeScript strictness
- **Linting**: Pass linting checks
- **Format**: Auto-format with Prettier

### 3. Lightweight Testing
- **Unit Test**: Single test for functionality
- **Manual Test**: Quick manual verification
- **No Regression**: Quick check for breakage
- **Fast**: Test execution <10 seconds
- **Coverage**: Doesn't need 80%+ coverage
- **Simple**: Simple assertion-based tests

### 4. Minimal Documentation
- **Comments**: Add if non-obvious
- **JSDoc**: Add if new public function
- **Changelog**: Single line entry
- **No Architecture Docs**: Not needed
- **No README Update**: Unless critical
- **PR Description**: Clear description of change

### 5. Quick Review
- **Peer Review**: Quick review (5-10 min)
- **Code Quality**: Confirm standards met
- **Tests Pass**: Verify tests pass
- **No Breaking Changes**: Confirm no breaking changes
- **Approval**: Simple approval

### 6. Direct Merge
- **Merge**: Merge directly to main
- **Tag**: No special tagging needed
- **Deploy**: Deploy directly if approved
- **Monitor**: Quick monitoring for issues
- **Rollback**: Ready to quick rollback if needed

## Examples of Quick Tasks

**Typo Fixes**
- Fix typos in comments/strings
- Fix variable naming issues
- Fix formatting inconsistencies

**Simple Bugs**
- Off-by-one errors
- Missing null checks
- Simple logic fixes
- Incorrect property access

**Documentation**
- Add missing comments
- Update README section
- Fix API documentation
- Add usage example

**Configuration**
- Update environment variables
- Change configuration values
- Adjust build settings
- Modify test configuration

**Tests**
- Add test for existing function
- Fix failing test
- Add edge case test
- Improve test coverage for small function

**Dependencies**
- Update patch version (1.2.3 → 1.2.4)
- Fix deprecation warning
- Update dev dependency
- Remove unused dependency

**Performance Tweaks**
- Add simple caching
- Optimize query (add index)
- Enable compression
- Add memoization

## Quality Standards (Simplified)

- Code follows project style
- TypeScript strict compliance
- Tests pass (if applicable)
- Linting passes
- No obvious bugs
- One peer review
- <100 lines of changes
- <10 minutes peer review time
- Clear commit message

## Output Expectations

- Changes committed to branch
- Code merged to main
- Deployed if applicable
- Quick check for errors
- Ready for next task
- Minimal documentation

## Success Indicators

- Change completes quickly
- Quality maintained
- Tests pass
- No regressions
- Change works as intended
- Team satisfied
- Ready to deploy immediately
- No followup work needed
