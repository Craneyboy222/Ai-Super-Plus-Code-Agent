---
name: /scaffold
description: >
  Initialize complete project structure from templates. Detects or asks for tech stack,
  creates directory structure, installs dependencies, configures build tools, linting,
  test framework, and git hooks. Produces a fully configured project ready for development.
---

# Scaffold Command

## Purpose
Initialize a brand new project with production-grade boilerplate, configuration, and tooling setup.

## When to Use
- Starting a new project from scratch
- Creating a new service in a monorepo
- Setting up project structure before development begins
- Establishing consistent project conventions

## Execution Steps

### 1. Tech Stack Detection
- **Ask if tech stack is known**: Use existing knowledge or detect from requirements
- **Frontend Stack**: React/Vue/Angular/Svelte selection, TypeScript yes/no
- **Backend Stack**: Node/Python/Go/Rust selection, framework choice
- **Database**: PostgreSQL/MySQL/MongoDB/Redis selection
- **Build Tool**: Webpack/Vite/Esbuild selection
- **Testing**: Jest/Vitest/Pytest/Mocha selection
- **Confirm selections**: Review and confirm with user

### 2. Directory Structure Creation
```
project/
├── src/
│   ├── components/ (frontend) or controllers/ (backend)
│   ├── pages/ (frontend)
│   ├── services/
│   ├── types/
│   ├── utils/
│   └── hooks/ (frontend)
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── public/ (frontend)
├── docs/
│   ├── architecture/
│   ├── api/
│   └── guides/
├── .github/
│   └── workflows/
├── .vscode/
│   └── settings.json
├── node_modules/ (after install)
├── .env.example
├── .env.local (git ignored)
├── .gitignore
├── .eslintrc.json
├── .prettierrc.json
├── package.json
├── tsconfig.json
├── jest.config.js
├── README.md
└── LICENSE
```

### 3. Dependency Installation
- **Initialize package.json**: With project name, version, description
- **Install Core Dependencies**:
  - Frontend: React/Vue/Svelte, routing, state management
  - Backend: Express/Nest/FastAPI, ORM, validation
  - Testing: Jest/Vitest test runner and libraries
  - Build: Webpack/Vite/Babel configuration
- **Install Dev Dependencies**:
  - Linting: ESLint, Prettier, TypeScript
  - Testing: @testing-library, mocking libraries
  - Build: Configuration and transform plugins
  - Tools: Nodemon, concurrently, etc
- **Create lock file**: package-lock.json or yarn.lock

### 4. Configuration Files
- **.gitignore**: Exclude node_modules, env files, build outputs
- **.env.example**: Template for environment variables
- **.eslintrc.json**: Linting rules configuration
- **.prettierrc.json**: Code formatting configuration
- **tsconfig.json**: TypeScript compiler options
- **jest.config.js**: Jest testing configuration
- **.vscode/settings.json**: VS Code recommendations
- **babel.config.js**: Babel transpilation (if needed)

### 5. Build Tools Configuration
- **package.json scripts**: dev, build, test, lint, format
- **Webpack/Vite config**: Entry points, loaders, plugins, optimization
- **Development server**: Hot reload, source maps, logging
- **Production build**: Minification, tree-shaking, split chunks
- **Build optimization**: Output size targets, performance budgets

### 6. Linting & Formatting Setup
- **ESLint configuration**: Rules for code quality
- **Prettier integration**: Code formatting consistency
- **Pre-commit hooks**: Lint and format before commit
- **Husky installation**: Git hooks management
- **lint-staged**: Run linters on staged files only

### 7. Test Framework Setup
- **Test configuration**: Jest/Vitest setup
- **Test helpers**: Setup files, test utilities
- **Mock factories**: Reusable test data
- **Coverage configuration**: Coverage thresholds
- **CI integration**: Test script in package.json

### 8. Git Setup
- **Initialize git**: git init if not already
- **Create .gitignore**: Standard ignores for tech stack
- **Initial commit**: "Initial commit: scaffold project"
- **Git hooks**: Pre-commit hooks with Husky
- **Husky config**: .husky directory with hooks

### 9. Documentation
- **README.md**: Project overview, setup instructions
- **CONTRIBUTING.md**: Guidelines for contributors
- **docs/architecture**: Initial architecture structure
- **docs/guides**: Setup and development guides
- **LICENSE**: MIT or selected license

### 10. Example Files
- **Sample component**: Example component with structure
- **Sample service**: Example service with patterns
- **Sample test**: Example test with best practices
- **Sample page**: Example page with routing
- **README examples**: Usage examples in documentation

## Quality Criteria

- All dependencies install without errors
- project runs after scaffold (npm start or equivalent)
- Linting runs without errors on new files
- Tests can be run with npm test
- All scripts in package.json work correctly
- No TypeScript errors if TypeScript selected
- Hot reload works in development
- Production build completes successfully
- Git commits work with pre-commit hooks
- README has clear getting-started instructions

## Output Expectations

- Clean project directory ready for development
- All dependencies installed and working
- Build tools configured and functional
- Linting and formatting working on save
- Test framework ready with example test
- Git hooks running on commits
- Documentation for next developer steps
- Example files showing code patterns
- .env.example with all needed variables
- Ready to run `npm start` immediately

## Success Indicators

- `npm start` launches dev server without errors
- `npm test` runs tests successfully
- `npm run build` creates production bundle
- `npm run lint` shows no errors on scaffolded files
- File changes are auto-formatted on save
- Git commit triggers pre-commit hooks
- README provides complete setup walkthrough
- Tech stack matches user selection
- Hot module replacement works (HMR)
- All example files follow established patterns
