# Medusa Project Structure

**Version:** 2.11.3
**Last Updated:** November 18, 2025
**For:** Developers navigating the Medusa monorepo

---

## Table of Contents

1. [Monorepo Overview](#monorepo-overview)
2. [Root Directory Structure](#root-directory-structure)
3. [Packages Breakdown](#packages-breakdown)
4. [Module Anatomy](#module-anatomy)
5. [Navigation Guide by Task](#navigation-guide-by-task)
6. [Entry Points & Key Files](#entry-points--key-files)
7. [Build & Development](#build--development)

---

## Monorepo Overview

Medusa uses a **monorepo architecture** managed by:
- **Yarn 3.2.1** (Berry) - Package manager with workspaces
- **Turborepo 1.6.3** - Build orchestration and caching
- **TypeScript 5.6.2** - Type safety across all packages

🧠 **Mental Model:** Monorepo = One Git repository containing many npm packages

**Benefits:**
- ✅ Shared dependencies (TypeScript, React, testing tools)
- ✅ Atomic commits across multiple packages
- ✅ Simplified cross-package refactoring
- ✅ Single source of truth

🌉 **Bridge from JavaScript:** Think of it like a React project where every folder in `/src/components` is its own npm package.

---

## Root Directory Structure

```
/home/user/medusa/
├── .yarn/                    # Yarn 3 (Berry) configuration
│   ├── cache/                # Offline package cache
│   ├── patches/              # Custom package patches
│   └── plugins/              # Yarn plugins
├── docs/                     # Documentation
│   └── learning/             # Learning materials (you are here!)
├── integration-tests/        # Full integration test suites
│   ├── api/                  # API integration tests
│   ├── http/                 # HTTP layer tests
│   └── modules/              # Module integration tests
├── packages/                 # All publishable packages ⭐
│   ├── admin/                # Admin dashboard (React app)
│   ├── cli/                  # CLI tools
│   ├── core/                 # Core framework packages
│   ├── deps/                 # Shared dependencies wrapper
│   ├── design-system/        # UI component library
│   ├── medusa/               # Main Medusa package
│   ├── medusa-telemetry/     # Telemetry/analytics
│   ├── medusa-test-utils/    # Testing utilities
│   ├── modules/              # Commerce modules (33+ modules)
│   └── plugins/              # Official plugins
├── scripts/                  # Build and utility scripts
├── .eslintrc.js             # ESLint configuration
├── .gitignore               # Git ignore rules
├── .prettierrc              # Code formatting config
├── package.json             # Root package.json (workspaces)
├── tsconfig.json            # Root TypeScript config
├── turbo.json               # Turborepo configuration
└── yarn.lock                # Yarn lock file
```

---

## Packages Breakdown

The `/packages` directory contains all publishable npm packages.

```
/packages/
├── admin/                   # Admin Dashboard (React)
├── cli/                     # CLI Tools
├── core/                    # Core Framework Packages
├── deps/                    # Dependency Wrapper
├── design-system/           # UI Components
├── medusa/                  # Main Package
├── medusa-telemetry/        # Telemetry
├── medusa-test-utils/       # Test Utilities
├── modules/                 # Commerce Modules ⭐⭐⭐
└── plugins/                 # Official Plugins
```

---

### 1. `/packages/admin` - Admin Dashboard

**Tech Stack:** React 18.3.1, Vite 5.4.21, TanStack Query 5.64.2, Tailwind 3.4.3

```
/packages/admin/
├── admin-bundler/          # Vite bundler for admin extensions
├── admin-shared/           # Shared utilities and types
├── admin-vite-plugin/      # Vite plugin for admin
└── dashboard/              # Main admin dashboard app ⭐
    ├── src/
    │   ├── routes/         # File-based routing (35 route folders)
    │   │   ├── customers/
    │   │   ├── orders/
    │   │   ├── products/
    │   │   ├── inventory/
    │   │   └── ...
    │   ├── components/     # Reusable React components
    │   ├── hooks/          # Custom React hooks
    │   ├── providers/      # React Context providers
    │   ├── lib/            # Utility functions
    │   ├── i18n/           # Internationalization
    │   └── app.tsx         # App entry point
    ├── vite.config.ts
    └── package.json
```

**Key Files:**
- `/packages/admin/dashboard/src/main.tsx` - Entry point
- `/packages/admin/dashboard/src/app.tsx` - Root component
- `/packages/admin/dashboard/src/routes/` - All admin pages

🌉 **Bridge from React:** This is a standard Vite + React SPA with file-based routing (like Next.js App Router).

---

### 2. `/packages/cli` - Command Line Tools

**Purpose:** Development and deployment CLI commands

```
/packages/cli/
├── cli/                    # Main CLI package (@medusajs/cli)
│   ├── src/
│   │   ├── commands/       # CLI commands
│   │   │   ├── develop.ts  # medusa develop
│   │   │   ├── build.ts    # medusa build
│   │   │   ├── start.ts    # medusa start
│   │   │   └── ...
│   │   ├── loaders/        # Configuration loaders
│   │   └── util/           # Utilities
│   └── package.json
├── cli-app/                # CLI application runtime
└── oas/                    # OpenAPI Specification tools
```

**Usage:**
```bash
medusa develop     # Start dev server
medusa build       # Build for production
medusa start       # Start production server
medusa db:migrate  # Run database migrations
```

---

### 3. `/packages/core` - Core Framework Packages ⭐

**Most Important Directory** - Contains the runtime framework.

```
/packages/core/
├── framework/              # Main framework package ⭐⭐⭐
│   ├── src/
│   │   ├── http/           # Express HTTP server
│   │   ├── database/       # MikroORM configuration
│   │   ├── config/         # Configuration system
│   │   ├── logger/         # Winston logging
│   │   ├── caching/        # Cache abstraction
│   │   ├── subscribers/    # Event subscribers
│   │   ├── workflows/      # Workflow runtime
│   │   ├── links/          # Link module system
│   │   ├── jobs/           # Background jobs
│   │   ├── telemetry/      # Observability
│   │   ├── modules-sdk/    # Module SDK
│   │   ├── workflows-sdk/  # Workflow SDK
│   │   ├── orchestration/  # Orchestration engine
│   │   └── medusa-app-loader.ts  # Bootstrap
│   └── package.json
├── core-flows/             # Pre-built workflows ⭐
│   ├── src/
│   │   ├── cart/           # Cart workflows
│   │   ├── order/          # Order workflows
│   │   ├── payment/        # Payment workflows
│   │   ├── product/        # Product workflows
│   │   ├── customer/       # Customer workflows
│   │   └── ...
│   └── package.json
├── js-sdk/                 # JavaScript SDK for frontend
├── modules-sdk/            # Module development SDK
├── orchestration/          # Workflow orchestration
├── types/                  # TypeScript type definitions
├── utils/                  # Shared utilities
└── workflows-sdk/          # Workflow development SDK
```

#### Framework Exports

**File:** `/packages/core/framework/package.json`

```json
{
  "exports": {
    ".": "./dist/index.js",
    "./config": "./dist/config/index.js",
    "./caching": "./dist/caching/index.js",
    "./logger": "./dist/logger/index.js",
    "./database": "./dist/database/index.js",
    "./subscribers": "./dist/subscribers/index.js",
    "./workflows": "./dist/workflows/index.js",
    "./links": "./dist/links/index.js",
    "./jobs": "./dist/jobs/index.js",
    "./http": "./dist/http/index.js",
    "./workflows-sdk": "./dist/workflows-sdk/index.js",
    "./modules-sdk": "./dist/modules-sdk/index.js"
  }
}
```

**Usage:**
```typescript
// Import from framework sub-exports
import { createWorkflow } from "@medusajs/framework/workflows-sdk"
import { defineLink } from "@medusajs/framework/links"
import { AuthenticatedMedusaRequest } from "@medusajs/framework/http"
import { MedusaApp } from "@medusajs/framework"
```

---

### 4. `/packages/modules` - Commerce Modules ⭐⭐⭐

**The Heart of Medusa** - 33+ isolated commerce modules.

```
/packages/modules/
├── cart/                   # Shopping cart
├── product/                # Product catalog
├── order/                  # Order management
├── payment/                # Payment processing
├── pricing/                # Price calculation
├── promotion/              # Discounts & promotions
├── customer/               # Customer management
├── auth/                   # Authentication
├── user/                   # User accounts
├── inventory/              # Stock management
├── fulfillment/            # Shipping & delivery
├── tax/                    # Tax calculation
├── region/                 # Geographic regions
├── currency/               # Multi-currency
├── sales-channel/          # Multi-channel sales
├── store/                  # Store configuration
├── stock-location/         # Warehouse locations
├── file/                   # File storage
├── notification/           # Notifications
├── api-key/                # API key management
├── analytics/              # Analytics tracking
├── settings/               # System settings
├── locking/                # Distributed locking
├── cache-inmemory/         # In-memory cache
├── cache-redis/            # Redis cache
├── event-bus-local/        # Local event bus
├── event-bus-redis/        # Redis event bus
├── workflow-engine-inmemory/  # In-memory workflows
├── workflow-engine-redis/     # Redis workflows
├── link-modules/           # Cross-module relationships
├── index/                  # Search indexing
└── providers/              # Provider implementations
    ├── payment-stripe/
    ├── payment-paypal/
    ├── file-s3/
    ├── file-local/
    ├── notification-sendgrid/
    └── ...
```

---

### 5. `/packages/medusa` - Main Application Package

**Purpose:** The main Medusa application package that ties everything together.

```
/packages/medusa/
├── src/
│   ├── api/                # API route handlers ⭐
│   │   ├── admin/          # Admin API routes
│   │   │   ├── products/
│   │   │   ├── orders/
│   │   │   ├── customers/
│   │   │   └── ...
│   │   ├── store/          # Storefront API routes
│   │   │   ├── carts/
│   │   │   ├── products/
│   │   │   ├── customers/
│   │   │   └── ...
│   │   ├── auth/           # Authentication routes
│   │   ├── hooks/          # Workflow hooks
│   │   ├── middlewares.ts  # Global middlewares
│   │   └── utils/          # API utilities
│   ├── loaders/            # Application loaders
│   ├── subscribers/        # Event subscribers
│   └── types/              # Type definitions
├── package.json
└── README.md
```

#### API Route Structure (File-Based Routing)

Medusa uses **file-based routing** like Next.js:

```
/packages/medusa/src/api/store/carts/
├── route.ts                    # GET/POST /store/carts
├── [id]/
│   ├── route.ts                # GET/PATCH/DELETE /store/carts/:id
│   ├── line-items/
│   │   └── route.ts            # POST /store/carts/:id/line-items
│   └── shipping-methods/
│       └── route.ts            # POST /store/carts/:id/shipping-methods
└── middlewares.ts              # Route-specific middlewares
```

**Route Example:** `/packages/medusa/src/api/store/carts/route.ts`

```typescript
import { createCartWorkflow } from "@medusajs/core-flows"
import {
  AuthenticatedMedusaRequest,
  MedusaResponse,
} from "@medusajs/framework/http"

// POST /store/carts
export const POST = async (
  req: AuthenticatedMedusaRequest<HttpTypes.StoreCreateCart>,
  res: MedusaResponse<HttpTypes.StoreCartResponse>
) => {
  const { result } = await createCartWorkflow(req.scope).run({
    input: {
      ...req.validatedBody,
      customer_id: req.auth_context?.actor_id,
    },
  })

  const cart = await refetchCart(result.id, req.scope, req.queryConfig.fields)
  res.status(200).json({ cart })
}
```

💡 **Aha Moment:** Each exported function name (`GET`, `POST`, `PATCH`, `DELETE`) maps directly to HTTP methods!

---

### 6. `/packages/design-system` - UI Component Library

**Purpose:** Shared UI components for admin dashboard.

```
/packages/design-system/
├── icons/                  # Icon components
├── ui/                     # UI components
│   ├── src/
│   │   ├── components/     # React components (Radix UI based)
│   │   │   ├── button/
│   │   │   ├── input/
│   │   │   ├── select/
│   │   │   ├── modal/
│   │   │   └── ...
│   │   └── theme/          # Tailwind theme
│   └── package.json
```

**Tech:** Radix UI primitives + Tailwind CSS

---

## Module Anatomy

Every module follows the **same structure** for consistency.

### Standard Module Structure

```
/packages/modules/{module-name}/
├── src/
│   ├── index.ts              # Module registration ⭐
│   ├── models/               # Database entities (MikroORM) ⭐
│   │   ├── {entity}.ts
│   │   └── index.ts
│   ├── services/             # Business logic ⭐
│   │   ├── {module}-service.ts
│   │   └── index.ts
│   ├── migrations/           # Database migrations
│   │   └── Migration{timestamp}.ts
│   ├── types/                # TypeScript types
│   │   └── index.ts
│   └── loaders/              # Module initialization (optional)
│       └── index.ts
├── integration-tests/        # Integration tests
├── package.json
└── README.md
```

### Real Example: Cart Module

**File Tree:**
```
/packages/modules/cart/
├── src/
│   ├── index.ts                        # ⭐ Module registration
│   ├── models/
│   │   ├── cart.ts                     # Cart entity
│   │   ├── line-item.ts                # Line item entity
│   │   ├── address.ts                  # Address entity
│   │   ├── shipping-method.ts          # Shipping method
│   │   └── index.ts
│   ├── services/
│   │   ├── cart-module.ts              # ⭐ Main service
│   │   └── index.ts
│   ├── migrations/
│   │   ├── Migration20240831125857.ts
│   │   └── ...
│   └── types/
│       └── index.ts
├── integration-tests/
├── package.json
└── README.md
```

#### Module Entry Point

**File:** `/packages/modules/cart/src/index.ts`

```typescript
import { Module, Modules } from "@medusajs/framework/utils"
import { CartModuleService } from "./services"

// Register module with framework
export default Module(Modules.CART, {
  service: CartModuleService,
})
```

**What this does:**
1. Registers the module under `Modules.CART` identifier
2. Specifies `CartModuleService` as the main service
3. Makes the module available for dependency injection

#### Module Service

**File:** `/packages/modules/cart/src/services/cart-module.ts`

```typescript
import { MedusaService } from "@medusajs/framework/utils"
import { Cart, LineItem } from "../models"

class CartModuleService extends MedusaService({
  Cart,
  LineItem,
  // ... other models
}) {
  // Custom methods
  async addLineItem(cartId: string, item: AddLineItemInput) {
    // Business logic here
  }

  async removeLineItem(cartId: string, lineItemId: string) {
    // Business logic here
  }
}

export default CartModuleService
```

🧠 **Mental Model:** Module Service = Repository + Business Logic
- Like a React custom hook that handles data fetching and state updates
- Extends `MedusaService` which provides CRUD operations
- Add custom methods for business logic

#### Module Models

**File:** `/packages/modules/cart/src/models/cart.ts`

```typescript
import { model } from "@medusajs/framework/utils"

const Cart = model.define("cart", {
  id: model.id().primaryKey(),
  email: model.text().nullable(),
  currency_code: model.text(),
  region_id: model.text(),

  // Relations
  items: model.hasMany(() => LineItem, { mappedBy: "cart" }),
  region: model.belongsTo(() => Region),
  customer: model.belongsTo(() => Customer),
})

export default Cart
```

🌉 **Bridge from TypeScript:** Model definitions = TypeScript interfaces + database schema + ORM relations combined.

---

## Navigation Guide by Task

### Task: Add a new API endpoint

**Path:** `/packages/medusa/src/api/{scope}/{resource}/`

1. Navigate to `/packages/medusa/src/api/`
2. Choose scope: `admin/` or `store/`
3. Find or create resource folder (e.g., `products/`)
4. Create `route.ts` with exported HTTP methods

**Example:** Add `GET /store/products/:id/reviews`

```
/packages/medusa/src/api/store/products/[id]/reviews/route.ts
```

```typescript
export const GET = async (req, res) => {
  const { id } = req.params
  // Fetch reviews for product
}
```

---

### Task: Create a new workflow

**Path:** `/packages/core/core-flows/src/{domain}/workflows/`

1. Navigate to `/packages/core/core-flows/src/`
2. Find domain folder (e.g., `cart/`, `order/`)
3. Create workflow in `workflows/` subfolder
4. Create steps in `steps/` subfolder

**Example Structure:**
```
/packages/core/core-flows/src/cart/
├── workflows/
│   └── add-custom-item.ts       # New workflow
└── steps/
    └── validate-custom-item.ts  # New step
```

---

### Task: Extend a module

**Path:** Create custom module in your project, or extend existing module

**Option 1:** Extend in your project
```
your-medusa-project/
├── src/
│   └── modules/
│       └── custom-cart/          # Custom module
│           ├── index.ts
│           ├── models/
│           └── services/
```

**Option 2:** Modify existing module (not recommended for production)
```
/packages/modules/cart/src/services/cart-module.ts
```

---

### Task: Add a new commerce module

**Path:** `/packages/modules/{new-module}/`

1. Create module directory: `/packages/modules/my-module/`
2. Follow standard module structure (see Module Anatomy)
3. Register in `/packages/modules/index/src/index.ts`
4. Add to workspace in root `package.json`

---

### Task: Customize admin dashboard

**Path:** `/packages/admin/dashboard/src/routes/`

1. Navigate to `/packages/admin/dashboard/src/`
2. Add routes in `routes/` folder
3. Add components in `components/`
4. Add hooks in `hooks/`

**Example:** Add custom product tab
```
/packages/admin/dashboard/src/routes/products/[id]/custom-tab/
└── page.tsx
```

---

### Task: Add event subscriber

**Path:** `/packages/medusa/src/subscribers/`

1. Navigate to `/packages/medusa/src/subscribers/`
2. Create new subscriber file

**Example:** `/packages/medusa/src/subscribers/cart-created.ts`

```typescript
import { SubscriberArgs } from "@medusajs/framework"

export default async function handleCartCreated({
  event,
  container,
}: SubscriberArgs<{ id: string }>) {
  const logger = container.resolve("logger")
  logger.info(`Cart created: ${event.data.id}`)
}

export const config = {
  event: "cart.created",
}
```

---

### Task: Configure Medusa

**Path:** Root of your Medusa project (not in monorepo)

```
your-medusa-project/
├── medusa-config.ts        # Main configuration ⭐
├── .env                    # Environment variables
└── src/
    └── ...                 # Your customizations
```

**Example:** `medusa-config.ts`

```typescript
import { defineConfig } from "@medusajs/framework"

export default defineConfig({
  projectConfig: {
    databaseUrl: process.env.DATABASE_URL,
    http: {
      storeCors: process.env.STORE_CORS,
      adminCors: process.env.ADMIN_CORS,
    },
  },
  modules: {
    payment: {
      resolve: "@medusajs/payment-stripe",
      options: {
        apiKey: process.env.STRIPE_API_KEY,
      },
    },
  },
})
```

---

## Entry Points & Key Files

### Framework Entry Point

**File:** `/packages/core/framework/src/medusa-app-loader.ts`

This file bootstraps the entire Medusa application:

1. Loads configuration from `medusa-config.ts`
2. Initializes dependency injection container
3. Loads all modules
4. Starts HTTP server
5. Runs database migrations
6. Initializes event bus

### HTTP Server Entry Point

**File:** `/packages/core/framework/src/http/index.ts`

Initializes Express server with:
- CORS middleware
- Authentication middleware
- Request validation
- File-based route loading
- Error handling

### Module Loading

**File:** `/packages/core/framework/src/modules-sdk/index.ts`

Handles:
- Module discovery
- Service registration in DI container
- Database connection per module
- Migration execution

### Workflow Engine Entry Point

**File:** `/packages/core/orchestration/src/index.ts`

Manages:
- Workflow registration
- Step execution
- Compensation (rollback) logic
- Workflow state persistence

---

## Build & Development

### Build System

**Root:** `turbo.json`

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": []
    }
  }
}
```

**Build Commands:**

```bash
# Build all packages
yarn build

# Build specific package
yarn workspace @medusajs/framework build

# Watch mode for development
yarn workspace @medusajs/framework watch
```

### Package Scripts

**File:** Root `package.json`

```json
{
  "scripts": {
    "build": "turbo run build --concurrency=100% --no-daemon",
    "test": "turbo run test --no-daemon --no-cache --force",
    "lint": "eslint --ignore-path .eslintignore --ext .js,.ts,.tsx .",
    "prettier": "prettier"
  }
}
```

### Workspace Configuration

**File:** Root `package.json`

```json
{
  "workspaces": {
    "packages": [
      "packages/medusa",
      "packages/medusa-test-utils",
      "packages/modules/*",
      "packages/modules/providers/*",
      "packages/plugins/*",
      "packages/core/*",
      "packages/framework/*",
      "packages/cli/*",
      "packages/admin/*",
      "packages/design-system/*"
    ]
  }
}
```

💡 **Aha Moment:** Workspaces allow you to `import "@medusajs/framework"` from any package without publishing to npm!

---

## Development Workflow

### 1. Working on Core Framework

```bash
cd /home/user/medusa/packages/core/framework
yarn watch  # Auto-rebuild on changes
```

### 2. Working on a Module

```bash
cd /home/user/medusa/packages/modules/cart
yarn test   # Run tests
yarn build  # Build module
```

### 3. Working on Admin Dashboard

```bash
cd /home/user/medusa/packages/admin/dashboard
yarn dev    # Start Vite dev server
```

### 4. Running Integration Tests

```bash
cd /home/user/medusa/integration-tests/api
yarn test   # Run API integration tests
```

### 5. Building Everything

```bash
cd /home/user/medusa
yarn build  # Turbo builds all packages in correct order
```

---

## Key Takeaways

### ✅ Remember These Paths

1. **Framework code:** `/packages/core/framework/src/`
2. **Commerce modules:** `/packages/modules/{module-name}/`
3. **Pre-built workflows:** `/packages/core/core-flows/src/`
4. **API routes:** `/packages/medusa/src/api/`
5. **Admin dashboard:** `/packages/admin/dashboard/src/`
6. **Configuration:** `medusa-config.ts` (in your project, not monorepo)

### 🎯 Navigation Shortcuts

| What You Want | Where to Go |
|---------------|-------------|
| Add API endpoint | `/packages/medusa/src/api/{scope}/{resource}/` |
| Create workflow | `/packages/core/core-flows/src/{domain}/workflows/` |
| Modify module | `/packages/modules/{module}/src/` |
| Admin UI page | `/packages/admin/dashboard/src/routes/` |
| Event subscriber | `/packages/medusa/src/subscribers/` |
| Type definitions | `/packages/core/types/src/` |

### 🔍 Finding Code

**Use glob patterns:**
```bash
# Find all route files
find . -name "route.ts"

# Find all workflow files
find . -path "*/workflows/*.ts"

# Find all module services
find . -path "*/modules/*/src/services/*.ts"
```

---

## Next Steps

1. Explore `/packages/modules/cart/` to see a complete module
2. Look at `/packages/core/core-flows/src/cart/workflows/create-carts.ts` for workflow example
3. Check `/packages/medusa/src/api/store/carts/route.ts` for API route example
4. Read `TECH_STACK_GUIDE.md` for technology details
5. Read `DATA_FLOW_GUIDE.md` for request flow examples

---

**File Path:** `/home/user/medusa/docs/learning/PROJECT_STRUCTURE.md`
**Related Files:**
- `ARCHITECTURE_OVERVIEW.md` - System architecture
- `TECH_STACK_GUIDE.md` - Technology deep dive
- `DATA_FLOW_GUIDE.md` - Request flow examples
