# Project Templates - Expert Reference

## Table of Contents

1. [REST API Templates](#rest-api-templates)
2. [Full-Stack Web App Templates](#full-stack-web-app-templates)
3. [CLI Tool Templates](#cli-tool-templates)
4. [Library/Package Templates](#librarypackage-templates)
5. [Microservice Templates](#microservice-templates)
6. [Data Pipeline Templates](#data-pipeline-templates)

---

## REST API Templates

### Express.js REST API

**Directory Structure:**

```
project-root/
├── src/
│   ├── app.js                 # Express app setup
│   ├── server.js              # Server startup
│   ├── config/
│   │   ├── database.js
│   │   ├── environment.js
│   │   └── logger.js
│   ├── middleware/
│   │   ├── errorHandler.js
│   │   ├── authentication.js
│   │   ├── validation.js
│   │   └── cors.js
│   ├── routes/
│   │   ├── v1/
│   │   │   ├── users.js
│   │   │   ├── products.js
│   │   │   └── index.js
│   │   └── health.js          # Health check endpoint
│   ├── controllers/
│   │   ├── userController.js
│   │   └── productController.js
│   ├── models/
│   │   ├── User.js
│   │   └── Product.js
│   ├── services/
│   │   ├── userService.js
│   │   └── emailService.js
│   ├── repositories/
│   │   ├── userRepository.js
│   │   └── productRepository.js
│   ├── utils/
│   │   ├── validation.js
│   │   ├── formatters.js
│   │   └── errors.js
│   └── tests/
│       ├── unit/
│       ├── integration/
│       └── fixtures/
├── migrations/
│   └── 001_create_users_table.sql
├── docs/
│   ├── api.md
│   └── architecture.md
├── .env.example
├── .env                       # (gitignored)
├── package.json
├── package-lock.json
├── jest.config.js
├── .gitignore
└── README.md
```

**Critical Files:**

`package.json`:
```json
{
  "name": "api-server",
  "version": "1.0.0",
  "description": "REST API server",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/",
    "migrate": "node migrations/run.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "pg": "^8.8.0",
    "joi": "^17.9.0",
    "jsonwebtoken": "^9.0.0",
    "dotenv": "^16.0.0",
    "winston": "^3.8.0"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.3.0",
    "nodemon": "^2.0.0",
    "eslint": "^8.0.0"
  }
}
```

`src/app.js`:
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const errorHandler = require('./middleware/errorHandler');
const authMiddleware = require('./middleware/authentication');
const routes = require('./routes');

const app = express();

// Security
app.use(helmet());
app.use(cors({ origin: process.env.CORS_ORIGIN }));

// Body parsing
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ limit: '10kb', extended: true }));

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Routes
app.use('/api/v1', routes);

// Not found
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

// Error handling
app.use(errorHandler);

module.exports = app;
```

---

### Nest.js REST API

**Directory Structure:**

```
project-root/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── common/
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts
│   │   ├── guards/
│   │   │   └── jwt-auth.guard.ts
│   │   ├── decorators/
│   │   │   └── current-user.decorator.ts
│   │   └── pipes/
│   │       └── validation.pipe.ts
│   ├── config/
│   │   └── database.config.ts
│   ├── modules/
│   │   ├── users/
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── users.module.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-user.dto.ts
│   │   │   │   └── update-user.dto.ts
│   │   │   ├── entities/
│   │   │   │   └── user.entity.ts
│   │   │   └── users.repository.ts
│   │   └── products/
│   │       └── ...similar structure...
│   └── shared/
│       ├── services/
│       │   └── email.service.ts
│       └── utils/
├── test/
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── .env
├── .env.example
├── nest-cli.json
├── tsconfig.json
├── package.json
└── README.md
```

---

### FastAPI (Python) REST API

**Directory Structure:**

```
project-root/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── dependencies.py
│   ├── api/
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── api.py
│   │   │   ├── endpoints/
│   │   │   │   ├── users.py
│   │   │   │   └── products.py
│   │   │   └── routers.py
│   ├── models/
│   │   ├── user.py
│   │   └── product.py
│   ├── schemas/
│   │   ├── user.py
│   │   └── product.py
│   ├── crud/
│   │   ├── user.py
│   │   └── product.py
│   ├── db/
│   │   ├── base.py
│   │   ├── session.py
│   │   └── init_db.py
│   ├── services/
│   │   ├── email.py
│   │   └── auth.py
│   ├── middleware/
│   │   └── error_handler.py
│   ├── utils/
│   │   └── security.py
│   └── tests/
│       ├── unit/
│       └── integration/
├── migrations/
│   └── versions/
├── requirements.txt
├── .env
├── .env.example
├── pytest.ini
├── docker-compose.yml
└── README.md
```

**Critical Files:**

`app/main.py`:
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1 import api
from app.config import settings
from app.db.init_db import init_db

app = FastAPI(
    title="API Server",
    description="REST API",
    version="1.0.0"
)

# Middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Health check
@app.get("/health")
async def health():
    return {"status": "ok"}

# Include routes
app.include_router(api.router, prefix="/api/v1")

# Startup
@app.on_event("startup")
async def startup():
    init_db()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## Full-Stack Web App Templates

### Next.js Full-Stack

**Directory Structure:**

```
project-root/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── api/
│   │   ├── users/
│   │   │   ├── route.ts         # GET /api/users, POST /api/users
│   │   │   └── [id]/
│   │   │       └── route.ts     # GET, PUT, DELETE /api/users/[id]
│   │   └── health/
│   │       └── route.ts
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── users/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   └── products/
│   │       └── page.tsx
│   ├── auth/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   └── error.tsx
├── lib/
│   ├── api.ts                  # API client
│   ├── auth.ts                 # Authentication
│   ├── db.ts                   # Database connection
│   └── utils.ts
├── components/
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   ├── UserTable.tsx
│   └── common/
│       ├── Button.tsx
│       ├── Modal.tsx
│       └── Spinner.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useUser.ts
│   └── useFetch.ts
├── styles/
│   ├── globals.css
│   └── variables.css
├── public/
│   └── images/
├── tests/
│   ├── unit/
│   └── e2e/
├── .env.local
├── .env.example
├── next.config.js
├── tsconfig.json
├── package.json
└── README.md
```

**Key Configuration:**

`next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    domains: ['cdn.example.com'],
    formats: ['image/avif', 'image/webp']
  },
  env: {
    API_BASE_URL: process.env.API_BASE_URL
  }
};

module.exports = nextConfig;
```

---

### Nuxt.js Full-Stack

**Directory Structure:**

```
project-root/
├── app.vue
├── nuxt.config.ts
├── pages/
│   ├── index.vue
│   ├── login.vue
│   ├── dashboard/
│   │   ├── index.vue
│   │   ├── users.vue
│   │   └── users/
│   │       └── [id].vue
│   └── 404.vue
├── components/
│   ├── Header.vue
│   ├── Sidebar.vue
│   └── UserTable.vue
├── layouts/
│   ├── default.vue
│   └── auth.vue
├── server/
│   ├── api/
│   │   ├── users.ts
│   │   ├── products.ts
│   │   └── health.ts
│   ├── middleware/
│   │   └── auth.ts
│   └── utils/
│       └── db.ts
├── composables/
│   ├── useAuth.ts
│   ├── useUser.ts
│   └── useFetch.ts
├── utils/
│   └── validation.ts
├── public/
│   └── images/
├── tests/
│   ├── unit/
│   └── e2e/
├── .env
├── .env.example
├── package.json
└── README.md
```

---

## CLI Tool Templates

### Node.js CLI Tool

**Directory Structure:**

```
project-root/
├── bin/
│   └── cli.js                  # Executable entry point
├── src/
│   ├── index.ts
│   ├── commands/
│   │   ├── init.ts
│   │   ├── deploy.ts
│   │   ├── build.ts
│   │   └── help.ts
│   ├── config/
│   │   ├── loader.ts
│   │   └── validator.ts
│   ├── services/
│   │   ├── deployment.ts
│   │   └── builder.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── spinner.ts
│   │   └── table.ts
│   └── types/
│       └── config.ts
├── tests/
│   └── commands.test.ts
├── templates/
│   ├── .gitignore
│   ├── package.json
│   └── src/
│       └── index.ts
├── package.json
├── tsconfig.json
└── README.md
```

**Critical Files:**

`bin/cli.js`:
```javascript
#!/usr/bin/env node

const cli = require('../dist/index.js');

cli.run(process.argv.slice(2))
  .catch(err => {
    console.error(err.message);
    process.exit(1);
  });
```

`package.json`:
```json
{
  "name": "my-cli-tool",
  "version": "1.0.0",
  "description": "CLI tool for X",
  "bin": {
    "my-cli": "./bin/cli.js"
  },
  "scripts": {
    "build": "tsc",
    "dev": "ts-node src/index.ts"
  },
  "dependencies": {
    "yargs": "^17.0.0",
    "chalk": "^5.0.0",
    "ora": "^6.0.0",
    "inquirer": "^9.0.0"
  }
}
```

`src/index.ts`:
```typescript
import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';
import * as initCommand from './commands/init';
import * as deployCommand from './commands/deploy';

export async function run(argv: string[]) {
  return yargs(hideBin(argv))
    .command(initCommand.command, initCommand.describe, initCommand.builder, initCommand.handler)
    .command(deployCommand.command, deployCommand.describe, deployCommand.builder, deployCommand.handler)
    .demandCommand(1)
    .strict()
    .argv;
}

run(process.argv.slice(2)).catch(console.error);
```

---

### Python CLI Tool

**Directory Structure:**

```
project-root/
├── my_cli/
│   ├── __init__.py
│   ├── main.py
│   ├── cli.py                  # Click/Typer setup
│   ├── commands/
│   │   ├── __init__.py
│   │   ├── init.py
│   │   ├── deploy.py
│   │   └── build.py
│   ├── config/
│   │   ├── __init__.py
│   │   ├── loader.py
│   │   └── schema.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── deployment.py
│   │   └── builder.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py
│   │   ├── spinner.py
│   │   └── table.py
│   └── templates/
│       ├── .gitignore
│       └── setup.py
├── tests/
│   ├── test_commands.py
│   └── test_config.py
├── pyproject.toml
├── setup.py
├── requirements.txt
└── README.md
```

---

## Library/Package Templates

### npm Package

**Directory Structure:**

```
project-root/
├── src/
│   ├── index.ts
│   ├── core/
│   │   └── calculator.ts
│   ├── utils/
│   │   └── helpers.ts
│   └── types/
│       └── index.ts
├── tests/
│   ├── unit/
│   │   └── calculator.test.ts
│   └── integration/
├── docs/
│   ├── README.md
│   ├── API.md
│   └── examples/
│       └── basic.ts
├── .github/
│   └── workflows/
│       ├── test.yml
│       └── publish.yml
├── .npmrc
├── tsconfig.json
├── package.json
├── README.md
└── LICENSE
```

**package.json**:
```json
{
  "name": "@myorg/my-package",
  "version": "1.0.0",
  "description": "Description",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "prepublishOnly": "npm run build && npm test"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/myorg/my-package"
  },
  "keywords": ["tag1", "tag2"],
  "author": "Your Name",
  "license": "MIT"
}
```

---

## Microservice Templates

**Directory Structure:**

```
project-root/
├── src/
│   ├── index.ts
│   ├── server.ts
│   ├── config/
│   │   └── logger.ts
│   ├── routes/
│   │   └── health.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   └── errorHandler.ts
│   ├── services/
│   ├── repositories/
│   └── tests/
├── docker/
│   ├── Dockerfile
│   └── .dockerignore
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

**Dockerfile**:
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**kubernetes/deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-service
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
      - name: my-service
        image: my-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 30
```

---

## Data Pipeline Templates

**Directory Structure:**

```
project-root/
├── src/
│   ├── main.py
│   ├── config.py
│   ├── extractors/
│   │   ├── api_extractor.py
│   │   └── database_extractor.py
│   ├── transformers/
│   │   ├── data_cleaner.py
│   │   └── aggregator.py
│   ├── loaders/
│   │   ├── database_loader.py
│   │   └── warehouse_loader.py
│   ├── schedulers/
│   │   └── dag.py
│   ├── monitoring/
│   │   ├── health.py
│   │   └── metrics.py
│   └── tests/
├── dags/
│   └── main_pipeline.py
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── README.md
```

**dags/main_pipeline.py** (Airflow):
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'data_pipeline',
    default_args=default_args,
    description='ETL pipeline',
    schedule_interval='0 2 * * *',  # Daily at 2 AM
    start_date=datetime(2024, 1, 1),
    catchup=False
)

extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    dag=dag
)

transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
    dag=dag
)

load = PythonOperator(
    task_id='load',
    python_callable=load_data,
    dag=dag
)

extract >> transform >> load
```

---

## Summary: Template Selection Guide

| Project Type | Recommended Framework | Best For |
|---|---|---|
| REST API | Express / Nest.js / FastAPI | Microservices, APIs |
| Full-Stack | Next.js / Nuxt.js | Web applications |
| CLI Tool | Click / Yargs | Command-line tools |
| Library | TypeScript + ESBuild | Reusable packages |
| Microservice | Express + Docker | Containerized services |
| Data Pipeline | Apache Airflow | Scheduled ETL jobs |

Each template is production-ready with:
- Proper dependency management
- Testing structure
- Environment configuration
- Deployment readiness
- Documentation scaffolding
