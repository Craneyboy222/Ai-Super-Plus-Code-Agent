---
name: /optimize
description: >
  Performance optimization. Profile, identify bottlenecks, implement optimizations,
  benchmark before/after, verify no regressions. Increases performance by 2-10x on
  common operations.
---

# Optimize Command

## Purpose
Systematically identify and eliminate performance bottlenecks to achieve 2-10x performance improvements.

## When to Use
- User reports slowness in specific feature
- Performance benchmarks exceed targets
- Regular performance improvement cycle
- Pre-deployment performance validation
- Memory usage concerns
- Database query slowness
- Bundle size too large
- API response times too slow

## Execution Steps

### 1. Establish Baselines
- **Identify Metrics**: What to measure
- **Create Benchmarks**: Baseline measurements
- **Load Testing**: Realistic load simulation
- **Profile**: Current resource usage
- **Measure**: Response times, throughput
- **Monitor**: Collection of baseline data
- **Document**: Record baseline measurements
- **Target**: Define desired performance target

### 2. Identify Bottlenecks

**CPU Profiling**
- **Hot Functions**: Functions consuming most CPU
- **Call Frequency**: Most-called functions
- **Call Duration**: Functions taking most time
- **Flame Graphs**: Visualize call hierarchy
- **Tools**: Node inspector, Chrome DevTools
- **Analysis**: Identify optimization targets
- **Prioritize**: Focus on high-impact areas

**Memory Profiling**
- **Heap Size**: Current heap usage
- **Retained Objects**: Objects kept in memory
- **Memory Leaks**: Potential memory leaks
- **Heap Snapshots**: Before/after comparisons
- **Tools**: Node inspector, DevTools
- **Allocation**: Object allocation hotspots

**Database Profiling**
- **Slow Queries**: Queries >100ms
- **Query Plans**: EXPLAIN analysis
- **Frequency**: Most-executed queries
- **Indexes**: Used vs unused indexes
- **Lock Contention**: Transaction locks
- **Connection Pool**: Pool utilization

**Network Profiling**
- **Request Size**: Payload sizes
- **Request Count**: Number of requests
- **Response Time**: Server response time
- **Latency**: Network latency
- **Waterfall**: Request timing breakdown
- **Caching**: Cache effectiveness

**Frontend Profiling**
- **Core Web Vitals**: LCP, FID, CLS
- **JavaScript**: Parse, compile, execution time
- **Rendering**: Layout, paint, composite time
- **Bundle Size**: JavaScript bundle size
- **Assets**: CSS, image, font size
- **Tools**: Lighthouse, WebPageTest, Chrome DevTools

### 3. Optimization Implementation

**Database Optimizations**
- **Indexes**: Add indexes on frequently filtered columns
- **Query Optimization**: Rewrite inefficient queries
- **N+1 Fixes**: Use JOINs instead of loops
- **Select Specificity**: Only fetch needed columns
- **Batch Operations**: Bulk insert/update
- **Connection Pooling**: Reuse database connections
- **Query Caching**: Cache expensive queries
- **Materialized Views**: Pre-computed data

**API Optimizations**
- **Pagination**: Limit results per request
- **Sparse Fields**: Client specifies columns needed
- **Caching**: Cache responses with proper TTL
- **Compression**: gzip/brotli compression
- **Rate Limiting**: Prevent overuse
- **Request Batching**: Combine multiple requests
- **GraphQL**: Query only needed fields

**Frontend Optimizations**
- **Code Splitting**: Split into smaller chunks
- **Lazy Loading**: Load code on-demand
- **Tree Shaking**: Remove unused code
- **Minification**: Reduce code size
- **Memoization**: Prevent unnecessary re-renders
- **Virtual Scrolling**: Render only visible items
- **Image Optimization**: Compress and format
- **Service Worker**: Offline caching
- **CDN**: Distribute globally
- **Compression**: Gzip/brotli compression

**Memory Optimizations**
- **Garbage Collection**: Tune GC parameters
- **Streaming**: Process large datasets in chunks
- **WeakMap**: Use for non-essential references
- **Cleanup**: Proper cleanup of event listeners
- **Avoid Closures**: Don't retain unnecessary variables
- **Object Reuse**: Reuse objects when possible
- **Size Limits**: Limit cache/buffer sizes

**Caching Strategies**
- **Browser Cache**: Cache static assets
- **HTTP Caching**: Set proper cache headers
- **Redis Cache**: Cache expensive queries
- **In-Memory Cache**: Application-level cache
- **CDN**: Distribute cached content
- **Cache Invalidation**: Smart invalidation strategy
- **Conditional Requests**: ETags and 304 responses

### 4. Verification & Testing
- **Benchmark**: Measure after optimization
- **Comparison**: Before/after comparison
- **Regression**: Ensure no behavior changes
- **Load Test**: Test with realistic load
- **Stress Test**: Test at peak capacity
- **Soak Test**: Long-duration stability test
- **Monitoring**: Monitor metrics post-deploy
- **User Feedback**: Verify user-perceived improvement

### 5. Performance Benchmarking
- **Baseline**: Record baseline measurements
- **Target**: Set performance target (2x, 3x improvement)
- **Implement**: Implement optimizations
- **Measure**: Measure optimized version
- **Compare**: Calculate improvement percentage
- **Document**: Document optimization and results
- **Commit**: Commit with before/after metrics
- **Monitor**: Track metrics over time

### 6. Regression Testing
- **Unit Tests**: All tests still pass
- **Integration Tests**: Components work together
- **E2E Tests**: User workflows unchanged
- **Performance**: Performance targets met
- **Memory**: No memory leaks introduced
- **Type Checking**: TypeScript strict compliance
- **Lint**: No linting errors
- **Manual Testing**: Verify key workflows

### 7. Production Monitoring
- **Metrics**: Monitor key performance metrics
- **Alerts**: Alert on performance regressions
- **Dashboards**: Real-time performance visibility
- **Logging**: Structured logging for debugging
- **Tracing**: Distributed tracing of requests
- **Error Rate**: Monitor for increased errors
- **User Impact**: Monitor user-reported performance
- **Comparison**: Compare to baseline

### 8. Documentation & Knowledge
- **Optimization**: Document what was optimized
- **Results**: Record performance improvements
- **Technique**: Document optimization technique
- **Lessons**: Share learnings with team
- **Playbook**: Add to optimization playbook
- **Prevent**: Suggest process to prevent regression
- **Best Practices**: Document best practices
- **Training**: Teach team optimization skills

## Optimization Techniques

**Quick Wins**
- Enable gzip/brotli compression (30-70% savings)
- Add database indexes (10-100x speedup)
- Implement HTTP caching (no server load)
- Code splitting (50% faster initial load)
- Remove unused dependencies (20-50% bundle)

**Medium Effort**
- Query optimization (5-10x speedup)
- Implement caching layer (10-100x speedup)
- API response pagination (10x memory)
- Image optimization (40-60% savings)
- Lazy loading components (faster initial)

**High Impact**
- Database schema redesign (100x speedup possible)
- Service architecture (horizontal scaling)
- CDN implementation (90%+ cache hit)
- Connection pooling (10x throughput)
- Async job processing (responsive UI)

## Quality Criteria

- 2-10x performance improvement achieved
- No behavior changes introduced
- All tests passing
- No memory leaks
- No new errors introduced
- Monitoring shows improvement maintained
- Documentation complete
- Team understands optimization

## Output Expectations

- Performance improvement measured and documented
- Before/after metrics in commit message
- Optimizations explain in code comments
- Monitoring setup for tracking
- Regression tests added
- Documentation updated
- Team trained on optimization
- Process change to prevent regression

## Success Indicators

- Benchmark shows 2-10x improvement
- All tests pass
- No regressions in behavior
- Monitoring shows sustained improvement
- User-perceived improvement confirmed
- No errors introduced
- Code quality maintained
- Team satisfied with optimization
