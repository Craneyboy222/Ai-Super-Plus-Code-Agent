# Deployment Systems — Docker, K8s, CI/CD, Cloud

## Docker Configuration

### Dockerfile (Multi-stage for Node.js)

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Runtime
FROM node:20-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Copy from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy application
COPY --chown=nodejs:nodejs . .

# Build app (if needed)
RUN npm run build

# Switch to non-root user
USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3001/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["node", "dist/server.js"]
```

### docker-compose.yml (Development)

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-password}
      POSTGRES_DB: ${DB_NAME:-blog_dev}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://${DB_USER:-postgres}:${DB_PASSWORD:-password}@postgres:5432/${DB_NAME:-blog_dev}
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET:-dev-secret}
    ports:
      - "3001:3001"
    volumes:
      - ./backend/src:/app/src
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: npm run dev

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:3001
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
    depends_on:
      - backend

volumes:
  postgres_data:
```

---

## Kubernetes Deployment

### Deployment Manifest

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog-backend
  labels:
    app: blog-backend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: blog-backend
  template:
    metadata:
      labels:
        app: blog-backend
    spec:
      serviceAccountName: blog-backend
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
      containers:
        - name: backend
          image: myregistry.azurecr.io/blog-backend:latest
          imagePullPolicy: Always
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          ports:
            - containerPort: 3001
              name: http
          env:
            - name: NODE_ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: blog-secrets
                  key: database-url
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: blog-secrets
                  key: jwt-secret
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3001
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /health
              port: 3001
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

### Service Manifest

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: blog-backend
spec:
  type: LoadBalancer
  selector:
    app: blog-backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3001
      name: http
```

### Secrets

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: blog-secrets
type: Opaque
stringData:
  database-url: postgresql://user:password@postgres:5432/blog
  jwt-secret: your-secret-key-here
```

---

## CI/CD Pipeline (GitHub Actions)

### Workflow File

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: password
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:coverage
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | grep lines | grep pct | cut -d: -f2 | cut -d, -f1 | tr -d ' ')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage too low: $COVERAGE%"
            exit 1
          fi

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm audit --production
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          registry: myregistry.azurecr.io
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: myregistry.azurecr.io/blog-backend:latest,myregistry.azurecr.io/blog-backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: azure/setup-kubectl@v3
      - uses: azure/aks-set-context@v3
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
          cluster-name: my-aks-cluster
          resource-group: my-resource-group
      - run: kubectl set image deployment/blog-backend backend=myregistry.azurecr.io/blog-backend:${{ github.sha }}
      - run: kubectl rollout status deployment/blog-backend
```

---

## Cloud Deployments

### Vercel (Next.js Frontend)

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "env": {
    "NEXT_PUBLIC_API_URL": "@api_url"
  },
  "env.production": {
    "NEXT_PUBLIC_API_URL": "https://api.example.com"
  },
  "env.preview": {
    "NEXT_PUBLIC_API_URL": "https://staging-api.example.com"
  }
}
```

### AWS Elastic Beanstalk (Backend)

```yaml
# .ebextensions/app.config
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeVersion: 20.0.0
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    NPM_USE_PRODUCTION: true
  aws:rds:dbinstance:
    DBEngineVersion: 15.1
    Engine: postgres
    MultiAZ: true

  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 6

  aws:autoscaling:trigger:
    MeasureName: CPUUtilization
    Statistic: Average
    Unit: Percent
    UpperThreshold: 70
    LowerThreshold: 30

commands:
  01_migrate:
    command: "npm run db:migrate"
    leader_only: true

container_commands:
  01_seed:
    command: "npm run db:seed"
    leader_only: true
```

### AWS CloudFormation (Infrastructure as Code)

```yaml
# infrastructure.yaml
AWSTemplateFormatVersion: '2010-09-09'

Parameters:
  EnvironmentName:
    Type: String
    Default: production

Resources:
  RDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: blog-db
      Engine: postgres
      EngineVersion: '15.1'
      DBInstanceClass: db.t4g.micro
      AllocatedStorage: '20'
      MasterUsername: postgres
      MasterUserPassword: !Sub '{{resolve:secretsmanager:blog-db-password:SecretString:password}}'
      DBName: blog
      VPCSecurityGroups:
        - !Ref RDSSecurityGroup
      MultiAZ: true
      BackupRetentionPeriod: 30
      StorageEncrypted: true

  ElastiCacheCluster:
    Type: AWS::ElastiCache::CacheCluster
    Properties:
      CacheNodeType: cache.t4g.micro
      Engine: redis
      EngineVersion: '7.0'
      NumCacheNodes: 1
      VpcSecurityGroupIds:
        - !Ref CacheSecurityGroup

  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Scheme: internet-facing
      Type: application
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2

Outputs:
  DatabaseEndpoint:
    Value: !GetAtt RDSInstance.Endpoint.Address
  CacheEndpoint:
    Value: !GetAtt ElastiCacheCluster.RedisEndpoint.Address
  LoadBalancerDNS:
    Value: !GetAtt ApplicationLoadBalancer.DNSName
```

---

## Environment Configuration

### .env.example

```bash
# Application
NODE_ENV=production
PORT=3001
LOG_LEVEL=info

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/blog
DB_POOL_MIN=2
DB_POOL_MAX=10

# Cache
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=your-secret-key
JWT_EXPIRY=15m
REFRESH_TOKEN_EXPIRY=7d

# Email (if needed)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-password

# External APIs
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret

# Monitoring
SENTRY_DSN=your-sentry-dsn
DATADOG_API_KEY=your-datadog-key

# CORS
ALLOWED_ORIGINS=https://example.com,https://app.example.com

# Security
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

---

## Production Deployment Checklist

```
PRE-DEPLOYMENT
☐ All tests passing (>80% coverage)
☐ No TypeScript errors
☐ No ESLint warnings
☐ Security audit passed
☐ Performance targets met
☐ Load test completed
☐ Database migrations tested
☐ Rollback plan documented

DEPLOYMENT
☐ Secrets configured in environment
☐ Database backups enabled
☐ Monitoring configured
☐ Logging configured
☐ Error tracking active
☐ Health checks passing
☐ DNS configured
☐ SSL certificate valid

POST-DEPLOYMENT
☐ Smoke tests passing
☐ API responding correctly
☐ Database connections stable
☐ Logs no critical errors
☐ Performance metrics normal
☐ Error rate at baseline
☐ Team notified
☐ Rollback procedure ready
```

---

## Monitoring & Observability

### Health Check Endpoint

```typescript
app.get('/health', async (req, res) => {
  try {
    await db.$queryRaw`SELECT 1`;
    await redis.ping();

    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
      environment: process.env.NODE_ENV
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error instanceof Error ? error.message : 'Unknown error'
    });
  }
});
```

### Structured Logging

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: false,
      singleLine: true,
      translateTime: 'SYS:standard'
    }
  }
});

// Usage
logger.info({
  event: 'user_login',
  userId: user.id,
  timestamp: new Date().toISOString(),
  ip: req.ip
});
```

All systems configured for reliability, scalability, and observability.
