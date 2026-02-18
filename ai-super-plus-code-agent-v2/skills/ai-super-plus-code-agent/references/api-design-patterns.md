# API Design Patterns - Expert Reference

## Table of Contents

1. [REST API Design](#rest-api-design)
2. [GraphQL Patterns](#graphql-patterns)
3. [gRPC Patterns](#grpc-patterns)
4. [Authentication Strategies](#authentication-strategies)
5. [Rate Limiting](#rate-limiting)
6. [Pagination](#pagination)
7. [Versioning](#versioning)
8. [Error Handling](#error-handling)
9. [OpenAPI Specification](#openapi-specification)
10. [Request/Response Schemas](#requestresponse-schemas)

---

## REST API Design

### Resource-Oriented Design

**CRITICAL**: Resources as nouns, operations as HTTP verbs:

```
✓ CORRECT: Resource-oriented
POST   /api/users                    # Create user
GET    /api/users                    # List users
GET    /api/users/{id}               # Get user
PUT    /api/users/{id}               # Replace user
PATCH  /api/users/{id}               # Update user
DELETE /api/users/{id}               # Delete user

✗ WRONG: Action-oriented (RPC-style)
POST   /api/createUser
POST   /api/getUser
POST   /api/updateUser
POST   /api/deleteUser
```

**HIGH**: Resource hierarchy for related data:

```
# Correct hierarchy (max 2 levels deep)
GET    /api/users/{userId}/orders                # List user's orders
GET    /api/users/{userId}/orders/{orderId}      # Get specific order
POST   /api/users/{userId}/orders                # Create order for user

# ✗ WRONG: Too deep
GET    /api/users/{userId}/orders/{orderId}/items/{itemId}/details/...
```

### HTTP Status Codes

**CRITICAL**: Correct status code for each scenario:

```
2xx Success:
  200 OK           - Request succeeded, response contains data
  201 Created      - Resource created (include Location header)
  202 Accepted     - Async operation started (include progress endpoint)
  204 No Content   - Success, but no response body

3xx Redirection:
  301 Moved Permanently - Permanent URL change
  304 Not Modified      - Client cache is current (use ETags)
  307 Temporary Redirect - Temporary move (preserve method)

4xx Client Error:
  400 Bad Request       - Invalid request format
  401 Unauthorized      - Authentication required
  403 Forbidden         - Authenticated but not authorized
  404 Not Found         - Resource doesn't exist
  409 Conflict          - Request conflicts with state (e.g., duplicate email)
  422 Unprocessable Entity - Validation failed
  429 Too Many Requests - Rate limited

5xx Server Error:
  500 Internal Server Error - Unexpected error
  503 Service Unavailable   - Maintenance or overload
```

**HIGH**: Implementation:

```javascript
// ✓ CORRECT: Semantic status codes
app.post('/api/users', (req, res) => {
  if (!isValidEmail(req.body.email)) {
    return res.status(400).json({
      error: 'Invalid request',
      message: 'Email must be valid'
    });
  }

  if (await userExists(req.body.email)) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already registered'
    });
  }

  const user = await createUser(req.body);
  res.status(201)
    .set('Location', `/api/users/${user.id}`)
    .json(user);
});
```

### HATEOAS (Hypermedia)

**MEDIUM**: Include links for discoverability:

```javascript
// Response with HATEOAS links
{
  "id": 123,
  "email": "user@test.com",
  "name": "John Doe",
  "_links": {
    "self": {
      "href": "/api/users/123"
    },
    "all_users": {
      "href": "/api/users"
    },
    "update": {
      "href": "/api/users/123",
      "method": "PATCH"
    },
    "delete": {
      "href": "/api/users/123",
      "method": "DELETE"
    },
    "orders": {
      "href": "/api/users/123/orders"
    }
  }
}
```

### Content Negotiation

**HIGH**: Support multiple formats:

```javascript
// Client specifies desired format
GET /api/users/123
Accept: application/json

GET /api/users/123
Accept: application/xml

GET /api/users/123
Accept: text/csv

// Server responds with appropriate format
app.get('/api/users/:id', (req, res) => {
  const user = getUser(req.params.id);

  switch (req.accepts(['json', 'xml', 'csv'])) {
    case 'json':
      res.json(user);
      break;
    case 'xml':
      res.type('xml').send(convertToXml(user));
      break;
    case 'csv':
      res.type('csv').send(convertToCsv(user));
      break;
    default:
      res.status(406).send('Not Acceptable');
  }
});
```

---

## GraphQL Patterns

### Schema Design

**CRITICAL**: Clear, well-typed schema:

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  role: Role!
  createdAt: DateTime!
  orders: [Order!]! @relationship(name: "user_orders")
}

type Order {
  id: ID!
  user: User!
  items: [OrderItem!]!
  total: Float!
  status: OrderStatus!
  createdAt: DateTime!
}

enum Role {
  ADMIN
  USER
  GUEST
}

enum OrderStatus {
  PENDING
  PROCESSING
  COMPLETED
  CANCELLED
}

type Query {
  user(id: ID!): User
  users(first: Int!, after: String): UserConnection!
  searchProducts(query: String!): [Product!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

type Subscription {
  orderStatusChanged(userId: ID!): Order!
  userActivityUpdated(userId: ID!): Activity!
}
```

### Resolver Implementation

**HIGH**: Efficient resolvers with caching:

```javascript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      // Check auth
      if (!context.user) throw new AuthenticationError('Not authenticated');

      // Use dataloader for N+1 prevention
      return context.loaders.userById.load(id);
    },

    users: async (_, { first, after }, context) => {
      const pageInfo = parseCursor(after, first);
      const users = await User.find()
        .skip(pageInfo.offset)
        .limit(first);

      return {
        edges: users.map(user => ({
          node: user,
          cursor: encodeCursor(user.id)
        })),
        pageInfo: {
          hasNextPage: users.length === first,
          endCursor: users[users.length - 1]?.id
        }
      };
    }
  },

  User: {
    orders: async (user, _, context) => {
      // Use dataloader for efficiency
      return context.loaders.ordersByUserId.load(user.id);
    }
  },

  Mutation: {
    createUser: async (_, { input }, context) => {
      // Validate input
      const errors = validateUserInput(input);
      if (errors.length) {
        throw new ValidationError(errors);
      }

      // Check auth
      if (!context.user?.role === 'ADMIN') {
        throw new ForbiddenError('Only admins can create users');
      }

      const user = await User.create(input);
      return user;
    }
  }
};
```

### DataLoader Pattern

**CRITICAL**: Prevent N+1 queries:

```javascript
import DataLoader from 'dataloader';

// Batch load users by ID
const userLoaderByIds = new DataLoader(async (userIds) => {
  const users = await User.find({ _id: { $in: userIds } });

  // Return in same order as requested
  return userIds.map(id => users.find(u => u._id === id));
});

// Usage in context
const context = {
  loaders: {
    userById: userLoaderByIds,
    ordersByUserId: new DataLoader(async (userIds) => {
      const orders = await Order.find({ userId: { $in: userIds } });
      return userIds.map(id => orders.filter(o => o.userId === id));
    })
  }
};

// Resolvers use loaders
const resolvers = {
  Order: {
    user: (order, _, context) => {
      return context.loaders.userById.load(order.userId);
    }
  }
};
```

### Subscriptions

**MEDIUM**: Real-time data:

```javascript
const resolvers = {
  Subscription: {
    orderStatusChanged: {
      subscribe: (_, { userId }, context) => {
        // Verify auth
        if (context.user.id !== userId && context.user.role !== 'ADMIN') {
          throw new ForbiddenError('Cannot subscribe to others orders');
        }

        return pubsub.asyncIterator([`ORDER_${userId}_STATUS_CHANGED`]);
      }
    }
  },

  Mutation: {
    updateOrderStatus: async (_, { id, status }, context) => {
      const order = await Order.findByIdAndUpdate(id, { status });

      // Publish event to subscribers
      pubsub.publish(`ORDER_${order.userId}_STATUS_CHANGED`, {
        orderStatusChanged: order
      });

      return order;
    }
  }
};
```

---

## gRPC Patterns

### Protobuf Schema Design

**CRITICAL**: Well-defined message contracts:

```protobuf
syntax = "proto3";

package api.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
  rpc WatchUser(WatchUserRequest) returns (stream User);
}

message User {
  int32 id = 1;
  string email = 2;
  string name = 3;
  Role role = 4;
  google.protobuf.Timestamp created_at = 5;
}

message GetUserRequest {
  int32 id = 1;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  string next_page_token = 2;
}

enum Role {
  ROLE_UNSPECIFIED = 0;
  ADMIN = 1;
  USER = 2;
  GUEST = 3;
}
```

### Streaming Patterns

**HIGH**: Bidirectional streaming:

```go
// Server streaming
func (s *Server) ListUsers(req *ListUsersRequest, stream UserService_ListUsersServer) error {
  users := s.db.GetUsers(req.PageSize)

  for _, user := range users {
    if err := stream.Send(&user); err != nil {
      return err
    }
  }
  return nil
}

// Client streaming
func (c *Client) BatchCreateUsers(stream UserService_BatchCreateUsersServer) error {
  var results []*User

  for {
    req, err := stream.Recv()
    if err == io.EOF {
      return stream.SendAndClose(&BatchCreateResponse{
        Users: results,
      })
    }
    if err != nil {
      return err
    }

    user, err := c.createUser(req)
    if err != nil {
      return err
    }
    results = append(results, user)
  }
}

// Bidirectional streaming
func (s *Server) SyncUsers(stream UserService_SyncUsersServer) error {
  for {
    req, err := stream.Recv()
    if err == io.EOF {
      return nil
    }
    if err != nil {
      return err
    }

    result, err := s.syncUser(req)
    if err != nil {
      return err
    }

    if err := stream.Send(result); err != nil {
      return err
    }
  }
}
```

### Error Handling in gRPC

**HIGH**: Standard gRPC error codes:

```go
import "google.golang.org/grpc/codes"
import "google.golang.org/grpc/status"

func (s *Server) GetUser(ctx context.Context, req *GetUserRequest) (*User, error) {
  user, err := s.db.GetUser(req.Id)

  if err != nil {
    if err == sql.ErrNoRows {
      return nil, status.Error(codes.NotFound, "User not found")
    }
    return nil, status.Error(codes.Internal, "Database error")
  }

  if !s.hasAccess(ctx, user.Id) {
    return nil, status.Error(codes.PermissionDenied, "Access denied")
  }

  return user, nil
}
```

---

## Authentication Strategies

### JWT (JSON Web Tokens)

**CRITICAL**: Stateless authentication:

```javascript
// Generate JWT
const generateToken = (user) => {
  const payload = {
    sub: user.id,          // Subject (user ID)
    email: user.email,
    role: user.role,
    iat: Math.floor(Date.now() / 1000),
    exp: Math.floor(Date.now() / 1000) + (24 * 60 * 60)  // 24h
  };

  return jwt.sign(payload, process.env.JWT_SECRET, {
    algorithm: 'HS256',
    issuer: 'api.example.com',
    audience: 'app.example.com'
  });
};

// Verify JWT middleware
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token' });
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET, {
      issuer: 'api.example.com',
      audience: 'app.example.com'
    });

    req.user = payload;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Usage
app.use('/api', verifyToken);
app.get('/api/profile', (req, res) => {
  res.json({ user: req.user });
});
```

### OAuth 2.0

**HIGH**: Delegated authorization:

```javascript
// OAuth 2.0 Authorization Code Flow
app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query;

  try {
    // Exchange code for token
    const tokenResponse = await fetch('https://oauth2.googleapis.com/token', {
      method: 'POST',
      body: new URLSearchParams({
        code,
        client_id: process.env.GOOGLE_CLIENT_ID,
        client_secret: process.env.GOOGLE_CLIENT_SECRET,
        redirect_uri: 'http://localhost:3000/auth/google/callback',
        grant_type: 'authorization_code'
      })
    });

    const { access_token } = await tokenResponse.json();

    // Get user info
    const userResponse = await fetch('https://openidconnect.googleapis.com/v1/userinfo', {
      headers: { Authorization: `Bearer ${access_token}` }
    });

    const googleUser = await userResponse.json();

    // Find or create local user
    let user = await User.findByEmail(googleUser.email);
    if (!user) {
      user = await User.create({
        email: googleUser.email,
        name: googleUser.name
      });
    }

    // Issue JWT
    const token = generateToken(user);
    res.redirect(`http://localhost:3000?token=${token}`);
  } catch (err) {
    res.status(500).json({ error: 'Auth failed' });
  }
});
```

### API Keys

**MEDIUM**: Simple API authentication:

```javascript
// Generate API key
const generateApiKey = () => crypto.randomBytes(32).toString('hex');

// Create and store key
const key = {
  key: generateApiKey(),
  userId: 123,
  name: 'Mobile app',
  rate_limit: 1000,  // requests per hour
  created_at: new Date()
};

// Verify API key middleware
const verifyApiKey = async (req, res, next) => {
  const apiKey = req.headers['x-api-key'];

  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }

  const key = await ApiKey.findOne({ key: apiKey });
  if (!key) {
    return res.status(401).json({ error: 'Invalid API key' });
  }

  req.user = await User.findById(key.userId);
  req.apiKey = key;
  next();
};
```

---

## Rate Limiting

### Token Bucket Algorithm

**CRITICAL**: Distribute request allowance:

```javascript
// Token bucket implementation
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity;
    this.refillRate = refillRate;    // tokens per second
    this.tokens = capacity;
    this.lastRefillTime = Date.now();
  }

  // Refill tokens based on time elapsed
  refill() {
    const now = Date.now();
    const elapsedSeconds = (now - this.lastRefillTime) / 1000;
    const tokensToAdd = elapsedSeconds * this.refillRate;

    this.tokens = Math.min(this.capacity, this.tokens + tokensToAdd);
    this.lastRefillTime = now;
  }

  // Consume tokens
  consume(tokens = 1) {
    this.refill();
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    return false;
  }
}

// Middleware
const rateLimitMiddleware = (req, res, next) => {
  const userId = req.user?.id || req.ip;

  // Get or create bucket for user
  let bucket = bucketCache.get(userId);
  if (!bucket) {
    bucket = new TokenBucket(100, 10);  // 100 requests, refill at 10/sec
    bucketCache.set(userId, bucket);
  }

  if (!bucket.consume()) {
    return res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: Math.ceil(100 / 10)  // seconds until capacity
    });
  }

  res.set('X-RateLimit-Limit', 100);
  res.set('X-RateLimit-Remaining', Math.floor(bucket.tokens));
  next();
};
```

### Sliding Window Counter

**HIGH**: Per-endpoint rate limiting:

```javascript
// Sliding window counter (alternative algorithm)
class SlidingWindowCounter {
  constructor(limit, windowSeconds) {
    this.limit = limit;
    this.windowSeconds = windowSeconds;
    this.requests = [];
  }

  addRequest() {
    const now = Date.now();
    // Remove old requests outside window
    this.requests = this.requests.filter(
      time => now - time < this.windowSeconds * 1000
    );

    this.requests.push(now);
  }

  isAllowed() {
    if (this.requests.length < this.limit) {
      this.addRequest();
      return true;
    }
    return false;
  }
}

// Per-endpoint limiting
const endpointLimits = {
  'POST /api/users': new SlidingWindowCounter(10, 60),    // 10 creates per minute
  'GET /api/users': new SlidingWindowCounter(1000, 60),   // 1000 reads per minute
  'POST /api/auth/login': new SlidingWindowCounter(5, 300) // 5 login attempts per 5 min
};
```

---

## Pagination

### Cursor-Based Pagination

**CRITICAL**: Efficient for large datasets:

```javascript
// Cursor: base64-encoded offset
const encodeCursor = (id) => Buffer.from(id.toString()).toString('base64');
const decodeCursor = (cursor) => parseInt(Buffer.from(cursor, 'base64').toString());

app.get('/api/users', async (req, res) => {
  const limit = Math.min(req.query.limit || 20, 100);
  const after = req.query.after ? decodeCursor(req.query.after) : 0;

  // Query: get limit+1 to detect if more results exist
  const users = await User.find()
    .where('id').gt(after)
    .limit(limit + 1)
    .sort({ id: 1 });

  const hasMore = users.length > limit;
  const edges = users.slice(0, limit);

  res.json({
    data: edges,
    pageInfo: {
      hasNextPage: hasMore,
      endCursor: edges.length > 0 ? encodeCursor(edges[edges.length - 1].id) : null
    }
  });
});

// Client usage
GET /api/users?limit=20
# Response
{
  data: [...20 users...],
  pageInfo: {
    hasNextPage: true,
    endCursor: "MTAw"  // Encode ID 100
  }
}

# Next request
GET /api/users?limit=20&after=MTAw
```

### Keyset Pagination

**HIGH**: Efficient for ordering:

```javascript
// Keyset pagination (more efficient than cursor for complex sorts)
app.get('/api/orders', async (req, res) => {
  const limit = 20;
  const { lastId, lastCreatedAt } = req.query;

  let query = Order.find();

  // Build keyset conditions for efficient pagination
  if (lastId && lastCreatedAt) {
    query = query.where('createdAt').lt(new Date(lastCreatedAt))
      .or([
        { createdAt: new Date(lastCreatedAt), id: { $lt: lastId } }
      ]);
  }

  const orders = await query
    .sort({ createdAt: -1, id: -1 })
    .limit(limit + 1);

  const hasMore = orders.length > limit;
  const edges = orders.slice(0, limit);

  res.json({
    data: edges,
    pageInfo: {
      hasMore,
      lastId: edges[edges.length - 1]?.id,
      lastCreatedAt: edges[edges.length - 1]?.createdAt
    }
  });
});
```

---

## Versioning

### URL Path Versioning

**CRITICAL**: Clear version in path:

```
GET /api/v1/users
GET /api/v2/users
GET /api/v3/users
```

```javascript
// Express routing
app.use('/api/v1', require('./routes/v1'));
app.use('/api/v2', require('./routes/v2'));
app.use('/api/v3', require('./routes/v3'));

// Migration path
// V1: { id, email, name }
// V2: Added { role, createdAt }
// V3: Removed { email }, added { emails: [{ value, primary }] }
```

### Header-Based Versioning

**HIGH**: Version negotiation header:

```javascript
// Client specifies version
GET /api/users
API-Version: 2

// Server routes based on version
app.get('/api/users', (req, res) => {
  const version = req.headers['api-version'] || '1';

  if (version === '1') {
    return res.json(v1Format(users));
  }
  if (version === '2') {
    return res.json(v2Format(users));
  }

  res.status(400).json({ error: 'Unsupported API version' });
});
```

---

## Error Handling

### RFC 7807 Problem Details

**CRITICAL**: Standardized error format:

```javascript
// RFC 7807 error response
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 422,
  "detail": "Request body validation failed",
  "instance": "/api/users",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format",
      "value": "invalid-email"
    },
    {
      "field": "age",
      "message": "Must be greater than 18",
      "value": 16
    }
  ]
}

// Implementation
app.post('/api/users', async (req, res) => {
  const validation = validateUserInput(req.body);

  if (!validation.isValid) {
    return res.status(422).json({
      type: 'https://api.example.com/errors/validation',
      title: 'Validation Failed',
      status: 422,
      detail: 'Request validation failed',
      instance: req.path,
      errors: validation.errors
    });
  }

  // Process request
});
```

### Error Codes and Retry Logic

**HIGH**: Specific error codes for client retry logic:

```javascript
const errors = {
  VALIDATION_FAILED: { code: 'VALIDATION_FAILED', status: 422, retryable: false },
  AUTHENTICATION_FAILED: { code: 'AUTHENTICATION_FAILED', status: 401, retryable: false },
  PERMISSION_DENIED: { code: 'PERMISSION_DENIED', status: 403, retryable: false },
  RESOURCE_NOT_FOUND: { code: 'RESOURCE_NOT_FOUND', status: 404, retryable: false },
  CONFLICT: { code: 'CONFLICT', status: 409, retryable: true },
  RATE_LIMIT_EXCEEDED: { code: 'RATE_LIMIT_EXCEEDED', status: 429, retryable: true },
  SERVICE_UNAVAILABLE: { code: 'SERVICE_UNAVAILABLE', status: 503, retryable: true },
  INTERNAL_ERROR: { code: 'INTERNAL_ERROR', status: 500, retryable: true }
};

// Client retry logic
async function makeRequestWithRetry(url, options, maxRetries = 3) {
  let lastError;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      const data = await response.json();

      if (!response.ok) {
        const error = errors[data.code];
        if (!error?.retryable) {
          throw new Error(data.detail);
        }
      }

      return data;
    } catch (err) {
      lastError = err;

      if (attempt < maxRetries - 1) {
        // Exponential backoff
        const delayMs = Math.pow(2, attempt) * 1000;
        await new Promise(resolve => setTimeout(resolve, delayMs));
      }
    }
  }

  throw lastError;
}
```

---

## OpenAPI Specification

### Annotation-Based Generation

**HIGH**: Auto-generate from code annotations:

```javascript
/**
 * @swagger
 * /api/users:
 *   post:
 *     summary: Create a new user
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateUserInput'
 *     responses:
 *       201:
 *         description: User created
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       400:
 *         description: Validation failed
 *       409:
 *         description: Email already exists
 */
app.post('/api/users', async (req, res) => {
  // Implementation
});

// Generate swagger.json
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerSpec = swaggerJsdoc({
  definition: {
    openapi: '3.0.0',
    info: { title: 'API', version: '1.0.0' },
    servers: [{ url: 'http://localhost:3000' }]
  },
  apis: ['./routes/**/*.js']
});
```

### Schema Definition

**CRITICAL**: Reusable schema components:

```yaml
components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - name
      properties:
        id:
          type: integer
          format: int64
        email:
          type: string
          format: email
        name:
          type: string
        role:
          type: string
          enum: [admin, user, guest]
        createdAt:
          type: string
          format: date-time

    CreateUserInput:
      type: object
      required:
        - email
        - name
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 1
          maxLength: 255

    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object
```

---

## Request/Response Schemas

### Input Validation

**CRITICAL**: Validate at API boundary:

```javascript
// Validation middleware
const validateSchema = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,      // Report all errors
      stripUnknown: true      // Remove extra fields
    });

    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details.map(e => ({
          field: e.path.join('.'),
          message: e.message,
          type: e.type
        }))
      });
    }

    req.validated = value;
    next();
  };
};

// Define schemas
const Joi = require('joi');

const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  name: Joi.string().min(1).max(255).required(),
  password: Joi.string().min(8).required()
});

// Usage
app.post('/api/users', validateSchema(createUserSchema), (req, res) => {
  const { email, name, password } = req.validated;
  // Process validated input
});
```

### Response Transformation

**HIGH**: Consistent response envelope:

```javascript
// Response wrapper
class ApiResponse {
  static success(data, meta = {}) {
    return {
      success: true,
      data,
      meta: {
        timestamp: new Date().toISOString(),
        ...meta
      }
    };
  }

  static error(error, statusCode = 500) {
    return {
      success: false,
      error: {
        code: error.code || 'INTERNAL_ERROR',
        message: error.message,
        details: error.details || {}
      },
      meta: {
        timestamp: new Date().toISOString()
      }
    };
  }

  static paginated(items, pagination) {
    return {
      success: true,
      data: items,
      pagination: {
        total: pagination.total,
        limit: pagination.limit,
        offset: pagination.offset,
        hasMore: pagination.offset + pagination.limit < pagination.total
      },
      meta: {
        timestamp: new Date().toISOString()
      }
    };
  }
}

// Usage
app.get('/api/users', async (req, res) => {
  const users = await User.find().limit(20);
  res.json(ApiResponse.paginated(users, {
    total: await User.countDocuments(),
    limit: 20,
    offset: 0
  }));
});
```

---

## Summary: API Design Checklist

- [ ] Resources use nouns (not verbs)
- [ ] HTTP verbs (GET, POST, PUT, DELETE) used correctly
- [ ] Status codes semantic (201 for create, 409 for conflict, etc.)
- [ ] HATEOAS links for discoverability
- [ ] Content negotiation supported
- [ ] Authentication (JWT/OAuth/API key) implemented
- [ ] Rate limiting enforced
- [ ] Pagination cursor-based (for efficiency)
- [ ] Versioning strategy clear (URL/header)
- [ ] Error format RFC 7807 compliant
- [ ] OpenAPI spec generated/maintained
- [ ] Input validation at API boundary
- [ ] Response envelope consistent
- [ ] GraphQL uses DataLoader for N+1 prevention
- [ ] gRPC uses proper error codes
