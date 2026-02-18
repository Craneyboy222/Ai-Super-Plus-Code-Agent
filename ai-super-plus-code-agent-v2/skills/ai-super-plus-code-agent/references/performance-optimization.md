# Performance Optimization - Expert Reference

## Table of Contents

1. [Profiling Methodology](#profiling-methodology)
2. [Frontend Optimization](#frontend-optimization)
3. [Backend Optimization](#backend-optimization)
4. [Caching Strategies](#caching-strategies)
5. [Memory Management](#memory-management)
6. [Async Patterns](#async-patterns)
7. [Load Testing](#load-testing)
8. [Monitoring and APM](#monitoring-and-apm)

---

## Profiling Methodology

### CPU Profiling

**CRITICAL**: Identify hot code paths:

```python
# Python: cProfile
import cProfile
import pstats
from io import StringIO

def expensive_operation():
    total = 0
    for i in range(1000000):
        total += i ** 2
    return total

# Profile
profiler = cProfile.Profile()
profiler.enable()

expensive_operation()

profiler.disable()
stats = pstats.Stats(profiler, stream=StringIO())
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10 functions

# Output shows:
# ncalls: Number of function calls
# tottime: Total time in function (excluding called functions)
# cumtime: Cumulative time (including called functions)
```

**HIGH**: Node.js profiling:

```javascript
// Node.js: Built-in profiling
const profiler = require('v8').writeHeapSnapshot;
const fs = require('fs');

console.time('expensive-operation');
// Code to profile
expensiveOperation();
console.timeEnd('expensive-operation');

// Flame graphs (with clinic.js)
// npm install -g clinic
// clinic flame -- node app.js
```

### Memory Profiling

**HIGH**: Detect memory leaks:

```python
# Python: Memory profiler
from memory_profiler import profile

@profile  # Decorator tracks memory line-by-line
def memory_intensive():
    large_list = list(range(1000000))
    result = sum(large_list)
    return result

# Run with: python -m memory_profiler script.py
# Output shows memory usage per line

# Using tracemalloc for snapshots
import tracemalloc

tracemalloc.start()

# Code execution

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

for stat in top_stats[:10]:
    print(stat)
```

### I/O Profiling

**CRITICAL**: Identify I/O bottlenecks:

```javascript
// Node.js: I/O monitoring
const fs = require('fs');
const perf_hooks = require('perf_hooks');

const { performance, PerformanceObserver } = perf_hooks;

const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach((entry) => {
    if (entry.duration > 10) {  // Flag slow operations
      console.warn(`Slow I/O: ${entry.name} took ${entry.duration}ms`);
    }
  });
});

obs.observe({ entryTypes: ['measure', 'function'] });

performance.mark('read-start');
const data = fs.readFileSync('large-file.txt');
performance.mark('read-end');
performance.measure('file-read', 'read-start', 'read-end');
```

---

## Frontend Optimization

### Core Web Vitals

**CRITICAL**: Optimize for user experience:

```
LCP (Largest Contentful Paint):
  Target: < 2.5 seconds
  - Optimize server response time (TTFB)
  - Minimize CSS that blocks rendering
  - Optimize images and fonts
  - Remove unused JavaScript

FID (First Input Delay) / INP (Interaction to Next Paint):
  Target: < 100ms
  - Break up long JavaScript tasks (> 50ms)
  - Use Web Workers for heavy computation
  - Defer non-critical JavaScript

CLS (Cumulative Layout Shift):
  Target: < 0.1
  - Avoid layout shifts from late-loading content
  - Set explicit width/height on images
  - Avoid inserting content above existing content
```

### Bundle Size Optimization

**HIGH**: Minimal JavaScript:

```javascript
// Webpack configuration
module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist')
  },

  optimization: {
    minimize: true,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor libraries in separate bundle
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        },
        // Common code shared by multiple modules
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader',
        options: {
          presets: [
            ['@babel/preset-env', { modules: false }]
          ],
          plugins: [
            '@babel/plugin-syntax-dynamic-import'
          ]
        }
      }
    ]
  }
};

// Analyze bundle
// npm install -D webpack-bundle-analyzer
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
plugins: [new BundleAnalyzerPlugin()]
```

### Lazy Loading and Code Splitting

**CRITICAL**: Load code when needed:

```javascript
// Dynamic imports for code splitting
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

export default function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// Route-based code splitting
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Profile = React.lazy(() => import('./pages/Profile'));

function Router() {
  return (
    <Routes>
      <Route path="/" element={<Suspense fallback={<Loading />}><Dashboard /></Suspense>} />
      <Route path="/profile" element={<Suspense fallback={<Loading />}><Profile /></Suspense>} />
    </Routes>
  );
}

// Library code splitting
import('lodash').then(_ => {
  // Use lodash only when needed
});
```

### Image Optimization

**HIGH**: Modern formats and responsive images:

```html
<!-- Modern image formats with fallback -->
<picture>
  <source srcset="image.webp" type="image/webp">
  <source srcset="image.jpg" type="image/jpeg">
  <img src="image.jpg" alt="Description" loading="lazy">
</picture>

<!-- Responsive images -->
<img
  srcset="small.jpg 480w,
          medium.jpg 768w,
          large.jpg 1200w"
  sizes="(max-width: 480px) 100vw,
         (max-width: 768px) 50vw,
         33vw"
  src="medium.jpg"
  alt="Responsive image"
  loading="lazy">
```

### Font Optimization

**MEDIUM**: Efficient font loading:

```css
/* Preload critical fonts */
<link rel="preload" href="fonts/main.woff2" as="font" type="font/woff2" crossorigin>

/* font-display: swap to prevent FOIT/FOUT */
@font-face {
  font-family: 'Main';
  src: url('fonts/main.woff2') format('woff2');
  font-display: swap;  /* Show fallback immediately */
}

/* Subsetting for specific languages */
@font-face {
  font-family: 'Main';
  src: url('fonts/main-latin.woff2') format('woff2');
  unicode-range: U+0000-00FF;  /* Latin characters only */
}
```

---

## Backend Optimization

### Query Optimization

**CRITICAL**: Analyze and optimize database queries:

```sql
-- PostgreSQL query analysis
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- Look for:
-- - Seq Scan (bad) vs Index Scan (good)
-- - Nested Loop (expensive) vs Hash Join (better)
-- - Buffers: Hits (cache) vs Reads (disk)

-- If slow, add indexes
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);
CREATE INDEX idx_users_created ON users(created_at);
```

### Connection Pooling

**CRITICAL**: Right-size connection pool:

```python
# SQLAlchemy pool configuration
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=20,           # Connections to keep in pool
    max_overflow=10,        # Additional connections if needed
    pool_recycle=3600,      # Recycle connections hourly
    pool_pre_ping=True      # Test connection before use
)

# Monitor pool state
print(f"Checked-in: {engine.pool.checkedin()}")
print(f"Checked-out: {engine.pool.checkedout()}")
print(f"Size: {engine.pool.size()}")
```

### Batch Operations

**HIGH**: Reduce round-trips to database:

```python
# ✗ WRONG: N queries
for user_id in user_ids:
    orders = db.session.query(Order).filter(Order.user_id == user_id).all()

# ✓ CORRECT: 1 query
orders = db.session.query(Order).filter(Order.user_id.in_(user_ids)).all()

# Bulk insert for large datasets
users = [User(email=f'user{i}@test.com') for i in range(10000)]
db.session.bulk_insert_mappings(User, users)
db.session.commit()

# vs
for user in users:
    db.session.add(user)
db.session.commit()  # Much slower
```

---

## Caching Strategies

### Redis Caching

**CRITICAL**: Cache hot data:

```python
import redis
import json
from functools import wraps
from datetime import timedelta

cache = redis.Redis(host='localhost', port=6379, decode_responses=True)

def cache_result(ttl_seconds=3600):
    """Decorator to cache function results"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Generate cache key from function name and arguments
            cache_key = f"{func.__name__}:{args}:{kwargs}"

            # Try to get from cache
            cached = cache.get(cache_key)
            if cached is not None:
                return json.loads(cached)

            # If not cached, call function
            result = func(*args, **kwargs)

            # Store in cache
            cache.setex(
                cache_key,
                timedelta(seconds=ttl_seconds),
                json.dumps(result)
            )

            return result
        return wrapper
    return decorator

@cache_result(ttl_seconds=3600)
def get_user_orders(user_id):
    """Expensive query, cached for 1 hour"""
    return db.session.query(Order).filter(Order.user_id == user_id).all()

# Explicit cache invalidation
def update_user(user_id, data):
    user = db.session.query(User).get(user_id)
    for key, value in data.items():
        setattr(user, key, value)
    db.session.commit()

    # Invalidate related caches
    cache.delete(f"get_user_orders:{(user_id,):{{}}")
```

### CDN Caching

**HIGH**: Cache static assets:

```python
# Flask with CDN headers
from flask import Flask, make_response
from datetime import datetime, timedelta

app = Flask(__name__)

@app.route('/static/<path:filename>')
def serve_static(filename):
    response = make_response(send_file(f'static/{filename}'))

    # Cache for 1 year (for versioned assets)
    response.headers['Cache-Control'] = 'public, max-age=31536000, immutable'
    response.headers['ETag'] = generate_etag(filename)

    return response

@app.route('/api/data')
def api_data():
    response = make_response(get_data())

    # No cache for dynamic content
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'

    return response
```

### Browser Caching

**MEDIUM**: Leverage browser cache:

```javascript
// Service Worker for advanced caching
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll([
        '/',
        '/css/main.css',
        '/js/app.js'
      ]);
    })
  );
});

self.addEventListener('fetch', event => {
  // Network first, fallback to cache
  event.respondWith(
    fetch(event.request)
      .then(response => {
        // Cache successful responses
        if (response.status === 200) {
          const cache = caches.open('v1');
          cache.then(c => c.put(event.request, response.clone()));
        }
        return response;
      })
      .catch(() => caches.match(event.request))  // Fallback to cached
  );
});
```

---

## Memory Management

### Memory Leak Detection

**CRITICAL**: Monitor heap and find leaks:

```python
# Python: Detect unreleased objects
import gc
import tracemalloc

tracemalloc.start()

# Run suspicious code
process_large_batch()

# Take snapshot
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024}MB; Peak: {peak / 1024 / 1024}MB")

# If memory grows unbounded, enable garbage collection debugging
gc.set_debug(gc.DEBUG_LEAK)

# Find unreferenced objects
gc.collect()
unreachable = gc.garbage
for obj in unreachable[:10]:
    print(f"Unreachable: {type(obj)} - {obj}")
```

**HIGH**: JavaScript memory management:

```javascript
// Node.js heap snapshot
const v8 = require('v8');
const fs = require('fs');
const path = require('path');

function writeHeapSnapshot(label) {
  const filename = path.join('.', `heap-${label}-${Date.now()}.heapsnapshot`);
  const snapshot = v8.writeHeapSnapshot(filename);
  console.log(`Heap snapshot written to ${filename}`);
}

// Periodic snapshots
setInterval(() => {
  writeHeapSnapshot('periodic');
}, 60000);  // Every minute

// Analyze with Chrome DevTools Memory tab
```

### Garbage Collection Optimization

**HIGH**: Tune GC for application workload:

```bash
# Node.js GC tuning
# Default: Automatic, triggered when heap reaches threshold

# Increase heap for less frequent GC
node --max-old-space-size=8192 app.js  # 8GB heap

# Force explicit GC calls (for profiling)
node --expose-gc app.js

# Code: Manually trigger GC
if (global.gc) {
  global.gc();
} else {
  console.warn('GC not exposed, run with --expose-gc');
}

# Monitor GC pauses
node --trace-gc app.js
```

### Pool Allocation Patterns

**MEDIUM**: Reuse objects to reduce allocations:

```python
# Object pool pattern (reduce GC pressure)
class ObjectPool:
    def __init__(self, object_class, initial_size=100):
        self.pool = [object_class() for _ in range(initial_size)]
        self.available = list(self.pool)

    def acquire(self):
        if self.available:
            return self.available.pop()
        return type(self.pool[0])()  # Create new if exhausted

    def release(self, obj):
        obj.reset()  # Clear state
        self.available.append(obj)

# Usage
buffer_pool = ObjectPool(BytesIO, initial_size=1000)

def process_request():
    buffer = buffer_pool.acquire()
    try:
        # Use buffer
        buffer.write(b'data')
    finally:
        buffer_pool.release(buffer)
```

---

## Async Patterns

### Event Loop Optimization

**CRITICAL**: Non-blocking I/O:

```javascript
// ✓ CORRECT: Non-blocking I/O
const fs = require('fs').promises;

async function readFile(path) {
  try {
    const data = await fs.readFile(path);
    return data;
  } catch (err) {
    console.error(err);
  }
}

// ✗ WRONG: Blocking I/O (blocks event loop)
const data = fs.readFileSync(path);

// Handling long-running tasks
async function processLargeDataset() {
  const items = await getItems();

  // Process in batches to yield to event loop
  const batchSize = 100;
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await processBatch(batch);

    // Yield to event loop
    await new Promise(resolve => setImmediate(resolve));
  }
}
```

### Worker Threads

**HIGH**: CPU-intensive work off main thread:

```javascript
// Main thread
const { Worker } = require('worker_threads');

function heavyComputation(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js');

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0)
        reject(new Error(`Worker stopped with exit code ${code}`));
    });

    worker.postMessage(data);
  });
}

// Usage
const result = await heavyComputation(largeDataset);

// worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (data) => {
  const result = expensiveCalculation(data);
  parentPort.postMessage(result);
});
```

### Concurrency Patterns

**MEDIUM**: Rate limiting concurrent operations:

```python
import asyncio
from asyncio import Semaphore

async def rate_limited_requests(urls, max_concurrent=5):
    """Limit concurrent requests to 5"""
    semaphore = Semaphore(max_concurrent)

    async def fetch_with_semaphore(url):
        async with semaphore:
            return await fetch(url)

    tasks = [fetch_with_semaphore(url) for url in urls]
    return await asyncio.gather(*tasks)

# Usage
results = await rate_limited_requests([...100 urls...])
```

---

## Load Testing

### Baseline Establishment

**CRITICAL**: Document expected performance:

```bash
# k6 baseline test
#!/bin/bash
k6 run --vus 10 --duration 30s tests/baseline.js > baseline.json

# Results should show:
# - Response time p95 < 200ms
# - Error rate < 0.1%
# - Throughput > 100 req/s

# Store baseline for regression detection
echo "Baseline established: $(date)" >> performance-log.md
jq '.metrics' baseline.json >> performance-log.md
```

### Stress Testing

**HIGH**: Find breaking point:

```javascript
// k6 stress test
import http from 'k6/http';

export const options = {
  stages: [
    { duration: '5m', target: 10 },    // Warm up
    { duration: '10m', target: 50 },   // Ramp up
    { duration: '5m', target: 100 },   // Stress
    { duration: '5m', target: 200 },   // Spike
    { duration: '5m', target: 0 }      // Cool down
  ],
  thresholds: {
    http_req_duration: ['p(99)<1000', 'p(95)<500'],
    http_req_failed: ['rate<0.05']
  }
};

export default function () {
  http.get('https://api.example.com/data');
}

// Run: k6 run stress-test.js
// Monitor when tests fail to find capacity limits
```

### Capacity Planning

**MEDIUM**: Calculate infrastructure needs:

```
Capacity Planning Formula:

Peak Load = (Concurrent Users) × (Requests per User per Second)
            × (Average Response Time in Seconds)

Example:
- 10,000 concurrent users
- 2 requests/user/sec during peak
- 100ms response time

Peak Load = 10,000 × 2 × 0.1 = 2,000 requests/sec

If each server handles 100 req/s:
Servers needed = 2,000 / 100 = 20 servers

With 30% headroom for traffic spikes:
Servers needed = 20 × 1.3 = 26 servers
```

---

## Monitoring and APM

### Application Performance Monitoring

**CRITICAL**: Real-time performance visibility:

```javascript
// New Relic agent
require('newrelic');

const newrelic = require('newrelic');

app.get('/api/users/:id', (req, res) => {
  newrelic.startSegment('database-query', true, () => {
    // Database query - timed automatically
    return getUserFromDB(req.params.id);
  });

  // Custom metric
  newrelic.recordMetric('custom/user-lookup', 1);
});

// Custom transaction
newrelic.noticeError(new Error('User not found'), {
  userId: req.params.id
});
```

### Key Metrics to Monitor

**HIGH**: Essential performance indicators:

```
LATENCY:
  - p50, p95, p99 response times
  - API endpoint latencies
  - Database query times
  - Cache hit rates

THROUGHPUT:
  - Requests per second
  - Transactions per second
  - Data processed per second

ERROR RATE:
  - Overall error rate (%)
  - Error rate by endpoint
  - Timeout rate

RESOURCE UTILIZATION:
  - CPU usage
  - Memory usage
  - Disk I/O
  - Network I/O
  - Database connection pool utilization
```

### Dashboard and Alerting

**MEDIUM**: Actionable alerts:

```yaml
# Prometheus alerting rules
groups:
  - name: performance
    rules:
      - alert: HighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 1
        for: 5m
        annotations:
          summary: "API latency p95 > 1 second"

      - alert: HighErrorRate
        expr: rate(http_requests_failed[5m]) > 0.05
        for: 2m
        annotations:
          summary: "Error rate > 5%"

      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes / 1024 / 1024 > 1024
        for: 1m
        annotations:
          summary: "Memory usage > 1GB"
```

---

## Summary: Performance Checklist

- [ ] CPU profiling completed, hot paths identified
- [ ] Memory profiling shows no leaks
- [ ] Core Web Vitals: LCP < 2.5s, INP < 100ms, CLS < 0.1
- [ ] Bundle size < 150KB (gzipped)
- [ ] Code splitting implemented for routes
- [ ] Images optimized (WebP, lazy-loading)
- [ ] Database queries analyzed with EXPLAIN
- [ ] Connection pool properly sized
- [ ] N+1 queries eliminated
- [ ] Redis caching for hot data
- [ ] CDN configured for static assets
- [ ] Service Worker for offline support
- [ ] Memory pools for reused objects
- [ ] Event loop not blocked (async/await)
- [ ] CPU work offloaded to workers
- [ ] Baseline performance established
- [ ] Stress tests completed
- [ ] APM monitoring in place
- [ ] Alerts configured for degradation
- [ ] Capacity planning documented
