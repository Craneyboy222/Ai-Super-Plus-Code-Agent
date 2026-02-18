# Error Handling Patterns - Expert Reference

## Table of Contents

1. [Error Hierarchy Design](#error-hierarchy-design)
2. [HTTP Error Mapping](#http-error-mapping)
3. [Validation Error Patterns](#validation-error-patterns)
4. [Retry Strategies](#retry-strategies)
5. [Logging Strategy](#logging-strategy)
6. [Error Boundaries](#error-boundaries)
7. [Graceful Degradation](#graceful-degradation)
8. [User-Facing Errors](#user-facing-errors)

---

## Error Hierarchy Design

### Base Error Classes

**CRITICAL**: Extensible error hierarchy:

```python
# Error base classes
class AppError(Exception):
    """Base application error"""
    def __init__(self, message, code=None, details=None, cause=None):
        self.message = message
        self.code = code or self.__class__.__name__
        self.details = details or {}
        self.cause = cause
        super().__init__(message)

    def to_dict(self):
        return {
            'error': self.code,
            'message': self.message,
            'details': self.details
        }

# Domain errors (business logic failures)
class ValidationError(AppError):
    """Invalid input data"""
    pass

class DuplicateResourceError(AppError):
    """Resource already exists"""
    pass

class ResourceNotFoundError(AppError):
    """Requested resource doesn't exist"""
    pass

class PermissionDeniedError(AppError):
    """User lacks required permissions"""
    pass

# Infrastructure errors (external service failures)
class DatabaseError(AppError):
    """Database connection or query failed"""
    pass

class ExternalServiceError(AppError):
    """External API call failed"""
    pass

class ConfigurationError(AppError):
    """Missing or invalid configuration"""
    pass

# Usage
try:
    user = find_user_by_email('test@test.com')
    if not user:
        raise ResourceNotFoundError(
            message='User not found',
            code='USER_NOT_FOUND',
            details={'email': 'test@test.com'}
        )
except Exception as err:
    logger.error(str(err), extra={'code': err.code, 'details': err.details})
```

### Error Context and Cause Chains

**HIGH**: Preserve error causality:

```javascript
// Error with context
class AppError extends Error {
  constructor(message, options = {}) {
    super(message);
    this.code = options.code || this.constructor.name;
    this.details = options.details || {};
    this.cause = options.cause;  // Original error
    this.timestamp = new Date();
    this.context = options.context || {};
  }

  toString() {
    let str = `${this.code}: ${this.message}`;
    if (this.cause) {
      str += `\nCaused by: ${this.cause.toString()}`;
    }
    return str;
  }
}

// Usage: Chain errors to preserve stack
async function loadUserProfile(userId) {
  try {
    const user = await fetchUserFromDB(userId);
    const profile = await enrichUserProfile(user);
    return profile;
  } catch (err) {
    throw new AppError(
      'Failed to load user profile',
      {
        code: 'PROFILE_LOAD_FAILED',
        cause: err,
        context: { userId }
      }
    );
  }
}

// Error with full context
try {
  const profile = await loadUserProfile(123);
} catch (err) {
  console.error({
    message: err.message,
    code: err.code,
    originalError: err.cause?.message,
    context: err.context,
    stack: err.stack
  });
}
```

---

## HTTP Error Mapping

### Domain Errors to HTTP Status Codes

**CRITICAL**: Semantic HTTP responses:

```python
# Mapper from domain errors to HTTP
ERROR_TO_STATUS = {
    'ValidationError': 400,
    'DuplicateResourceError': 409,
    'ResourceNotFoundError': 404,
    'PermissionDeniedError': 403,
    'UnauthorizedError': 401,
    'DatabaseError': 500,
    'ExternalServiceError': 502,
    'ConfigurationError': 500
}

def error_to_http_response(error):
    """Convert domain error to HTTP response"""
    status = ERROR_TO_STATUS.get(error.code, 500)

    return {
        'status': status,
        'body': {
            'error': error.code,
            'message': error.message,
            'details': error.details,
            'timestamp': error.timestamp.isoformat()
        }
    }

# Flask integration
@app.errorhandler(AppError)
def handle_app_error(error):
    response = error_to_http_response(error)
    return jsonify(response['body']), response['status']

@app.errorhandler(Exception)
def handle_unexpected_error(error):
    # Log unexpected errors
    logger.exception('Unhandled exception')

    return jsonify({
        'error': 'INTERNAL_ERROR',
        'message': 'An unexpected error occurred',
        'timestamp': datetime.now().isoformat()
    }), 500
```

### Resource-Specific Error Details

**HIGH**: Rich error information:

```javascript
// Express error handler
app.post('/api/users', async (req, res, next) => {
  try {
    const user = await createUser(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);  // Pass to error handler
  }
});

// Comprehensive error handler
app.use((err, req, res, next) => {
  const response = {
    success: false,
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message,
      timestamp: new Date().toISOString(),
      path: req.path,
      method: req.method
    }
  };

  // Add details for validation errors
  if (err.code === 'VALIDATION_ERROR') {
    response.error.validation = err.details;
  }

  // Add retry info for transient errors
  if (err.retryable) {
    response.error.retryAfter = err.retryAfter || 60;
  }

  // Production: Don't leak stack traces
  if (process.env.NODE_ENV === 'development') {
    response.error.stack = err.stack;
  }

  const status = ERROR_STATUS_MAP[err.code] || 500;
  res.status(status).json(response);
});
```

---

## Validation Error Patterns

### Field-Level Validation

**CRITICAL**: Comprehensive validation with detailed feedback:

```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class ValidationResult:
    is_valid: bool
    errors: List[dict]  # [{'field': 'email', 'message': 'Invalid format'}]

def validate_user_input(data) -> ValidationResult:
    """Validate user input with detailed field errors"""
    errors = []

    # Email validation
    if not data.get('email'):
        errors.append({
            'field': 'email',
            'message': 'Email is required',
            'code': 'REQUIRED'
        })
    elif not is_valid_email(data['email']):
        errors.append({
            'field': 'email',
            'message': 'Invalid email format',
            'code': 'INVALID_FORMAT',
            'value': data['email']
        })

    # Name validation
    if not data.get('name'):
        errors.append({
            'field': 'name',
            'message': 'Name is required',
            'code': 'REQUIRED'
        })
    elif len(data['name']) < 1:
        errors.append({
            'field': 'name',
            'message': 'Name must be at least 1 character',
            'code': 'TOO_SHORT',
            'minLength': 1
        })

    # Password validation
    if not data.get('password'):
        errors.append({
            'field': 'password',
            'message': 'Password is required',
            'code': 'REQUIRED'
        })
    elif len(data['password']) < 8:
        errors.append({
            'field': 'password',
            'message': 'Password must be at least 8 characters',
            'code': 'TOO_SHORT',
            'minLength': 8
        })

    return ValidationResult(
        is_valid=len(errors) == 0,
        errors=errors
    )

# Usage in API
@app.post('/api/users')
def create_user():
    validation = validate_user_input(request.json)

    if not validation.is_valid:
        return {
            'error': 'VALIDATION_FAILED',
            'validationErrors': validation.errors
        }, 422
```

### Cross-Field Validation

**HIGH**: Validation involving multiple fields:

```javascript
// Cross-field validation
function validatePasswordChange(data) {
  const errors = [];

  // Individual field validation
  if (!data.currentPassword) {
    errors.push({ field: 'currentPassword', message: 'Required' });
  }

  if (!data.newPassword) {
    errors.push({ field: 'newPassword', message: 'Required' });
  }

  if (!data.confirmPassword) {
    errors.push({ field: 'confirmPassword', message: 'Required' });
  }

  // Cross-field validation (fields must match each other)
  if (data.newPassword && data.confirmPassword) {
    if (data.newPassword !== data.confirmPassword) {
      errors.push({
        field: 'confirmPassword',
        message: 'Passwords do not match',
        relatedField: 'newPassword'
      });
    }
  }

  // Cross-field validation (new password different from current)
  if (data.currentPassword && data.newPassword) {
    if (data.currentPassword === data.newPassword) {
      errors.push({
        field: 'newPassword',
        message: 'New password must be different from current',
        relatedField: 'currentPassword'
      });
    }
  }

  return errors;
}
```

### Async Validation

**MEDIUM**: Validation requiring external calls:

```javascript
// Async validation (check uniqueness in database)
async function validateUserEmail(email) {
  const errors = [];

  // Format validation
  if (!isValidEmailFormat(email)) {
    errors.push({
      field: 'email',
      message: 'Invalid email format'
    });
  }

  // Async check: uniqueness
  if (errors.length === 0) {
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      errors.push({
        field: 'email',
        message: 'Email already registered',
        code: 'DUPLICATE'
      });
    }
  }

  return errors;
}

// Usage in API
app.post('/api/users', async (req, res) => {
  const emailErrors = await validateUserEmail(req.body.email);
  const otherErrors = validateUserInput(req.body);

  const allErrors = [...emailErrors, ...otherErrors];

  if (allErrors.length > 0) {
    return res.status(422).json({
      error: 'VALIDATION_FAILED',
      validationErrors: allErrors
    });
  }

  const user = await createUser(req.body);
  res.status(201).json(user);
});
```

---

## Retry Strategies

### Exponential Backoff

**CRITICAL**: Implement for transient failures:

```python
import time
import random

def exponential_backoff(func, max_retries=3, base_delay=1):
    """Retry with exponential backoff + jitter"""
    for attempt in range(max_retries):
        try:
            return func()
        except TransientError as err:
            if attempt == max_retries - 1:
                raise  # Last attempt, propagate error

            # Exponential backoff: 1s, 2s, 4s
            delay = base_delay * (2 ** attempt)

            # Add jitter to prevent thundering herd
            jitter = random.uniform(0, delay * 0.1)
            total_delay = delay + jitter

            logger.warning(
                f'Transient error, retrying in {total_delay:.1f}s',
                extra={'attempt': attempt + 1, 'delay': total_delay}
            )

            time.sleep(total_delay)

# Usage
def call_external_api():
    return exponential_backoff(
        lambda: requests.get('https://api.example.com/data'),
        max_retries=3,
        base_delay=1
    )
```

### Circuit Breaker Pattern

**CRITICAL**: Fail fast for cascading failures:

```javascript
class CircuitBreaker {
  constructor(func, options = {}) {
    this.func = func;
    this.failureThreshold = options.failureThreshold || 5;
    this.successThreshold = options.successThreshold || 2;
    this.timeout = options.timeout || 60000;  // 60 seconds

    this.state = 'CLOSED';  // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.successCount = 0;
    this.nextAttempt = Date.now();
  }

  async execute(args) {
    // OPEN: Reject all requests
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      // Try half-open
      this.state = 'HALF_OPEN';
      this.successCount = 0;
    }

    try {
      const result = await this.func(...args);

      // Success
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  onSuccess() {
    this.failureCount = 0;

    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = 'CLOSED';
      }
    }
  }

  onFailure() {
    this.failureCount++;

    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}

// Usage
const apiBreaker = new CircuitBreaker(callExternalAPI, {
  failureThreshold: 5,
  timeout: 60000
});

try {
  await apiBreaker.execute([data]);
} catch (err) {
  // Fallback behavior
  logger.error('API unavailable, using cache');
  return getCachedData();
}
```

### Dead Letter Queue

**MEDIUM**: Handle failed async tasks:

```python
import json
from datetime import datetime

class DeadLetterQueue:
    """Store failed messages for later processing"""
    def __init__(self, redis_client):
        self.redis = redis_client
        self.queue_key = 'dlq:failed_messages'

    def push(self, message, error, context=None):
        """Add failed message to DLQ"""
        dlq_entry = {
            'message': message,
            'error': str(error),
            'timestamp': datetime.now().isoformat(),
            'context': context or {},
            'retries': 0
        }

        self.redis.rpush(self.queue_key, json.dumps(dlq_entry))
        logger.error(
            'Message added to DLQ',
            extra={'message_id': message.get('id'), 'error': str(error)}
        )

    def retry_failed_messages(self, max_retries=3):
        """Periodically retry DLQ messages"""
        while True:
            entry_str = self.redis.lpop(self.queue_key)
            if not entry_str:
                break

            entry = json.loads(entry_str)

            if entry['retries'] >= max_retries:
                # Give up
                logger.error('Message exhausted retries, discarding', extra=entry)
                continue

            try:
                # Retry original operation
                process_message(entry['message'])
                logger.info('Retried message succeeded', extra=entry)
            except Exception as err:
                # Re-queue for next retry
                entry['retries'] += 1
                entry['error'] = str(err)
                self.redis.rpush(self.queue_key, json.dumps(entry))
                logger.warning(f'Retry attempt {entry["retries"]}, re-queuing')
```

---

## Logging Strategy

### Structured Logging

**CRITICAL**: Machine-parseable logs:

```python
import logging
import json
from pythonjsonlogger import jsonlogger

# Configure JSON logging
logger = logging.getLogger(__name__)
logHandler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
logHandler.setFormatter(formatter)
logger.addHandler(logHandler)

# Log with context
logger.error(
    'User creation failed',
    extra={
        'event': 'user_creation_failed',
        'userId': 123,
        'email': 'user@test.com',
        'error_code': 'DUPLICATE_EMAIL',
        'error_message': 'Email already registered',
        'duration_ms': 245
    }
)

# Output: JSON-formatted for ELK/Splunk parsing
# {
#   "timestamp": "2024-01-15T10:30:45.123Z",
#   "level": "ERROR",
#   "message": "User creation failed",
#   "event": "user_creation_failed",
#   "userId": 123,
#   "email": "user@test.com",
#   "error_code": "DUPLICATE_EMAIL"
# }
```

### Correlation IDs

**CRITICAL**: Trace requests across services:

```javascript
import { v4 as uuid } from 'uuid';

// Express middleware: Add correlation ID
app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || uuid();
  res.set('X-Correlation-ID', req.correlationId);

  // Add to all logs in this request
  req.log = logger.child({ correlationId: req.correlationId });

  next();
});

// Usage
app.post('/api/users', async (req, res) => {
  req.log.info('Creating user', { email: req.body.email });

  try {
    const user = await createUser(req.body);

    // Forward correlation ID to downstream services
    const profile = await fetchUserProfile(user.id, {
      headers: { 'X-Correlation-ID': req.correlationId }
    });

    req.log.info('User created successfully', { userId: user.id });
    res.status(201).json(user);
  } catch (err) {
    req.log.error('User creation failed', {
      error: err.message,
      stack: err.stack
    });
    res.status(500).json({ error: 'Failed to create user' });
  }
});
```

### PII Redaction

**CRITICAL**: Don't log sensitive data:

```python
import re

def redact_pii(text):
    """Remove sensitive information from logs"""
    patterns = {
        'email': r'[\w\.-]+@[\w\.-]+\.\w+',
        'phone': r'\d{3}-\d{3}-\d{4}',
        'ssn': r'\d{3}-\d{2}-\d{4}',
        'credit_card': r'\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}',
        'api_key': r'api[_-]?key[=:][\w-]+'
    }

    redacted = text
    for pattern_name, pattern in patterns.items():
        redacted = re.sub(pattern, f'[REDACTED_{pattern_name.upper()}]', redacted)

    return redacted

# Usage in logging
logger.error(
    'Payment processing failed',
    extra={
        'message': redact_pii(f'Card {card_number} failed: {error}'),
        'user_email': '[REDACTED_EMAIL]'
    }
)
```

---

## Error Boundaries

### React Error Boundaries

**CRITICAL**: Catch and recover from render errors:

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    logger.error('React error boundary caught error', {
      error: error.toString(),
      componentStack: errorInfo.componentStack
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h1>Something went wrong</h1>
          <p>Please try refreshing the page</p>
          <button onClick={() => window.location.reload()}>Refresh</button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <Dashboard />
</ErrorBoundary>
```

### Express Global Handler

**CRITICAL**: Catch unhandled errors:

```javascript
// Express error handler (must be last)
app.use((err, req, res, next) => {
  logger.error('Unhandled error', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });

  res.status(500).json({
    error: 'INTERNAL_ERROR',
    message: 'An unexpected error occurred'
  });
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  logger.error('Unhandled promise rejection', {
    reason: reason.toString(),
    promise
  });
});

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  logger.fatal('Uncaught exception, exiting', { error: err.message });
  process.exit(1);
});
```

---

## Graceful Degradation

### Feature Flags

**HIGH**: Toggle features during failures:

```python
from functools import wraps

class FeatureFlags:
    def __init__(self, config):
        self.config = config

    def is_enabled(self, feature_name, user_id=None):
        """Check if feature is enabled"""
        enabled = self.config.get(feature_name, {}).get('enabled', False)

        # Per-user rollout
        if 'rollout_percentage' in self.config.get(feature_name, {}):
            rollout = self.config[feature_name]['rollout_percentage']
            user_hash = hash(str(user_id)) % 100
            return enabled and (user_hash < rollout)

        return enabled

# Usage with fallback
ff = FeatureFlags(config)

def get_user_recommendations(user_id):
    """Get recommendations, with fallback if ML service fails"""
    if not ff.is_enabled('ml_recommendations', user_id):
        return []  # Feature disabled, return empty

    try:
        return fetch_ml_recommendations(user_id)
    except Exception as err:
        logger.error('ML recommendation failed', extra={'user_id': user_id})

        # Disable feature if repeatedly failing
        if err.error_code == 'SERVICE_UNAVAILABLE':
            ff.disable('ml_recommendations')

        # Graceful fallback
        return get_simple_recommendations(user_id)
```

### Fallback Strategies

**MEDIUM**: Degrade gracefully:

```javascript
// Example: Fallback for real-time features
async function getUserOnlineStatus(userId) {
  try {
    // Try real-time status service
    return await realtimeService.getOnlineStatus(userId);
  } catch (err) {
    logger.warn('Real-time status unavailable, using cache', { userId });

    // Fallback 1: Use cached status
    const cachedStatus = await cache.get(`user:${userId}:status`);
    if (cachedStatus) {
      return { ...cachedStatus, isCached: true };
    }

    // Fallback 2: Use database
    const user = await db.users.findOne({ id: userId });
    return {
      userId,
      isOnline: false,
      lastSeen: user.lastSeen,
      isFallback: true
    };
  }
}
```

---

## User-Facing Errors

### Internationalization

**HIGH**: Localized error messages:

```javascript
// Error messages by language and code
const errorMessages = {
  en: {
    'VALIDATION_FAILED': 'Please check your input and try again',
    'USER_NOT_FOUND': 'User not found',
    'DUPLICATE_EMAIL': 'This email is already registered',
    'INTERNAL_ERROR': 'Something went wrong. Please try again later'
  },
  es: {
    'VALIDATION_FAILED': 'Por favor verifica tu entrada e intenta de nuevo',
    'USER_NOT_FOUND': 'Usuario no encontrado',
    'DUPLICATE_EMAIL': 'Este email ya está registrado',
    'INTERNAL_ERROR': 'Algo salió mal. Por favor intenta más tarde'
  }
};

function getUserFacingMessage(error, language = 'en') {
  return errorMessages[language]?.[error.code]
    || errorMessages[language]['INTERNAL_ERROR']
    || 'An error occurred';
}
```

### User-Friendly Error Display

**MEDIUM**: Avoid technical jargon:

```javascript
// ✗ WRONG: Technical error
"TypeError: Cannot read property 'email' of undefined"

// ✓ CORRECT: User-friendly error
"We couldn't load your profile. Please try refreshing the page."

// Implementation
function handleUserError(error) {
  const userMessage = {
    'NETWORK_ERROR': 'Connection failed. Check your internet.',
    'TIMEOUT': 'Request took too long. Please try again.',
    'AUTH_FAILED': 'Please log in again',
    'PERMISSION_DENIED': 'You do not have permission to do this'
  };

  return {
    title: 'Error',
    message: userMessage[error.code] || 'Something went wrong',
    action: error.retryable ? 'Retry' : 'Dismiss'
  };
}
```

---

## Summary: Error Handling Checklist

- [ ] Error hierarchy defined (domain, infrastructure, validation)
- [ ] Errors have codes, not just messages
- [ ] Error chains preserved (cause)
- [ ] HTTP status codes semantic
- [ ] Validation errors field-level with codes
- [ ] Cross-field validation implemented
- [ ] Retry logic with exponential backoff
- [ ] Circuit breaker for external services
- [ ] Dead letter queue for failed messages
- [ ] Structured logging (JSON)
- [ ] Correlation IDs for request tracing
- [ ] PII redaction in logs
- [ ] Error boundaries in React
- [ ] Global error handlers (Express)
- [ ] Feature flags for graceful degradation
- [ ] Fallback strategies implemented
- [ ] Error messages i18n-ready
- [ ] User-facing messages clear and friendly
