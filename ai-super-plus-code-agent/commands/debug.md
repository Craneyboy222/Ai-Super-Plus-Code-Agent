---
name: /debug
description: >
  Systematic debugging. Hypothesis generation, root cause analysis, fix implementation,
  regression test creation. Provides guided debugging process to identify and fix bugs
  methodically.
---

# Debug Command

## Purpose
Systematically identify, analyze, and fix bugs with comprehensive root cause analysis and regression testing.

## When to Use
- Investigating reported bugs
- Fixing production issues
- Performance regressions
- Intermittent failures
- Unexpected behavior
- Test failures
- User-reported issues

## Execution Steps

### 1. Bug Report Analysis
- **Describe Issue**: Clear description of what's broken
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happens
- **Reproduction Steps**: Steps to reproduce
- **Environment**: Browser, OS, Node version
- **Logs**: Error messages and stack traces
- **Screenshots**: Visual evidence if applicable
- **Frequency**: Always, intermittent, rare

### 2. Hypothesis Generation
- **Understand Context**: What code path is involved
- **List Possibilities**: Possible causes (5-10)
- **Prioritize Likely**: Most probable causes first
- **Quick Tests**: Tests to quickly eliminate possibilities
- **Narrow Scope**: Focus on likely issue area
- **Test Plan**: Steps to verify each hypothesis
- **Evidence Collection**: What proves/disproves each

### 3. Root Cause Analysis
- **Investigate Code**: Examine relevant code paths
- **Add Logging**: Strategic logging to trace execution
- **Set Breakpoints**: Debugger breakpoints at key points
- **Step Through**: Execute step-by-step
- **Inspect State**: Check variable values
- **Monitor Network**: Watch API calls if relevant
- **Check Database**: Verify database state
- **Review Logs**: Search logs for clues
- **Timeline**: When did this start occurring
- **Recent Changes**: What changed recently

### 4. Testing in Isolation
- **Create Test Case**: Minimal reproducible example
- **Isolate Variables**: Change one variable at a time
- **Unit Test**: Test just the problematic function
- **Mock Dependencies**: Mock external dependencies
- **Control Environment**: Control all variables
- **Verify Hypothesis**: Prove/disprove theory
- **Document Findings**: Record what you discovered

### 5. Fix Implementation
- **Understand Impact**: What else might be affected
- **Plan Changes**: How to fix without breaking other things
- **Code Review**: Review fix logic before implementation
- **Implement Fix**: Make minimal changes
- **Test Fix**: Verify fix resolves issue
- **No New Issues**: Ensure no regressions introduced
- **Code Quality**: Fix follows project standards
- **Document Change**: Explain the fix in commit message

### 6. Regression Test Creation
- **Test the Bug**: Create test that fails before fix
- **Verify Fix**: Test passes after fix
- **Test Edge Cases**: Test boundary conditions
- **Test Related**: Test related functionality
- **Integration**: Test with other components
- **E2E Test**: User workflow test if applicable
- **Performance**: Ensure no performance regression
- **Monitor**: Add monitoring for early detection

### 7. Verification & Validation
- **Manual Testing**: Test in development
- **Test Suite**: Run all tests, ensure pass
- **Integration Test**: Test with other features
- **Performance Test**: No performance degradation
- **Production-like**: Test in production-like environment
- **User Acceptance**: User confirms fix works
- **Monitoring**: Set up alerts for issue recurrence
- **Documentation**: Document fix and prevention

### 8. Root Cause Understanding
- **Why It Happened**: Understand root cause
- **When It Started**: Identify when bug introduced
- **Contributing Factors**: What made it possible
- **Prevention**: How to prevent in future
- **Process Improvement**: Better processes to catch earlier
- **Code Review**: Suggest improvements to prevent similar bugs
- **Testing**: Suggest tests that would catch this
- **Documentation**: Update docs if relevant

### 9. Post-Mortem (if applicable)
- **Timeline**: When did issue occur and get detected
- **Detection**: How was issue discovered
- **Impact**: What was affected and for how long
- **Root Cause**: Why did this happen
- **Contributing Factors**: What made it worse
- **What Went Well**: Good response
- **What Could Improve**: Process improvements
- **Action Items**: Prevent recurrence

### 10. Fix Deployment
- **Code Review**: Peer review of fix
- **CI Pass**: All tests pass
- **Documentation**: Commit message is clear
- **Merge**: Merge to main/master
- **Deploy Staging**: Deploy to staging for final testing
- **Monitor**: Watch monitoring during deploy
- **Deploy Production**: Deploy to production
- **Communication**: Notify affected parties

## Debugging Tools & Techniques

**JavaScript/Node.js**
- **Chrome DevTools**: Breakpoints, stepping, inspecting
- **Node Inspector**: `node --inspect script.js`
- **Debugger Statement**: Debug keyword in code
- **Console Logging**: Strategic console.log statements
- **Profiler**: Performance profiler for timing
- **Memory Profiler**: Heap snapshots for leaks
- **Network Tab**: HTTP request inspection

**Database**
- **Slow Query Logs**: Identify slow queries
- **EXPLAIN ANALYZE**: Query execution plan
- **Monitoring**: Database monitoring tools
- **Connection Pooling**: Check connection status
- **Locks**: Identify lock contention
- **Replication**: Check replication lag
- **Backups**: Verify backup integrity

**Frontend**
- **React DevTools**: Component inspection
- **Redux DevTools**: State inspection
- **Vue DevTools**: Vue component inspection
- **Lighthouse**: Performance and accessibility audit
- **WebPageTest**: Detailed performance analysis
- **Sentry**: Error tracking and monitoring
- **LogRocket**: Session replay and logging

## Quality Criteria

- Root cause clearly identified
- Fix addresses root cause, not symptom
- Regression test prevents recurrence
- Fix doesn't introduce new issues
- Code quality standards maintained
- Performance not degraded
- Documentation updated if needed
- Monitoring set up for issue

## Output Expectations

```
BUG_FIX_REPORT.md
├── Bug Description
├── Reproduction Steps
├── Environment Details
├── Initial Investigation
├── Hypotheses Tested
│   ├── Hypothesis 1 (rejected)
│   ├── Hypothesis 2 (rejected)
│   └── Hypothesis 3 (confirmed)
├── Root Cause Analysis
│   ├── Root Cause
│   ├── Contributing Factors
│   └── Timeline
├── Fix Implementation
│   ├── Code Changes
│   ├── Files Modified
│   └── Reasoning
├── Regression Test
│   ├── Test Code
│   └── Test Result
├── Verification
│   ├── Manual Testing
│   ├── Test Suite Results
│   └── Performance Impact
├── Prevention
│   ├── Future Prevention
│   ├── Process Improvements
│   └── Testing Suggestions
├── Monitoring
│   └── Alerts Set Up
└── References
    ├── Related Issues
    └── Documentation Links
```

## Success Indicators

- Root cause identified and understood
- Fix solves the reported problem
- No new issues introduced
- Regression test prevents recurrence
- All tests pass
- Performance not degraded
- Code quality maintained
- Cause prevents similar future bugs
