# Security Hardening — OWASP Top 10 + Production Checklist

## OWASP Top 10 (2021) Implementation Guide

### A01:2021 – Broken Access Control

**What**: Users can perform unauthorized actions or access data they shouldn't.

**Prevention Implementation**:
```typescript
// Middleware for role-based access control
export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user as AuthenticatedUser;

    if (!roles.includes(user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
};

// Usage
app.delete('/api/posts/:id', authenticate, authorize('admin', 'moderator'), deletPostHandler);

// Resource-level authorization
export const canEditPost = async (userId: string, postId: string): Promise<boolean> => {
  const post = await db.post.findUnique({
    where: { id: postId },
    select: { userId: true }
  });

  return post?.userId === userId;
};

app.patch('/api/posts/:id', authenticate, async (req, res) => {
  if (!(await canEditPost(req.user.id, req.params.id))) {
    return res.status(403).json({ error: 'You can only edit your own posts' });
  }

  // Update logic
});
```

---

### A02:2021 – Cryptographic Failures

**What**: Sensitive data is exposed due to weak encryption or transmission.

**Prevention Implementation**:
```typescript
// Password hashing
import bcrypt from 'bcrypt';

export const hashPassword = async (password: string): Promise<string> => {
  const salt = await bcrypt.genSalt(12);
  return bcrypt.hash(password, salt);
};

export const verifyPassword = async (password: string, hash: string): Promise<boolean> => {
  return bcrypt.compare(password, hash);
};

// Sensitive data encryption
import crypto from 'crypto';

const ENCRYPTION_KEY = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex');

export const encrypt = (text: string): string => {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', ENCRYPTION_KEY, iv);
  let encrypted = cipher.update(text);
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  return iv.toString('hex') + ':' + encrypted.toString('hex');
};

export const decrypt = (text: string): string => {
  const parts = text.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encrypted = Buffer.from(parts[1], 'hex');
  const decipher = crypto.createDecipheriv('aes-256-cbc', ENCRYPTION_KEY, iv);
  let decrypted = decipher.update(encrypted);
  decrypted = Buffer.concat([decrypted, decipher.final()]);
  return decrypted.toString();
};

// HTTPS only (in production)
app.use(helmet.hsts({ maxAge: 31536000, includeSubDomains: true }));
```

---

### A03:2021 – Injection

**What**: User input is used in commands/queries without validation.

**Prevention Implementation**:
```typescript
// Use parameterized queries (Prisma handles this)
const post = await prisma.post.findUnique({
  where: { id: userId } // Parameterized, never vulnerable
});

// Input validation with Zod
import { z } from 'zod';

const CreatePostSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1),
  published: z.boolean().default(false)
});

app.post('/api/posts', async (req, res) => {
  try {
    const data = CreatePostSchema.parse(req.body);
    // Validated input - safe to use
  } catch (error) {
    res.status(400).json({ error: 'Invalid input' });
  }
});

// Output encoding (XSS prevention in React)
// React auto-escapes by default
export const PostCard = ({ title, content }: Post) => {
  return (
    <div>
      <h2>{title}</h2> {/* Automatically escaped */}
      <p>{content}</p> {/* Automatically escaped */}
    </div>
  );
};

// If using dangerouslySetInnerHTML, sanitize first
import DOMPurify from 'dompurify';

export const RichPostCard = ({ title, htmlContent }: Post) => {
  return (
    <div>
      <h2>{title}</h2>
      <div dangerouslySetInnerHTML={{
        __html: DOMPurify.sanitize(htmlContent)
      }} />
    </div>
  );
};
```

---

### A04:2021 – Insecure Design

**What**: Missing security controls in design phase.

**Prevention**:
```typescript
// Rate limiting (prevent brute force)
import rateLimit from 'express-rate-limit';

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false
});

app.post('/api/auth/login', loginLimiter, loginHandler);

// Account lockout after failed attempts
let failedAttempts = new Map<string, { count: number; until: Date }>();

export const trackFailedLogin = (email: string) => {
  const attempt = failedAttempts.get(email);
  const now = new Date();

  if (attempt && attempt.until > now) {
    throw new Error('Account temporarily locked');
  }

  const count = (attempt?.count || 0) + 1;
  if (count >= 5) {
    failedAttempts.set(email, {
      count: 5,
      until: new Date(now.getTime() + 15 * 60 * 1000)
    });
    throw new Error('Account locked for 15 minutes');
  }

  failedAttempts.set(email, { count, until: now });
};

export const clearFailedLogins = (email: string) => {
  failedAttempts.delete(email);
};
```

---

### A05:2021 – Security Misconfiguration

**What**: Default settings, incomplete configs, or exposed debug features.

**Prevention**:
```typescript
// Remove default headers (helmet)
import helmet from 'helmet';

app.use(helmet());

// Set security headers
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"], // Minimize unsafe-inline
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"]
  }
}));

// CORS configuration
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(','),
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

// No debug mode in production
if (process.env.NODE_ENV !== 'production') {
  app.use(express.json({ limit: '10mb' }));
} else {
  app.use(express.json({ limit: '1mb' }));
}

// Error handling (no stack traces in production)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error('Error:', err);

  const status = err instanceof ValidationError ? 400 : 500;
  const message = process.env.NODE_ENV === 'production'
    ? 'Internal server error'
    : err.message;

  res.status(status).json({ error: message });
});

// Dependency scanning
// Run: npm audit, pip audit, cargo audit
```

---

### A06:2021 – Vulnerable & Outdated Components

**What**: Using libraries with known vulnerabilities.

**Prevention**:
```json
// package.json
{
  "scripts": {
    "audit": "npm audit --production",
    "audit:fix": "npm audit fix --production"
  },
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=10.0.0"
  }
}
```

**CI/CD Check**:
```yaml
# .github/workflows/security.yml
name: Security Audit
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm audit --audit-level=moderate
      - run: npm run test:security
```

---

### A07:2021 – Identification and Authentication Failures

**What**: Weak authentication, session management, or password policies.

**Prevention**:
```typescript
// Strong password requirements
const PASSWORD_REQUIREMENTS = z.object({
  password: z.string()
    .min(12, 'Password must be at least 12 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[!@#$%^&*]/, 'Must contain special character')
});

// Multi-factor authentication
export const generateTOTP = (secret: string): string => {
  const totp = new TOTP({ secret });
  return totp.generate();
};

export const verifyTOTP = (secret: string, token: string): boolean => {
  const totp = new TOTP({ secret });
  return totp.check(token, secret);
};

// Session management with short expiration
export const createSession = (userId: string): string => {
  const token = jwt.sign(
    { userId, iat: Date.now() },
    process.env.JWT_SECRET!,
    { expiresIn: '15m' } // Short expiration
  );
  return token;
};

// Refresh token with longer expiration (stored in secure cookie)
export const createRefreshToken = (userId: string): string => {
  const token = jwt.sign(
    { userId, type: 'refresh' },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: '7d' }
  );
  return token;
};

// Logout clears session
export const logoutHandler = (req: Request, res: Response) => {
  res.clearCookie('refreshToken');
  req.logout((err) => {
    if (err) return res.status(500).json({ error: 'Logout failed' });
    res.json({ success: true });
  });
};
```

---

### A08:2021 – Software and Data Integrity Failures

**What**: Unsafe CI/CD, unsigned dependencies, or insecure updates.

**Prevention**:
```typescript
// Verify package integrity
// Use npm with audit-level
// Lock dependencies (package-lock.json)

// Code signing in CI/CD
// .github/workflows/deploy.yml
// - Sign commits
// - Verify signatures before deploy
```

---

### A09:2021 – Logging and Monitoring Failures

**What**: Insufficient logging or no alerting on suspicious activity.

**Prevention**:
```typescript
// Structured logging with Pino
import pino from 'pino';

const logger = pino({
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      singleLine: process.env.NODE_ENV === 'production'
    }
  }
});

// Never log sensitive data
logger.info({
  event: 'user_login',
  userId: user.id, // OK
  // NO: password, ssn, credit_card, etc.
});

// HTTP request logging
import pinoHttp from 'pino-http';
app.use(pinoHttp({ logger }));

// Alert on suspicious activity
export const monitorSuspiciousActivity = () => {
  // Multiple failed logins
  // Unusual API usage
  // Data access anomalies
  // Permission changes

  logger.warn({
    event: 'suspicious_activity',
    userId: user.id,
    action: 'multiple_failed_logins',
    count: 5,
    alert: true
  });
};
```

---

### A10:2021 – Server-Side Request Forgery (SSRF)

**What**: Application fetches remote resources without proper validation.

**Prevention**:
```typescript
// Whitelist allowed domains
const ALLOWED_DOMAINS = [
  'api.github.com',
  'api.twitter.com',
  'cdn.example.com'
];

export const fetchExternalResource = async (url: string): Promise<unknown> => {
  const parsedUrl = new URL(url);

  // Whitelist check
  if (!ALLOWED_DOMAINS.includes(parsedUrl.hostname)) {
    throw new Error('Domain not allowed');
  }

  // Prevent access to internal IPs
  const ip = require('ip');
  if (ip.isPrivate(parsedUrl.hostname)) {
    throw new Error('Cannot access private IPs');
  }

  // Timeout to prevent slowloris attacks
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 5000);

  try {
    const response = await fetch(url, {
      signal: controller.signal,
      headers: { 'User-Agent': 'MyApp/1.0' }
    });
    return response.json();
  } finally {
    clearTimeout(timeout);
  }
};
```

---

## Production Security Checklist

```
AUTHENTICATION & AUTHORIZATION
☐ Multi-factor authentication enabled
☐ Password requirements enforced (12+ chars, special chars)
☐ Session timeout implemented (15 min for sensitive ops)
☐ Refresh token mechanism in place
☐ API keys rotated regularly
☐ OAuth2/OIDC properly configured
☐ Role-based access control (RBAC) implemented
☐ Resource-level authorization checks

DATA SECURITY
☐ Passwords hashed with bcrypt/Argon2
☐ Sensitive data encrypted at rest
☐ Encrypted at transit (TLS 1.2+)
☐ No PII in logs
☐ No hardcoded secrets or API keys
☐ Database credentials in environment variables
☐ Automated data backups
☐ Backup encryption enabled

API SECURITY
☐ Input validation on all endpoints (Zod/Joi)
☐ Output encoding (XSS prevention)
☐ Rate limiting per endpoint
☐ CSRF tokens on state-changing requests
☐ CORS properly configured
☐ API versioning implemented
☐ API authentication required
☐ Request size limits enforced

DEPENDENCIES
☐ npm audit clean (no moderate/high/critical)
☐ Dependencies locked (package-lock.json)
☐ Dependencies updated monthly
☐ Known vulnerabilities checked

INFRASTRUCTURE
☐ Firewall rules configured
☐ Security groups restricted
☐ DDoS protection enabled
☐ WAF (Web Application Firewall) enabled
☐ SSL/TLS certificates valid
☐ Security headers set (CSP, HSTS, X-Frame-Options)

MONITORING & LOGGING
☐ Structured logging configured
☐ Log retention policy set
☐ Error tracking enabled (Sentry)
☐ Performance monitoring active
☐ Uptime monitoring configured
☐ Security event alerts configured
☐ No sensitive data in logs

DEPLOYMENT
☐ Secrets manager configured (AWS Secrets, HashiCorp Vault)
☐ Environment-specific configs
☐ No secrets in code repositories
☐ Signed commits required
☐ CI/CD security scanning enabled
☐ Deployment approval process
☐ Automated rollback configured
```

---

## Security Testing

```typescript
// Security test suite
describe('Security', () => {
  it('should not allow XSS injection', async () => {
    const xssPayload = '<script>alert("XSS")</script>';
    const response = await request(app)
      .post('/api/posts')
      .send({ title: xssPayload, content: 'test' });

    expect(response.body.title).toBe(xssPayload); // Escaped in response
  });

  it('should prevent SQL injection', async () => {
    const sqlPayload = "'; DROP TABLE users; --";
    const response = await request(app)
      .get(`/api/posts/${sqlPayload}`);

    expect(response.status).toBe(400);
  });

  it('should enforce RBAC', async () => {
    const response = await request(app)
      .delete('/api/admin/users/123')
      .set('Authorization', `Bearer ${userToken}`);

    expect(response.status).toBe(403);
  });

  it('should rate limit on login', async () => {
    for (let i = 0; i < 6; i++) {
      await request(app)
        .post('/api/auth/login')
        .send({ email: 'test@test.com', password: 'wrong' });
    }

    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@test.com', password: 'correct' });

    expect(response.status).toBe(429);
  });
});
```
