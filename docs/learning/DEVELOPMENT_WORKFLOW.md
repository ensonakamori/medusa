# Development Workflow

**Your complete guide to daily Medusa development workflow**

**Last Updated:** November 19, 2025
**Version:** 2.11.3

---

## Table of Contents

1. [Git Workflow](#git-workflow)
2. [Build System (Turbo)](#build-system-turbo)
3. [Development Environment Setup](#development-environment-setup)
4. [Hot Reload / Watch Mode](#hot-reload--watch-mode)
5. [Testing Workflow](#testing-workflow)
6. [Debugging Setup](#debugging-setup)
7. [Code Review Checklist](#code-review-checklist)
8. [CI/CD Awareness](#cicd-awareness)
9. [Common Commands Reference](#common-commands-reference)

---

## Git Workflow

### Branch Naming Convention

```bash
# Feature branches
feat/add-review-module
feat/stripe-oxxo-payment

# Bug fixes
fix/cart-total-calculation
fix/product-image-upload

# Chores (refactoring, dependencies)
chore/update-dependencies
chore/refactor-workflow-types

# Documentation
docs/add-api-examples
docs/update-getting-started
```

**Pattern:** `type/short-description-kebab-case`

### Daily Workflow

#### 1. Start Your Day

```bash
# Navigate to project
cd /home/user/medusa

# Pull latest changes
git checkout main
git pull origin main

# Check status
git status
```

#### 2. Create Feature Branch

```bash
# Create and switch to new branch
git checkout -b feat/add-review-ratings

# Verify branch
git branch
# * feat/add-review-ratings
#   main
```

#### 3. Make Changes

```bash
# Check what you've changed
git status

# Review your changes
git diff

# Review staged changes
git diff --staged
```

#### 4. Commit Changes

```bash
# Stage specific files
git add packages/modules/review/src/models/review.ts
git add packages/modules/review/src/services/review-module.ts

# Or stage all changes (use carefully)
git add .

# Commit with meaningful message
git commit -m "feat(review): add rating field to review model

- Add rating field (1-5 stars)
- Add average_rating calculation method
- Add database index on product_id and rating
- Add validation for rating range"

# View commit
git log --oneline -1
```

**Commit Message Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `chore`: Maintenance
- `docs`: Documentation
- `test`: Adding tests
- `refactor`: Code refactoring

**Examples:**
```bash
git commit -m "feat(product): add external_id field"
git commit -m "fix(cart): correct total calculation with tax"
git commit -m "chore(deps): update dependencies"
git commit -m "docs: add workflow examples"
git commit -m "test(review): add integration tests"
```

#### 5. Push to Remote

```bash
# First push (set upstream)
git push -u origin feat/add-review-ratings

# Subsequent pushes
git push
```

#### 6. Create Pull Request

```bash
# Using GitHub CLI (recommended)
gh pr create \
  --title "feat(review): Add product review ratings" \
  --body "## What
- Add rating field (1-5 stars) to review model
- Add average rating calculation
- Add validation

## Why
Customers need to rate products

## Testing
- Unit tests added
- Tested manually in local dev

## Screenshots
[Add if relevant]"

# Or visit GitHub and create PR manually
```

### Pull Request Best Practices

**Title Format:**
```
feat(review): Add product review ratings
fix(cart): Correct tax calculation
chore(deps): Update TypeScript to 5.6.2
```

**PR Description Template:**
```markdown
## What
- Brief bullet points of changes

## Why
- Problem this solves
- Business context

## How
- Technical approach
- Key decisions made

## Testing
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing completed
- [ ] Documentation updated

## Screenshots
[If UI changes]

## Breaking Changes
[If any]

## Migration Guide
[If needed]
```

### Handling Code Review

```bash
# Fetch latest reviews
gh pr view

# Make requested changes
git add .
git commit -m "fix: address review comments"
git push

# After approval, merge via GitHub UI or:
gh pr merge --squash

# Delete local branch
git branch -d feat/add-review-ratings

# Delete remote branch (if not auto-deleted)
git push origin --delete feat/add-review-ratings
```

### Keeping Branch Updated

```bash
# Get latest from main
git checkout main
git pull origin main

# Switch back to feature branch
git checkout feat/add-review-ratings

# Rebase on main (cleaner history)
git rebase main

# Or merge main into branch (preserves history)
git merge main

# Push (force if rebased)
git push --force-with-lease
```

### Handling Merge Conflicts

```bash
# During rebase/merge
git status
# Shows conflicted files

# Open conflicted file, look for:
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> main

# Resolve conflicts, then:
git add path/to/resolved/file
git rebase --continue
# or
git merge --continue

# Push
git push --force-with-lease
```

### Git Aliases (Time Savers)

Add to `~/.gitconfig`:

```ini
[alias]
  st = status
  co = checkout
  br = branch
  cm = commit -m
  ca = commit --amend
  pl = pull
  ps = push
  lg = log --oneline --graph --decorate --all
  df = diff
  dfs = diff --staged
```

Usage:
```bash
git st          # instead of git status
git co main     # instead of git checkout main
git lg          # pretty log
```

---

## Build System (Turbo)

Medusa uses **Turborepo** for fast, cached builds across packages.

### Understanding Turbo

**File:** `/home/user/medusa/turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["!node_modules/**", "!src/**", "*/**"]
    },
    "test": {
      "outputs": []
    },
    "test:integration": {
      "outputs": []
    }
  }
}
```

**What This Means:**
- `"dependsOn": ["^build"]`: Build dependencies first
- `"outputs": [...]`: Cache these directories
- Turbo skips rebuilding unchanged packages

### Build Commands

```bash
# Build all packages (with caching)
yarn build

# Build specific package
cd packages/modules/product
yarn build

# Build with verbose output
yarn build --verbose

# Force rebuild (ignore cache)
yarn build --force

# Build only changed packages
yarn build --filter=[SINCE_GIT_REF]
```

### Build Order

Turbo automatically builds in correct order:

```
1. Core packages:
   - @medusajs/framework/utils
   - @medusajs/framework/types
   - @medusajs/framework/workflows-sdk

2. Modules:
   - @medusajs/product
   - @medusajs/cart
   - @medusajs/order
   - ...

3. Core flows:
   - @medusajs/core-flows

4. Main package:
   - @medusajs/medusa

5. Admin dashboard:
   - @medusajs/admin/dashboard
```

### Watching Build Outputs

```bash
# Watch all packages
yarn workspaces foreach --parallel run build:watch

# Watch specific package
cd packages/modules/product
yarn build:watch
```

### Cache Management

```bash
# Clear Turbo cache
rm -rf node_modules/.cache/turbo

# Check cache hits
yarn build --dry-run

# Turbo cache location
ls -la node_modules/.cache/turbo
```

### Common Build Issues

**Issue 1: Type errors across packages**
```bash
# Solution: Clean and rebuild all
rm -rf packages/*/dist
yarn build
```

**Issue 2: Stale cache**
```bash
# Solution: Force rebuild
yarn build --force
```

**Issue 3: Dependency order**
```bash
# Solution: Check turbo.json dependencies
# Ensure "dependsOn": ["^build"] is set
```

---

## Development Environment Setup

### Initial Setup (One-Time)

```bash
# 1. Clone repository
git clone https://github.com/medusajs/medusa.git
cd medusa

# 2. Install dependencies
yarn install

# 3. Build all packages
yarn build

# 4. Set up database
createdb medusa_db

# 5. Run migrations
yarn medusa db:migrate

# 6. Create admin user
yarn medusa user -e admin@test.com -p supersecret

# 7. Seed data (optional)
yarn medusa db:seed
```

### Environment Variables

**File:** `/home/user/medusa/.env`

```bash
# Database
DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa_db

# Redis (for caching and events)
REDIS_URL=redis://localhost:6379

# Server
PORT=9000
JWT_SECRET=your-secret-key-here
COOKIE_SECRET=your-cookie-secret-here

# Admin CORS
ADMIN_CORS=http://localhost:7001,http://localhost:9000

# Store CORS
STORE_CORS=http://localhost:8000

# Providers (if needed)
STRIPE_API_KEY=sk_test_...
SENDGRID_API_KEY=SG...
S3_BUCKET=my-bucket
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...
```

### IDE Setup (VSCode)

**File:** `.vscode/settings.json`

```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.turbo": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/yarn.lock": true
  }
}
```

**File:** `.vscode/extensions.json`

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss"
  ]
}
```

### Daily Start

```bash
# Terminal 1: Backend
yarn dev

# Terminal 2: Admin Dashboard (if developing UI)
cd packages/admin/dashboard
yarn dev

# Terminal 3: Watch builds (if developing modules)
yarn workspaces foreach --parallel run build:watch
```

---

## Hot Reload / Watch Mode

### Backend Hot Reload

```bash
# Start dev server with hot reload
yarn dev

# What happens:
# 1. Watches src/ files
# 2. Rebuilds on change
# 3. Restarts server automatically
# 4. Preserves database connections
```

**Logs:**
```
info:    Server is ready on port 9000
info:    Watching for changes...
info:    File changed: packages/modules/product/src/models/product.ts
info:    Rebuilding product module...
info:    Restarting server...
info:    Server is ready on port 9000
```

### Admin Dashboard Hot Reload

```bash
cd packages/admin/dashboard
yarn dev

# Uses Vite HMR (Hot Module Replacement)
# Changes reflect instantly without full reload
```

**What's Hot Reloaded:**
- React components
- Styles (CSS, Tailwind)
- Routes
- TypeScript types

**What Requires Refresh:**
- Environment variables
- Build configuration
- Dependencies

### Module Development Workflow

**Terminal 1:**
```bash
# Watch module builds
cd packages/modules/review
yarn build:watch
```

**Terminal 2:**
```bash
# Run dev server
cd /home/user/medusa
yarn dev
```

**Workflow:**
1. Edit `packages/modules/review/src/models/review.ts`
2. Watch terminal 1 - sees rebuild
3. Watch terminal 2 - sees server restart
4. Test changes

### Debugging Hot Reload Issues

```bash
# If changes not reflecting:

# 1. Check file is watched
ls -la packages/modules/product/src/models/

# 2. Check build output
ls -la packages/modules/product/dist/

# 3. Restart dev server
# Ctrl+C, then:
yarn dev

# 4. Clear build cache
rm -rf packages/modules/*/dist
yarn build
yarn dev
```

---

## Testing Workflow

### Test Types

**1. Unit Tests:**
```bash
# Run all unit tests
yarn test

# Run specific module tests
cd packages/modules/product
yarn test

# Run specific test file
yarn test src/services/__tests__/product.spec.ts

# Watch mode
yarn test --watch
```

**2. Integration Tests:**
```bash
# Run all integration tests
yarn test:integration

# Run specific integration test
cd packages/modules/product
yarn test:integration

# Run with coverage
yarn test:integration --coverage
```

**3. API Tests:**
```bash
# Run API integration tests
yarn test:integration:api

# Run HTTP tests
yarn test:integration:http
```

### Test-Driven Development (TDD)

**Workflow:**

1. **Write failing test:**
```typescript
// packages/modules/review/integration-tests/__tests__/review.spec.ts
it("should calculate average rating", async () => {
  await service.createReview({ product_id: "prod_1", rating: 5 })
  await service.createReview({ product_id: "prod_1", rating: 3 })

  const avg = await service.getAverageRating("prod_1")

  expect(avg).toBe(4) // (5 + 3) / 2
})
```

2. **Run test (should fail):**
```bash
yarn test
# ✗ should calculate average rating
#   Error: service.getAverageRating is not a function
```

3. **Implement feature:**
```typescript
// packages/modules/review/src/services/review-module.ts
async getAverageRating(productId: string): Promise<number> {
  const reviews = await this.listReviews({ product_id: productId })
  if (reviews.length === 0) return 0
  const sum = reviews.reduce((acc, r) => acc + r.rating, 0)
  return sum / reviews.length
}
```

4. **Run test (should pass):**
```bash
yarn test
# ✓ should calculate average rating (45ms)
```

### Coverage Reports

```bash
# Generate coverage report
yarn test --coverage

# View in terminal:
# ----------------------|---------|----------|---------|---------|
# File                  | % Stmts | % Branch | % Funcs | % Lines |
# ----------------------|---------|----------|---------|---------|
# All files            |   78.45 |    65.23 |   82.10 |   79.32 |
#  product-module.ts   |   85.20 |    72.50 |   88.00 |   86.10 |
# ----------------------|---------|----------|---------|---------|

# Open HTML report
open coverage/lcov-report/index.html
```

### Pre-Commit Testing

```bash
# Run before committing
yarn test
yarn test:integration

# Or use git hooks
# .husky/pre-commit
#!/bin/sh
yarn test
```

---

## Debugging Setup

### VSCode Debugger

**File:** `.vscode/launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Medusa Server",
      "runtimeExecutable": "yarn",
      "runtimeArgs": ["dev"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"],
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Current Test",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": [
        "${file}",
        "--runInBand",
        "--no-cache"
      ],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

**Usage:**
1. Set breakpoint (click left of line number)
2. Press F5 or click "Run and Debug"
3. Choose "Debug Medusa Server"
4. Code pauses at breakpoint

**Debugging Tests:**
1. Open test file
2. Set breakpoint
3. Press F5
4. Choose "Debug Current Test"

### Console Debugging

```typescript
// In your code
export class ReviewModuleService {
  async createReview(data: CreateReviewInput) {
    console.log("📥 Input:", JSON.stringify(data, null, 2))

    const review = await this.create(data)

    console.log("✅ Created:", review.id)

    return review
  }
}
```

**Better Logging:**
```typescript
// Use logger (better than console.log)
export class ReviewModuleService {
  protected logger_: Logger

  async createReview(data: CreateReviewInput) {
    this.logger_.info("Creating review", { data })

    try {
      const review = await this.create(data)
      this.logger_.info("Review created", { id: review.id })
      return review
    } catch (error) {
      this.logger_.error("Failed to create review", { error, data })
      throw error
    }
  }
}
```

### Database Query Debugging

**Method 1: Enable Query Logging**

```bash
# In .env
DATABASE_LOGGING=true
```

**Method 2: PostgreSQL Logs**

```bash
# Enable in postgresql.conf
log_statement = 'all'
log_duration = on

# Tail logs
tail -f /var/log/postgresql/postgresql-13-main.log
```

**Method 3: MikroORM Debug**

```typescript
// In medusa-config.ts
module.exports = {
  projectConfig: {
    database_extra: {
      debug: true, // Log all queries
    },
  },
}
```

### Network Debugging

**Using curl:**
```bash
# Verbose request
curl -v http://localhost:9000/admin/products \
  -H "Authorization: Bearer YOUR_TOKEN"

# Save response headers
curl -D headers.txt http://localhost:9000/admin/products

# Pretty print JSON
curl http://localhost:9000/admin/products | jq
```

**Using httpie:**
```bash
# Cleaner output than curl
http :9000/admin/products Authorization:"Bearer TOKEN"

# POST request
http POST :9000/admin/products \
  title="Test Product" \
  Authorization:"Bearer TOKEN"
```

### Chrome DevTools (for Admin)

1. Open Admin Dashboard (http://localhost:7001)
2. Press F12
3. **Network tab:** See API calls
4. **Console tab:** See logs
5. **Sources tab:** Debug React code
6. **React DevTools:** Inspect component tree

---

## Code Review Checklist

### Before Submitting PR

**Code Quality:**
- [ ] Code follows naming conventions
- [ ] No console.log (use logger instead)
- [ ] Error handling implemented
- [ ] TypeScript types are proper (no `any`)
- [ ] Comments explain "why", not "what"

**Testing:**
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] All tests passing (`yarn test`)
- [ ] Manual testing completed

**Database:**
- [ ] Migrations created
- [ ] Migrations tested (up and down)
- [ ] Indexes added for foreign keys
- [ ] No breaking changes to existing data

**API:**
- [ ] Validators added (Zod schemas)
- [ ] Middleware registered
- [ ] Error responses proper
- [ ] Documentation updated

**Dependencies:**
- [ ] No unnecessary dependencies
- [ ] Dependencies are latest stable
- [ ] Security vulnerabilities checked

**Git:**
- [ ] Commit messages meaningful
- [ ] Branch name follows convention
- [ ] No merge commits (rebased)
- [ ] Changesets added (if releasing)

### Reviewing Others' PRs

**Checklist:**
- [ ] Read PR description
- [ ] Understand the "why"
- [ ] Check for breaking changes
- [ ] Review tests first
- [ ] Review core logic
- [ ] Check error handling
- [ ] Look for edge cases
- [ ] Test locally if complex

**Feedback Format:**
```markdown
## Approval
✅ LGTM (Looks Good To Me)

## Required Changes
❗ Missing error handling in createReview()

## Suggestions
💡 Consider adding index on product_id + customer_id

## Questions
❓ Why use Promise.all instead of sequential?

## Nitpicks
🔍 Typo: "recieve" → "receive"
```

**Tone:**
- ✅ "Consider using `transform()` here for clarity"
- ✅ "What's the reason for this approach?"
- ❌ "This is wrong"
- ❌ "Why didn't you use X?"

---

## CI/CD Awareness

### GitHub Actions Workflow

**File:** `.github/workflows/test.yml` (example)

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install dependencies
        run: yarn install

      - name: Build
        run: yarn build

      - name: Run tests
        run: yarn test

      - name: Run integration tests
        run: yarn test:integration
```

### What CI Checks

1. **Linting:** `yarn lint`
2. **Type Checking:** `tsc --noEmit`
3. **Unit Tests:** `yarn test`
4. **Integration Tests:** `yarn test:integration`
5. **Build:** `yarn build`

### Local Pre-Push Check

```bash
# Run what CI will run
yarn lint
yarn test
yarn test:integration
yarn build

# Or create script
cat > check.sh << 'EOF'
#!/bin/bash
set -e
echo "Running pre-push checks..."
yarn lint
yarn test
yarn build
echo "✅ All checks passed!"
EOF

chmod +x check.sh
./check.sh
```

### Deployment Process

```bash
# 1. Create changeset (for releases)
yarn changeset

# Follow prompts:
# - Which packages changed?
# - What type of change? (major/minor/patch)
# - Summary of changes

# 2. Commit changeset
git add .changeset/
git commit -m "chore: add changeset"

# 3. Create PR
gh pr create

# 4. After merge, CI creates release PR

# 5. Merge release PR
# CI publishes to npm automatically
```

---

## Common Commands Reference

### Git
```bash
git status                              # Check status
git diff                                # See changes
git add .                               # Stage all
git commit -m "message"                 # Commit
git push                                # Push to remote
git pull                                # Pull from remote
git checkout -b feat/new-feature        # Create branch
git rebase main                         # Rebase on main
git log --oneline --graph               # Pretty log
```

### Development
```bash
yarn dev                                # Start dev server
yarn build                              # Build all packages
yarn build --force                      # Force rebuild
yarn test                               # Run tests
yarn test:integration                   # Integration tests
yarn lint                               # Lint code
```

### Database
```bash
yarn medusa db:migrate                  # Run migrations
yarn medusa db:migrate:rollback         # Rollback migration
yarn medusa db:generate MODULE "Name"   # Generate migration
yarn medusa db:seed                     # Seed data
yarn medusa db:reset                    # Reset database
```

### Package Management
```bash
yarn install                            # Install dependencies
yarn add package-name                   # Add dependency
yarn remove package-name                # Remove dependency
yarn upgrade-interactive                # Update dependencies
```

### Monorepo
```bash
yarn workspace @medusajs/product build  # Build specific package
yarn workspaces foreach run build       # Build all workspaces
yarn workspaces foreach --parallel run dev # Run dev in parallel
```

### Turbo
```bash
yarn build                              # Build with cache
yarn build --force                      # Ignore cache
yarn build --dry-run                    # Show what would run
yarn build --filter=@medusajs/product   # Build specific package
```

### User Management
```bash
yarn medusa user                        # Create user interactively
yarn medusa user -e admin@test.com -p pass  # Create with email/password
```

### Debugging
```bash
yarn dev --verbose                      # Verbose output
NODE_ENV=development yarn dev           # Explicit dev mode
DEBUG=* yarn dev                        # Debug all
```

---

## Daily Workflow Example

**Morning:**
```bash
# 1. Pull latest
git checkout main
git pull

# 2. Start work
git checkout -b feat/add-product-rating

# 3. Start dev servers
yarn dev                    # Terminal 1
cd packages/admin/dashboard && yarn dev  # Terminal 2 (if needed)
```

**During Development:**
```bash
# 4. Make changes
vim packages/modules/product/src/models/product.ts

# 5. Check changes
git diff

# 6. Test
yarn test packages/modules/product

# 7. Commit
git add .
git commit -m "feat(product): add rating field"
```

**End of Day:**
```bash
# 8. Push
git push -u origin feat/add-product-rating

# 9. Create PR
gh pr create

# 10. Clean up
git checkout main
```

**After PR Merge:**
```bash
# 11. Delete branch
git branch -d feat/add-product-rating

# 12. Pull main
git pull
```

---

## Productivity Tips

### Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# Medusa shortcuts
alias md="cd ~/medusa"
alias mdev="cd ~/medusa && yarn dev"
alias mtest="cd ~/medusa && yarn test"
alias mbuild="cd ~/medusa && yarn build --force"
alias mdb="yarn medusa db:migrate"
```

### VSCode Snippets

**File:** `.vscode/medusa.code-snippets`

```json
{
  "Medusa Step": {
    "prefix": "mstep",
    "body": [
      "export const ${1:stepName}StepId = \"${1:step-name}\"",
      "",
      "export const ${1:stepName}Step = createStep(",
      "  ${1:stepName}StepId,",
      "  async (data: ${2:InputType}, { container }) => {",
      "    const service = container.resolve(Modules.${3:MODULE})",
      "    ",
      "    const result = await service.${4:method}(data)",
      "    ",
      "    return new StepResponse(result, result.id)",
      "  },",
      "  async (id, { container }) => {",
      "    if (!id) return",
      "    ",
      "    const service = container.resolve(Modules.${3:MODULE})",
      "    await service.delete${5:Entity}(id)",
      "  }",
      ")"
    ]
  }
}
```

### tmux Session

```bash
# Create medusa development session
tmux new -s medusa

# Split windows
# Ctrl+B, % (vertical split)
# Ctrl+B, " (horizontal split)

# Layout:
# +----------------+----------------+
# | Backend (dev)  | Admin (dev)    |
# +----------------+----------------+
# | Git/Commands   | Tests          |
# +----------------+----------------+

# Attach to session
tmux attach -t medusa
```

---

**Quick Reference Card:**

```
DAILY COMMANDS:
  yarn dev               Start dev server
  yarn test              Run tests
  yarn build             Build packages
  git status             Check changes
  git commit -m ""       Commit changes
  git push               Push to remote

DEBUGGING:
  F5                     Start debugger
  console.log()          Quick debug
  this.logger_.info()    Proper logging
  curl -v                Test API

GIT:
  git checkout -b        Create branch
  git add .              Stage all
  git commit -m          Commit
  git push               Push
  gh pr create           Create PR

DATABASE:
  yarn medusa db:migrate         Run migrations
  yarn medusa db:generate        Create migration
  yarn medusa db:rollback        Undo migration

PRODUCTIVITY:
  Ctrl+C                 Stop server
  Ctrl+Z, bg             Background process
  ↑                      Previous command
  Tab                    Autocomplete
```

---

**Next Steps:**
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Code patterns and best practices
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step task recipes
- [CODE_TOURS.md](./CODE_TOURS.md) - Guided code walkthroughs
