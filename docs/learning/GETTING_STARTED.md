# Getting Started with Medusa

**Welcome!** This guide will walk you through setting up the Medusa e-commerce platform locally from scratch.

**Version:** 2.11.3
**Last Updated:** November 18, 2025
**Time to Complete:** 30-60 minutes

---

## Table of Contents

1. [What You're Setting Up](#what-youre-setting-up)
2. [Prerequisites](#prerequisites)
3. [Quick Start (For the Impatient)](#quick-start-for-the-impatient)
4. [Detailed Setup Guide](#detailed-setup-guide)
5. [Accessing the Application](#accessing-the-application)
6. [Verification Steps](#verification-steps)
7. [Common Issues & Troubleshooting](#common-issues--troubleshooting)
8. [Next Steps](#next-steps)

---

## What You're Setting Up

Medusa is like **Express + React Admin + Shopify's backend** combined into one modular platform. Think of it as:

- **The Backend:** An Express-based API server with 33+ commerce modules (like microservices, but in a monolith)
- **The Admin:** A React 18 dashboard (like WordPress admin, but for e-commerce)
- **The Database:** PostgreSQL with MikroORM (like Sequelize or TypeORM, but newer)

**Analogy:** If React is a component library for UIs, Medusa is a **module library for commerce**.

---

## Prerequisites

### Required Software

| Tool | Version | Why You Need It | Check Command |
|------|---------|-----------------|---------------|
| **Node.js** | 20+ | JavaScript runtime (LTS recommended) | `node --version` |
| **Yarn** | 3.2.1+ | Package manager (faster than npm) | `yarn --version` |
| **PostgreSQL** | 13+ | Database (like MySQL, but better for complex data) | `psql --version` |
| **Git** | Latest | Version control | `git --version` |

### Knowledge Prerequisites

- ✅ Basic JavaScript/TypeScript
- ✅ Familiarity with Node.js
- ✅ Command line comfort
- ⚠️ React knowledge helpful (but not required for backend work)
- ⚠️ PostgreSQL basics helpful (but we'll guide you)

---

## Quick Start (For the Impatient)

**Option A: Create a New Medusa Project** (Recommended for new users)

```bash
# Create a new Medusa project (like create-react-app)
npx create-medusa-app@latest

# Follow the prompts to:
# 1. Choose a project name
# 2. Configure PostgreSQL database
# 3. Optionally seed demo data

# The admin dashboard will automatically open at http://localhost:9000/app
```

**Option B: Contribute to This Repository** (For contributors)

```bash
# Clone the repository
git clone https://github.com/medusajs/medusa.git
cd medusa

# Install dependencies (this will take a few minutes)
yarn install

# Build all packages
yarn build

# See "Detailed Setup Guide" below for next steps
```

**Not sure which option?**
- Choose **Option A** if you want to build a store
- Choose **Option B** if you want to contribute to Medusa core

---

## Detailed Setup Guide

### Step 1: Install Prerequisites

#### Install Node.js 20+

**macOS/Linux (using nvm - recommended):**
```bash
# Install nvm if you don't have it
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install Node.js 20
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version  # Should show v20.x.x
```

**Windows:**
Download from [nodejs.org](https://nodejs.org/) (LTS version)

**Quick Check:**
```bash
node --version  # Should be v20.x.x or higher
npm --version   # Should come with Node
```

#### Install Yarn 3

```bash
# Enable Corepack (comes with Node.js 16+)
corepack enable

# Install Yarn 3
corepack prepare yarn@3.2.1 --activate

# Verify
yarn --version  # Should show 3.2.1 or higher
```

**Why Yarn 3?**
- Faster installs (like npm, but 2-3x faster)
- Better workspace management for monorepos
- Plug'n'Play mode for instant dependency resolution

#### Install PostgreSQL

**macOS (using Homebrew):**
```bash
brew install postgresql@15
brew services start postgresql@15

# Create your user (if needed)
createuser -s postgres
```

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Windows:**
Download from [postgresql.org](https://www.postgresql.org/download/windows/)

**Docker (Alternative - Quick & Isolated):**
```bash
# Run PostgreSQL in a container
docker run -d \
  --name medusa-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=medusa_db \
  -p 5432:5432 \
  postgres:15

# Verify it's running
docker ps | grep medusa-postgres
```

**Quick Check:**
```bash
psql --version  # Should show PostgreSQL 13 or higher

# Test connection (password might be empty or 'postgres')
psql -U postgres -c "SELECT version();"
```

---

### Step 2: Clone the Repository (Option B - Contributors)

```bash
# Clone the repository
git clone https://github.com/medusajs/medusa.git
cd medusa

# Verify you're on the correct branch
git branch  # Should show 'develop' or your feature branch
```

**Directory Structure Preview:**
```
medusa/
├── packages/
│   ├── medusa/           # Core backend framework
│   ├── admin/            # Admin dashboard (React)
│   ├── modules/          # Commerce modules (product, order, etc.)
│   ├── cli/              # Medusa CLI tools
│   └── core/             # Core utilities and types
├── integration-tests/    # Test suites
├── docs/                 # Documentation (you are here!)
└── package.json          # Root workspace config
```

---

### Step 3: Install Dependencies

```bash
# This will install dependencies for ALL packages in the monorepo
# ⏱️ This takes 3-5 minutes on first run
yarn install
```

**What's happening behind the scenes?**
```bash
# Yarn is installing ~100+ packages across ~80+ workspaces
# Think of it like running npm install in 80 folders at once
```

**If you see warnings:** Most are harmless (peer dependency warnings are common in monorepos)

**Quick Check:**
```bash
# Verify node_modules exists
ls -la node_modules | head -10

# Check installed packages
yarn workspaces list | head -10
```

---

### Step 4: Build All Packages

```bash
# Build all packages using Turborepo
# ⏱️ This takes 5-10 minutes on first run
yarn build
```

**What's being built?**
- TypeScript compilation (.ts → .js)
- Admin dashboard bundling (Vite)
- Module packages
- CLI tools

**Analogy:** Like running `tsc` and `webpack build` across 80+ projects simultaneously.

**Quick Check:**
```bash
# Verify dist folders were created
ls packages/medusa/dist
ls packages/admin/dashboard/dist

# You should see compiled .js files
```

---

### Step 5: Set Up PostgreSQL Database

#### Create the Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Inside psql shell:
CREATE DATABASE medusa_db;
\l  # List databases - you should see medusa_db
\q  # Quit psql
```

**Alternative (One-liner):**
```bash
createdb -U postgres medusa_db
```

**If using Docker:**
```bash
# The database is already created in the container
# No additional setup needed!
```

---

### Step 6: Configure Environment Variables

Medusa uses environment variables for configuration (like Create React App's `.env` files).

#### For Option A (New Project via create-medusa-app)

The CLI will prompt you for database credentials automatically. You can also create a `.env` file:

```bash
# Navigate to your project directory
cd my-medusa-store

# Create .env file
cat > .env << 'EOF'
# Database
DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa_db

# Authentication Secrets (IMPORTANT: Change these in production!)
JWT_SECRET=supersecret_change_in_production
COOKIE_SECRET=supersecret_change_in_production

# Server Configuration
PORT=9000
ADMIN_CORS=http://localhost:9000,http://localhost:7001
STORE_CORS=http://localhost:8000

# Admin Dashboard Path (default is /app)
ADMIN_PATH=/app
EOF
```

#### For Option B (Contributing to Repository)

Create a test project to work with:

```bash
# From the medusa repository root
cd ..

# Create a test Medusa project
npx create-medusa-app@latest my-test-store

# Link local packages (see CONTRIBUTING.md for full details)
cd my-test-store
# Edit package.json to use local file:// references
```

**Mnemonic for Required Variables (JCDD):**
- **J**WT_SECRET - For token signing
- **C**OOKIE_SECRET - For session cookies
- **D**ATABASE_URL - Connection string
- **D**on't forget PORT! (9000 by default)

**Environment Variable Deep Dive:**

| Variable | Purpose | Example | Required? |
|----------|---------|---------|-----------|
| `DATABASE_URL` | PostgreSQL connection string | `postgres://user:pass@localhost:5432/medusa_db` | ✅ Yes |
| `JWT_SECRET` | Signs authentication tokens | Any random string (32+ chars recommended) | ✅ Yes |
| `COOKIE_SECRET` | Signs session cookies | Any random string (32+ chars recommended) | ✅ Yes |
| `PORT` | Server port | `9000` (default) | ⚠️ Optional |
| `ADMIN_CORS` | Allowed admin origins | `http://localhost:9000` | ⚠️ Optional |
| `STORE_CORS` | Allowed storefront origins | `http://localhost:8000` | ⚠️ Optional |
| `NODE_ENV` | Environment mode | `development` or `production` | ⚠️ Optional |

**Generate Secure Secrets (Recommended):**
```bash
# On macOS/Linux
node -e "console.log('JWT_SECRET=' + require('crypto').randomBytes(32).toString('hex'))"
node -e "console.log('COOKIE_SECRET=' + require('crypto').randomBytes(32).toString('hex'))"

# On Windows (PowerShell)
node -e "console.log('JWT_SECRET=' + require('crypto').randomBytes(32).toString('hex'))"
```

**Quick Check:**
```bash
# Verify .env file exists
cat .env

# Check DATABASE_URL format
echo $DATABASE_URL  # Should show your connection string
```

---

### Step 7: Run Database Migrations

Migrations set up your database schema (like running `CREATE TABLE` statements, but version-controlled).

```bash
# Navigate to your Medusa project
cd my-medusa-store  # Or your project name

# Run migrations
npx medusa db:migrate

# You should see output like:
# ✔ Migrations completed
```

**What's happening?**
- Creating tables for products, orders, customers, etc.
- Setting up relationships and indexes
- Like running Prisma migrate or Sequelize sync

**Quick Check:**
```bash
# Connect to database and verify tables
psql -U postgres -d medusa_db -c "\dt"

# You should see tables like:
# - product
# - order
# - customer
# - cart
# - etc.
```

---

### Step 8: Create an Admin User

You need an admin account to access the dashboard.

```bash
# Create admin user
npx medusa user -e admin@medusa-test.com -p supersecret

# You should see:
# ✔ User created successfully!
```

**Save These Credentials:**
- **Email:** `admin@medusa-test.com`
- **Password:** `supersecret`

**Alternative: Create During Seeding**

If you used `--seed` flag with `create-medusa-app`, a default user is created:
- **Email:** `admin@medusa-test.com`
- **Password:** `supersecret`

**Quick Check:**
```bash
# Verify user in database
psql -U postgres -d medusa_db -c "SELECT email FROM user;"
```

---

### Step 9: Start the Development Server

```bash
# Start Medusa in development mode with hot-reload
npx medusa develop

# Or for production mode (no hot-reload):
# npx medusa start
```

**What's happening?**
```
1. Loading modules (cart, order, product, etc.)
2. Connecting to PostgreSQL
3. Starting Express server on port 9000
4. Compiling admin dashboard (Vite)
5. Watching for file changes (in develop mode)
```

**You should see:**
```
info:    Initializing Medusa...
info:    Database connection established
info:    Modules loaded successfully
info:    Server is ready on port 9000
info:    Admin URL → http://localhost:9000/app
```

**Difference between `develop` and `start`:**

| Command | Use Case | Hot-Reload? | Speed |
|---------|----------|-------------|-------|
| `medusa develop` | Local development | ✅ Yes | Slower startup |
| `medusa start` | Production/testing | ❌ No | Faster startup |

**Analogy:** `develop` is like `npm run dev` (with nodemon), `start` is like `npm start` (plain node).

---

## Accessing the Application

### Admin Dashboard

**URL:** http://localhost:9000/app

**Login Credentials:**
- **Email:** `admin@medusa-test.com`
- **Password:** `supersecret`

**What You'll See:**
- Dashboard with orders, products, customers
- Like WordPress admin or Shopify admin
- Built with React 18 + TailwindCSS

**Navigation Map:**
```
Admin Dashboard (/app)
├── Orders - Manage customer orders
├── Products - Product catalog
├── Customers - Customer database
├── Settings - Store configuration
└── More modules...
```

### API Endpoints

**Admin API:** http://localhost:9000/admin
**Store API:** http://localhost:9000/store

**Test the API:**
```bash
# Health check
curl http://localhost:9000/health

# Get store details (no auth required)
curl http://localhost:9000/store/products

# Admin endpoints require authentication
# Use the admin dashboard to interact with these
```

### Development URLs

| Service | URL | Purpose |
|---------|-----|---------|
| **Admin Dashboard** | http://localhost:9000/app | React admin UI |
| **Admin API** | http://localhost:9000/admin/* | Backend admin endpoints |
| **Store API** | http://localhost:9000/store/* | Frontend store endpoints |
| **Health Check** | http://localhost:9000/health | Server status |

---

## Verification Steps

### ✅ Quick Check: Everything Working?

Run these checks to verify your setup:

**1. Server Health Check**
```bash
curl http://localhost:9000/health
# Expected: {"status": "ok"}
```

**2. Database Connection**
```bash
psql -U postgres -d medusa_db -c "SELECT COUNT(*) FROM product;"
# Should show a number (0 if not seeded, >0 if seeded)
```

**3. Admin Dashboard Access**
- Open http://localhost:9000/app
- You should see a login screen
- Login with your credentials
- Dashboard should load

**4. API Response**
```bash
curl http://localhost:9000/store/regions
# Should return JSON with regions
```

**5. Check Logs**
Look for these in your terminal:
- ✅ "Server is ready on port 9000"
- ✅ "Database connection established"
- ✅ "Admin URL → http://localhost:9000/app"
- ❌ No error messages

### 🎯 Success Criteria

You're ready to move forward if:
- ✅ Server starts without errors
- ✅ You can log into the admin dashboard
- ✅ Database queries work
- ✅ API endpoints respond

---

## Common Issues & Troubleshooting

### 🔴 Issue: "Cannot connect to database"

**Error Message:**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Solutions:**

1. **Check PostgreSQL is running:**
```bash
# macOS (Homebrew)
brew services list | grep postgresql

# Linux
sudo systemctl status postgresql

# Docker
docker ps | grep postgres
```

2. **Verify connection details:**
```bash
# Test connection manually
psql -U postgres -h localhost -p 5432

# Check your DATABASE_URL in .env
cat .env | grep DATABASE_URL
```

3. **Check DATABASE_URL format:**
```bash
# Correct format:
DATABASE_URL=postgres://username:password@host:port/database

# Common mistakes:
# ❌ Missing protocol: username:password@localhost/db
# ❌ Wrong port: postgres://user:pass@localhost:3306/db
# ✅ Correct: postgres://postgres:postgres@localhost:5432/medusa_db
```

4. **Using Docker? Check host:**
```bash
# For Docker on Mac/Windows, use host.docker.internal
DATABASE_URL=postgres://postgres:postgres@host.docker.internal:5432/medusa_db

# For Docker on Linux, use 172.17.0.1 or localhost
DATABASE_URL=postgres://postgres:postgres@172.17.0.1:5432/medusa_db
```

---

### 🔴 Issue: "Port 9000 already in use"

**Error Message:**
```
Error: listen EADDRINUSE: address already in use :::9000
```

**Solutions:**

1. **Find what's using port 9000:**
```bash
# macOS/Linux
lsof -i :9000

# Windows
netstat -ano | findstr :9000
```

2. **Kill the process:**
```bash
# macOS/Linux (replace PID with actual process ID)
kill -9 <PID>

# Windows (replace PID with actual process ID)
taskkill /PID <PID> /F
```

3. **Or use a different port:**
```bash
# In .env file
PORT=9001

# Then access at http://localhost:9001/app
```

---

### 🔴 Issue: "yarn install fails" / "Cannot find module"

**Error Message:**
```
Error: Cannot find module '@medusajs/framework'
```

**Solutions:**

1. **Clear Yarn cache and reinstall:**
```bash
yarn cache clean
rm -rf node_modules
rm yarn.lock
yarn install
```

2. **Ensure you're using Yarn 3:**
```bash
yarn --version  # Should show 3.2.1+

# If not, install Yarn 3
corepack enable
corepack prepare yarn@3.2.1 --activate
```

3. **Check Node.js version:**
```bash
node --version  # Should be v20.x.x or higher

# If not, upgrade Node.js
nvm install 20
nvm use 20
```

---

### 🔴 Issue: "Migrations failed"

**Error Message:**
```
Error: Migration failed: relation "product" already exists
```

**Solutions:**

1. **Reset database (⚠️ DELETES ALL DATA):**
```bash
# Drop and recreate database
psql -U postgres -c "DROP DATABASE medusa_db;"
psql -U postgres -c "CREATE DATABASE medusa_db;"

# Run migrations again
npx medusa db:migrate
```

2. **Check migration status:**
```bash
# 🔍 NEEDS VERIFICATION - This command may not exist
# Check available commands:
npx medusa --help

# Look for migration-related commands
```

---

### 🔴 Issue: "Cannot access admin dashboard"

**Symptoms:** Page loads but shows blank screen or 404

**Solutions:**

1. **Verify admin is enabled:**
```javascript
// In medusa-config.js/ts
module.exports = defineConfig({
  admin: {
    disable: false,  // Should be false or omitted
  },
  // ...
})
```

2. **Check ADMIN_PATH:**
```bash
# Default is /app
# In .env:
ADMIN_PATH=/app

# Then access: http://localhost:9000/app
```

3. **Clear browser cache:**
```bash
# Or try incognito/private mode
# Or try different browser
```

4. **Check build output:**
```bash
# Ensure admin was built
ls packages/admin/dashboard/dist

# If empty, rebuild
yarn build
```

---

### 🔴 Issue: "Authentication errors" / "Invalid credentials"

**Solutions:**

1. **Verify user was created:**
```bash
psql -U postgres -d medusa_db -c "SELECT email, first_name FROM user;"
```

2. **Create user if missing:**
```bash
npx medusa user -e admin@medusa-test.com -p supersecret
```

3. **Check JWT_SECRET:**
```bash
# Ensure JWT_SECRET is set in .env
cat .env | grep JWT_SECRET

# If missing, add it:
echo "JWT_SECRET=$(node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")" >> .env
```

4. **Restart server after changing .env:**
```bash
# Stop server (Ctrl+C)
# Start again
npx medusa develop
```

---

### 🔴 Issue: "Slow build times"

**Solutions:**

1. **Use `yarn build` with filters:**
```bash
# Build only specific packages
yarn workspace @medusajs/medusa build
yarn workspace @medusajs/dashboard build
```

2. **Use watch mode during development:**
```bash
# Instead of yarn build, use:
yarn workspace @medusajs/medusa watch

# This rebuilds only changed files
```

3. **Increase Node.js memory (for large projects):**
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
yarn build
```

---

### 📚 More Help

**Official Resources:**
- **Documentation:** https://docs.medusajs.com/learn
- **Troubleshooting Guide:** https://docs.medusajs.com/troubleshooting
- **Discord Community:** https://discord.gg/medusajs (14,000+ members)
- **GitHub Issues:** https://github.com/medusajs/medusa/issues
- **GitHub Discussions:** https://github.com/medusajs/medusa/discussions

**In This Repository:**
- [FAQ.md](./FAQ.md) - Frequently asked questions
- [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md) - Advanced debugging
- [CONTRIBUTING.md](../../CONTRIBUTING.md) - Contributing guidelines

---

## Next Steps

### 🎓 Learning Path

Now that you have Medusa running, here's what to explore next:

**1. Understand the Architecture (30 min)**
- Read: [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
- Learn: How modules, workflows, and providers work
- Analogy: Like learning React's component model

**2. Explore the Project Structure (20 min)**
- Read: [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)
- Learn: Where to find code in the monorepo
- Benefit: Never get lost in the codebase

**3. Deep Dive into Technologies (2 hours)**
- Read: [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md)
- Learn: MikroORM, Awilix, Workflows, etc.
- When: As needed, use as reference

**4. Follow a Code Tour (30 min)**
- Read: [CODE_TOURS.md](./CODE_TOURS.md)
- Follow: Real code flows (e.g., creating a product)
- Benefit: See how everything connects

**5. Try Building Something (1-2 hours)**
- Read: [EXERCISES.md](./EXERCISES.md)
- Build: A simple feature from scratch
- Solidify: Your understanding

### 🛠️ Try These Commands

**Explore the Admin Dashboard:**
```bash
# 1. Login at http://localhost:9000/app
# 2. Create a product
# 3. View orders
# 4. Add a customer
```

**Test the API:**
```bash
# Get all products
curl http://localhost:9000/store/products

# Get regions
curl http://localhost:9000/store/regions

# For authenticated endpoints, use the admin dashboard
```

**Explore the Code:**
```bash
# View a module structure
ls -la packages/modules/product/src

# View an API route
cat packages/medusa/src/api/admin/products/route.ts

# View a workflow
ls packages/core/core-flows/src/product
```

### 📖 Recommended Reading Order

```
Day 1: Setup & Basics
├── ✅ GETTING_STARTED.md (you are here!)
├── → ARCHITECTURE_OVERVIEW.md
└── → PROJECT_STRUCTURE.md

Day 2: Deep Dive
├── → TECH_STACK_GUIDE.md
├── → DATA_FLOW_GUIDE.md
└── → FRONTEND_ARCHITECTURE.md or BACKEND_ARCHITECTURE.md

Day 3: Hands-On
├── → CODE_TOURS.md
├── → EXERCISES.md
└── → HOW_TO_GUIDE.md

Week 2: Contributing
├── → DEVELOPMENT_WORKFLOW.md
├── → PATTERNS_AND_CONVENTIONS.md
└── → FIRST_CONTRIBUTIONS.md
```

### 🎯 Your First Task (If Contributing)

1. **Pick a good first issue:**
   - Visit: https://github.com/medusajs/medusa/labels/good%20first%20issue
   - Or read: [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

2. **Set up your development workflow:**
   - Read: [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
   - Learn: Git workflow, testing, building

3. **Make your first change:**
   - Start small: Fix a typo, add a test, improve docs
   - Follow: [CONTRIBUTING.md](../../CONTRIBUTING.md)

---

## Appendix

### A. Database Connection String Format

```
postgres://[user]:[password]@[host]:[port]/[database]?[options]

Examples:
✅ postgres://postgres:postgres@localhost:5432/medusa_db
✅ postgres://admin:secret123@db.example.com:5432/production_db
✅ postgres://user:pass@localhost:5432/db?schema=public

Common Mistakes:
❌ postgresql:// (should be postgres://)
❌ Missing database name
❌ Wrong port (5432 is default for PostgreSQL)
```

### B. Environment Variable Reference

**Complete .env Template:**
```bash
# =============================================================================
# MEDUSA ENVIRONMENT CONFIGURATION
# =============================================================================

# -----------------------------------------------------------------------------
# Database Configuration
# -----------------------------------------------------------------------------
DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa_db
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_NAME=medusa_db

# -----------------------------------------------------------------------------
# Authentication & Security
# -----------------------------------------------------------------------------
# Generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
JWT_SECRET=supersecret_change_in_production
COOKIE_SECRET=supersecret_change_in_production

# -----------------------------------------------------------------------------
# Server Configuration
# -----------------------------------------------------------------------------
PORT=9000
NODE_ENV=development

# -----------------------------------------------------------------------------
# CORS Configuration
# -----------------------------------------------------------------------------
# Comma-separated list of allowed origins
ADMIN_CORS=http://localhost:9000,http://localhost:7001
STORE_CORS=http://localhost:8000
AUTH_CORS=http://localhost:9000

# -----------------------------------------------------------------------------
# Admin Dashboard
# -----------------------------------------------------------------------------
ADMIN_PATH=/app
# Set to "true" to disable admin dashboard
# ADMIN_DISABLED=false

# -----------------------------------------------------------------------------
# Redis (Optional - for caching and sessions)
# -----------------------------------------------------------------------------
# REDIS_URL=redis://localhost:6379

# -----------------------------------------------------------------------------
# Feature Flags (Optional)
# -----------------------------------------------------------------------------
# Enable experimental features
# ENABLE_INDEX_MODULE=true

# -----------------------------------------------------------------------------
# Logging
# -----------------------------------------------------------------------------
LOG_LEVEL=info  # Options: error, warn, info, debug
```

### C. Useful Commands Reference

```bash
# Package Management
yarn install              # Install all dependencies
yarn build               # Build all packages
yarn workspace @medusajs/medusa build  # Build specific package

# Development
npx medusa develop       # Start dev server with hot-reload
npx medusa start         # Start production server

# Database
npx medusa db:migrate    # Run migrations
npx medusa db:generate [name]  # Generate new migration

# User Management
npx medusa user -e <email> -p <password>  # Create admin user

# Testing (from repository root)
yarn test                # Run all tests
yarn test:integration    # Run integration tests

# CLI Help
npx medusa --help        # Show all available commands
npx medusa [command] --help  # Show help for specific command
```

### D. Port Reference

| Port | Service | URL | Configurable? |
|------|---------|-----|---------------|
| 9000 | Medusa Server + Admin | http://localhost:9000 | ✅ Yes (PORT env var) |
| 5432 | PostgreSQL | postgres://localhost:5432 | ✅ Yes (in connection string) |
| 6379 | Redis (optional) | redis://localhost:6379 | ✅ Yes (REDIS_URL env var) |
| 8000 | Storefront (if using Next.js starter) | http://localhost:8000 | ✅ Yes (in storefront config) |

### E. File Locations Reference

**Configuration Files:**
- `/home/user/medusa/medusa-config.js` or `.ts` - Main configuration
- `/home/user/medusa/.env` - Environment variables
- `/home/user/medusa/package.json` - Root dependencies

**Key Packages:**
- `/home/user/medusa/packages/medusa/` - Core backend
- `/home/user/medusa/packages/admin/dashboard/` - Admin UI
- `/home/user/medusa/packages/modules/` - Commerce modules
- `/home/user/medusa/packages/cli/medusa-cli/` - CLI tool

**Testing:**
- `/home/user/medusa/integration-tests/` - Integration tests
- `/home/user/medusa/integration-tests/.env.test` - Test environment

---

## Feedback & Questions

**Found an issue with this guide?**
- Open an issue: https://github.com/medusajs/medusa/issues
- Or fix it: Submit a PR!

**Have questions?**
- Check [FAQ.md](./FAQ.md)
- Ask in Discord: https://discord.gg/medusajs
- Start a discussion: https://github.com/medusajs/medusa/discussions

**Want to improve this guide?**
- All contributions welcome!
- See: [CONTRIBUTING.md](../../CONTRIBUTING.md)

---

**Congratulations! 🎉** You now have Medusa running locally.

**Ready for the next step?** → [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)

---

**Last Updated:** November 18, 2025
**Version:** 2.11.3
**Maintainer:** Medusa Core Team
**License:** MIT
