# Frequently Asked Questions (FAQ)

**Quick answers to common Medusa development questions**

**Last Updated:** November 19, 2025
**Version:** 2.11.3

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development](#development)
3. [Architecture](#architecture)
4. [API](#api)
5. [Database](#database)
6. [Frontend](#frontend)
7. [Testing](#testing)
8. [Deployment](#deployment)
9. [Contributing](#contributing)
10. [Troubleshooting](#troubleshooting)

---

## Getting Started

### Q: How do I install Medusa?

**A:** There are two ways:

**Option 1: Create a new project (recommended for new users)**
```bash
npx create-medusa-app@latest
```

**Option 2: Clone the repository (for contributors)**
```bash
git clone https://github.com/medusajs/medusa.git
cd medusa
yarn install
yarn build
```

See `/home/user/medusa/docs/learning/GETTING_STARTED.md` for detailed instructions.

---

### Q: What are the system requirements?

**A:** You need:
- **Node.js:** 20+ (LTS recommended)
- **Yarn:** 3.2.1+
- **PostgreSQL:** 13+
- **RAM:** 8GB minimum, 16GB recommended
- **OS:** macOS, Linux, or WSL2 on Windows

Check versions:
```bash
node --version  # Should be 20+
yarn --version  # Should be 3.2.1+
psql --version  # Should be 13+
```

---

### Q: Port 9000 is already in use. What do I do?

**A:** Change the port in your configuration:

```typescript
// medusa-config.ts
export default defineConfig({
  projectConfig: {
    http: {
      port: 9001, // Changed from 9000
    },
  },
})
```

Or kill the process using port 9000:
```bash
# Find process
lsof -i :9000

# Kill it
kill -9 <PID>
```

---

### Q: Database connection failed. How do I fix it?

**A:** Check your database configuration:

1. **Verify PostgreSQL is running:**
   ```bash
   psql -U postgres -c "SELECT 1"
   ```

2. **Check connection string:**
   ```typescript
   // medusa-config.ts
   databaseUrl: "postgresql://user:password@localhost:5432/medusa"
   ```

3. **Create database if it doesn't exist:**
   ```bash
   createdb medusa
   ```

4. **Test connection:**
   ```bash
   psql -d medusa -c "SELECT version();"
   ```

---

### Q: I get "module not found" errors. What's wrong?

**A:** Usually a build or dependency issue:

```bash
# Clean install
rm -rf node_modules
yarn cache clean
yarn install

# Rebuild
yarn build

# If still failing, check:
ls node_modules/@medusajs  # Should see packages
```

---

### Q: How do I seed test data?

**A:** Use the seed script:

```bash
yarn seed

# Or manually:
psql -d medusa -f seed-data.sql
```

This creates sample products, customers, and orders for testing.

---

## Development

### Q: How do I add a new field to a model?

**A:** Follow these steps:

1. **Update the model:**
   ```typescript
   // In your model file
   const Product = model.define("product", {
     // ... existing fields
     custom_field: model.text().nullable(),
   })
   ```

2. **Generate migration:**
   ```bash
   yarn medusa db:generate product
   ```

3. **Run migration:**
   ```bash
   yarn medusa db:migrate product
   ```

4. **Verify:**
   ```bash
   psql -d medusa -c "\d product"
   ```

See `/home/user/medusa/docs/learning/EXERCISES.md` Exercise 7 for detailed guide.

---

### Q: How do I create a database migration?

**A:** Migrations are auto-generated from model changes:

```bash
# 1. Modify your model
# 2. Generate migration
yarn medusa db:generate <module-name>

# 3. Review the generated migration file
# 4. Run migration
yarn medusa db:migrate <module-name>

# 5. Rollback if needed
yarn medusa db:rollback <module-name>
```

**Manual migration:**
```typescript
import { Migration } from '@mikro-orm/migrations'

export class Migration20251119000000 extends Migration {
  async up(): Promise<void> {
    this.addSql('ALTER TABLE product ADD COLUMN new_field text;')
  }

  async down(): Promise<void> {
    this.addSql('ALTER TABLE product DROP COLUMN new_field;')
  }
}
```

---

### Q: How do I debug Medusa?

**A:** Several debugging options:

**1. Console logging:**
```typescript
console.log("Debug:", value)
```

**2. Logger service:**
```typescript
const logger = container.resolve("logger")
logger.info("Info message")
logger.error("Error message")
logger.debug("Debug message")
```

**3. VS Code debugger:**
```json
// .vscode/launch.json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Medusa",
  "program": "${workspaceFolder}/packages/medusa/dist/bin/medusa.js",
  "args": ["develop"],
  "console": "integratedTerminal"
}
```

**4. Chrome DevTools:**
```bash
node --inspect packages/medusa/dist/bin/medusa.js develop
# Open chrome://inspect
```

See `/home/user/medusa/docs/learning/DEBUGGING_GUIDE.md` for more.

---

### Q: Hot reload isn't working. Why?

**A:** Common causes:

1. **Not in dev mode:**
   ```bash
   yarn dev  # Use this, not yarn start
   ```

2. **Build required first:**
   ```bash
   yarn build
   yarn dev
   ```

3. **File not watched:**
   Only files in `packages/medusa/src` are watched. Changes in `node_modules` or `dist` won't trigger reload.

4. **Clear cache:**
   ```bash
   rm -rf .medusa
   yarn dev
   ```

---

### Q: Build errors - "Cannot find module" or type errors?

**A:** Try this sequence:

```bash
# 1. Clean everything
yarn clean
rm -rf node_modules

# 2. Reinstall
yarn install

# 3. Build dependencies first
cd packages/core
yarn build
cd ../..

# 4. Build all
yarn build

# 5. Check for TypeScript errors
yarn type-check
```

---

### Q: How do I run a specific test?

**A:** Use Jest's file pattern:

```bash
# Run all tests
yarn test

# Run specific file
yarn test product.spec.ts

# Run tests matching pattern
yarn test product

# Run single test case
yarn test -t "should create product"

# Run in watch mode
yarn test --watch

# Run with coverage
yarn test --coverage
```

---

## Architecture

### Q: What is a module?

**A:** A module is a self-contained unit of functionality in Medusa. Think of it like a microservice, but in a monolith.

**Structure:**
```
module/
├── src/
│   ├── models/       # Data models
│   ├── services/     # Business logic
│   ├── migrations/   # Database changes
│   └── index.ts      # Module definition
```

**Example:**
```typescript
// Product module provides product functionality
const productService = container.resolve("productModuleService")
const products = await productService.listProducts()
```

**Key modules:**
- `product` - Products and variants
- `order` - Orders and line items
- `customer` - Customer data
- `cart` - Shopping carts
- `payment` - Payment processing

See `/home/user/medusa/docs/learning/BACKEND_ARCHITECTURE.md` for details.

---

### Q: How do workflows work?

**A:** Workflows orchestrate complex operations with automatic rollback.

**Concept:**
```
Step 1 → Step 2 → Step 3
  ↓        ↓        ↓
  ✓        ✓        ✗ (fails)
  ↓        ↓
Rollback  Rollback
```

**Example:**
```typescript
const workflow = createWorkflow("create-order", (input) => {
  const cart = getCartStep(input.cartId)
  const order = createOrderStep(cart)
  const payment = processPaymentStep(order)
  return new WorkflowResponse({ order })
})

// If payment fails, order is automatically deleted
```

**Key concepts:**
- **Steps:** Individual operations
- **Compensation:** Undo logic for rollback
- **Hooks:** Before/after execution
- **Parallel execution:** Steps can run concurrently

See `/home/user/medusa/docs/learning/BACKEND_ARCHITECTURE.md` section on workflows.

---

### Q: What is dependency injection?

**A:** DI provides services to your code automatically.

**Traditional approach:**
```typescript
// Manual dependency management (bad)
const db = new Database()
const service = new ProductService(db)
```

**With DI:**
```typescript
// Container provides dependencies (good)
const productService = container.resolve("productModuleService")
```

**Benefits:**
- Easier testing (mock dependencies)
- Loose coupling
- Better organization
- Automatic lifecycle management

**In Medusa:**
```typescript
// In API route
export async function GET(req: MedusaRequest, res: MedusaResponse) {
  // Services injected via req.scope
  const productService = req.scope.resolve("productModuleService")
  const orderService = req.scope.resolve("orderModuleService")
}
```

---

### Q: How do modules communicate?

**A:** Through link modules and the module registry.

**Direct access:**
```typescript
const productService = container.resolve("productModuleService")
const product = await productService.retrieveProduct(id)
```

**Link modules:**
```typescript
// product_sales_channel links products to sales channels
const remoteLink = container.resolve("remoteLink")
await remoteLink.create({
  productService: { product_id: "prod_123" },
  salesChannelService: { sales_channel_id: "sc_123" },
})
```

**Events:**
```typescript
// Emit event
eventBus.emit("product.created", { id: product.id })

// Subscribe to event
export default async function handler({ event, container }) {
  // React to product creation
}
```

---

### Q: What are link modules?

**A:** Link modules create relationships between modules.

**Purpose:** Connect data across modules without tight coupling.

**Example:**
```typescript
// Link product to region (for availability)
product_region link table:
  product_id → references product module
  region_id  → references region module

// Query products in a region
const remoteQuery = container.resolve("remoteQuery")
const products = await remoteQuery({
  product: {
    fields: ["id", "title"],
    region: {
      fields: ["name"],
    },
  },
  filters: {
    region_id: "region_123",
  },
})
```

**Common link modules:**
- `product_sales_channel`
- `product_variant_inventory_item`
- `cart_payment_collection`

---

## API

### Q: How do I authenticate API requests?

**A:** Depends on the endpoint type.

**Admin API:**
```bash
# 1. Login to get token
TOKEN=$(curl -X POST http://localhost:9000/auth/user/emailpass \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@medusa-test.com",
    "password": "supersecret"
  }' | jq -r '.token')

# 2. Use token in requests
curl http://localhost:9000/admin/products \
  -H "Authorization: Bearer $TOKEN"
```

**Store API:**
```bash
# Most endpoints are public
curl http://localhost:9000/store/products

# Customer endpoints need auth
curl http://localhost:9000/store/customers/me \
  -H "Authorization: Bearer $CUSTOMER_TOKEN"
```

**API Keys (for integrations):**
```bash
# Create API key in admin
# Use in header
curl http://localhost:9000/admin/products \
  -H "x-publishable-api-key: pk_..."
```

---

### Q: How do I paginate API results?

**A:** Use `limit` and `offset` parameters:

```bash
# First page (20 items)
curl "http://localhost:9000/admin/products?limit=20&offset=0"

# Second page (items 21-40)
curl "http://localhost:9000/admin/products?limit=20&offset=20"

# Third page (items 41-60)
curl "http://localhost:9000/admin/products?limit=20&offset=40"
```

**In code:**
```typescript
const pageSize = 20
const page = 2
const products = await productService.listProducts(
  {}, // filters
  {
    take: pageSize,
    skip: (page - 1) * pageSize,
  }
)
```

**Response includes count:**
```json
{
  "products": [...],
  "count": 150,
  "limit": 20,
  "offset": 20
}
```

---

### Q: How do I filter and search?

**A:** Use query parameters:

```bash
# Filter by status
curl "http://localhost:9000/admin/products?status=published"

# Search by title
curl "http://localhost:9000/admin/products?q=shirt"

# Multiple filters
curl "http://localhost:9000/admin/products?status=published&q=shirt"

# Filter by date
curl "http://localhost:9000/admin/orders?created_at[gte]=2025-01-01"

# Filter by array
curl "http://localhost:9000/admin/products?id[]=prod_123&id[]=prod_456"
```

**In service:**
```typescript
const products = await productService.listProducts({
  status: "published",
  title: { $ilike: "%shirt%" },
  created_at: { $gte: new Date("2025-01-01") },
})
```

---

### Q: I'm getting CORS errors. How do I fix them?

**A:** Configure CORS in your config:

```typescript
// medusa-config.ts
export default defineConfig({
  projectConfig: {
    http: {
      cors: {
        // Allow all origins (development only!)
        origins: "*",

        // Or specific origins
        origins: [
          "http://localhost:3000",
          "http://localhost:8000",
          "https://my-store.com",
        ],

        // Allow credentials
        credentials: true,
      },
    },
  },
})
```

**Restart server after changing:**
```bash
yarn dev
```

---

### Q: Is there rate limiting on the API?

**A:** By default, no. But you can add it:

```typescript
// Custom middleware
import rateLimit from "express-rate-limit"

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
})

// Apply to routes
app.use("/store/", limiter)
```

**For production, use:**
- Nginx rate limiting
- CloudFlare
- API Gateway with rate limits

---

## Database

### Q: How do I query the database?

**A:** Use services or direct SQL.

**Using services (recommended):**
```typescript
const productService = container.resolve("productModuleService")

// List with filters
const products = await productService.listProducts({
  status: "published",
})

// Get one
const product = await productService.retrieveProduct("prod_123")

// With relations
const product = await productService.retrieveProduct("prod_123", {
  relations: ["variants", "images"],
})
```

**Using MikroORM:**
```typescript
const manager = container.resolve("manager")
const products = await manager.find(Product, {
  status: "published",
})
```

**Direct SQL (use sparingly):**
```typescript
const manager = container.resolve("manager")
const result = await manager.execute(
  "SELECT * FROM product WHERE status = $1",
  ["published"]
)
```

---

### Q: Migration failed. How do I rollback?

**A:** Use the rollback command:

```bash
# Rollback last migration
yarn medusa db:rollback <module-name>

# Rollback specific migration
yarn medusa db:rollback <module-name> --to=<migration-name>

# Check migration status
yarn medusa db:migrations:list <module-name>
```

**Manual rollback:**
```sql
-- Check migration table
SELECT * FROM mikro_orm_migrations ORDER BY executed_at DESC;

-- Manually run down() migration
-- (see the migration file)
```

**Fix and rerun:**
```bash
# Fix migration file
# Run again
yarn medusa db:migrate <module-name>
```

---

### Q: How do I handle relationships?

**A:** Use MikroORM relationships in models:

**One-to-Many:**
```typescript
// Product has many variants
const Product = model.define("product", {
  id: model.id().primaryKey(),
  variants: model.hasMany(() => ProductVariant),
})

const ProductVariant = model.define("product_variant", {
  id: model.id().primaryKey(),
  product: model.belongsTo(() => Product),
})
```

**Many-to-Many:**
```typescript
// Product belongs to many categories
const Product = model.define("product", {
  categories: model.manyToMany(() => ProductCategory),
})

const ProductCategory = model.define("product_category", {
  products: model.manyToMany(() => Product),
})
```

**Querying with relations:**
```typescript
const product = await productService.retrieveProduct("prod_123", {
  relations: ["variants", "categories"],
})

console.log(product.variants) // Array of variants
console.log(product.categories) // Array of categories
```

---

### Q: Database is slow. How do I optimize?

**A:** Several optimization strategies:

**1. Add indexes:**
```typescript
const Product = model.define("product", {
  // ... fields
}).indexes([
  {
    name: "IDX_product_status",
    on: ["status"],
  },
  {
    name: "IDX_product_created_at",
    on: ["created_at"],
  },
])
```

**2. Use pagination:**
```typescript
// Don't load all records
const products = await productService.listProducts(
  {},
  { take: 20, skip: 0 }
)
```

**3. Select only needed fields:**
```typescript
const products = await productService.listProducts(
  {},
  { select: ["id", "title", "handle"] }
)
```

**4. Analyze slow queries:**
```sql
-- Enable query logging
SET log_statement = 'all';

-- Analyze query
EXPLAIN ANALYZE
SELECT * FROM product WHERE status = 'published';
```

---

## Frontend

### Q: How do I add a page to the admin dashboard?

**A:** Create a page file in the admin routes:

```tsx
// packages/admin/dashboard/src/routes/my-page/page.tsx
import { Container, Heading } from "@medusajs/ui"

export default function MyPage() {
  return (
    <Container>
      <Heading level="h1">My Custom Page</Heading>
      <p>Content goes here</p>
    </Container>
  )
}
```

**Add navigation:**
```tsx
// packages/admin/dashboard/src/routes/my-page/my-page-nav.tsx
import { defineRouteConfig } from "@medusajs/admin-sdk"
import { Sparkles } from "@medusajs/icons"

export default defineRouteConfig({
  label: "My Page",
  icon: Sparkles,
})
```

See `/home/user/medusa/docs/learning/EXERCISES.md` Exercise 9 for detailed guide.

---

### Q: How do I fetch data in the admin UI?

**A:** Use fetch or TanStack Query:

**Basic fetch:**
```tsx
import { useEffect, useState } from "react"

export default function ProductList() {
  const [products, setProducts] = useState([])

  useEffect(() => {
    fetch("/admin/products", { credentials: "include" })
      .then(res => res.json())
      .then(data => setProducts(data.products))
  }, [])

  return <div>{/* Render products */}</div>
}
```

**With TanStack Query (recommended):**
```tsx
import { useQuery } from "@tanstack/react-query"

export default function ProductList() {
  const { data, isLoading } = useQuery({
    queryKey: ["products"],
    queryFn: async () => {
      const res = await fetch("/admin/products", {
        credentials: "include",
      })
      return res.json()
    },
  })

  if (isLoading) return <div>Loading...</div>

  return <div>{/* Render data.products */}</div>
}
```

---

### Q: How do I validate forms?

**A:** Use Zod with React Hook Form:

```tsx
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"

const schema = z.object({
  title: z.string().min(3, "Title must be at least 3 characters"),
  price: z.number().positive("Price must be positive"),
})

export default function ProductForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  })

  const onSubmit = (data) => {
    // Submit data
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("title")} />
      {errors.title && <span>{errors.title.message}</span>}

      <input {...register("price", { valueAsNumber: true })} />
      {errors.price && <span>{errors.price.message}</span>}

      <button type="submit">Submit</button>
    </form>
  )
}
```

---

### Q: Routing isn't working. What's wrong?

**A:** Check file-based routing structure:

**Correct structure:**
```
routes/
├── products/
│   ├── page.tsx          # /products
│   └── [id]/
│       └── page.tsx      # /products/:id
├── orders/
│   └── page.tsx          # /orders
└── settings/
    └── page.tsx          # /settings
```

**Common mistakes:**
```
routes/
├── products.tsx          # ❌ Wrong - needs to be products/page.tsx
└── product-detail.tsx    # ❌ Wrong - use [id]/page.tsx
```

**Dynamic routes:**
```tsx
// routes/products/[id]/page.tsx
export default function ProductDetail({ params }) {
  const { id } = params
  // Fetch product with id
}
```

---

## Testing

### Q: How do I run tests?

**A:** Use Jest commands:

```bash
# All tests
yarn test

# Specific module
cd packages/core/core-flows
yarn test

# Specific file
yarn test product.spec.ts

# Watch mode
yarn test --watch

# Coverage
yarn test --coverage

# Update snapshots
yarn test -u
```

---

### Q: How do I mock dependencies in tests?

**A:** Use Jest mocks:

```typescript
import { ProductService } from "../services/product"

// Mock the service
jest.mock("../services/product")

describe("Product workflow", () => {
  it("should create product", async () => {
    // Setup mock
    const mockCreate = jest.fn().mockResolvedValue({
      id: "prod_123",
      title: "Test Product",
    })

    ProductService.prototype.createProduct = mockCreate

    // Test code that uses ProductService
    const result = await createProductWorkflow.run({
      input: { title: "Test Product" },
    })

    expect(mockCreate).toHaveBeenCalledWith({
      title: "Test Product",
    })
    expect(result.id).toBe("prod_123")
  })
})
```

---

### Q: How do I set up a test database?

**A:** Use a separate test database:

```typescript
// jest.config.js
module.exports = {
  setupFilesAfterEnv: ["./test-setup.ts"],
}

// test-setup.ts
import { DataSource } from "typeorm"

beforeAll(async () => {
  // Connect to test database
  const testDb = new DataSource({
    type: "postgres",
    database: "medusa_test",
    // ... other config
  })
  await testDb.initialize()
})

afterAll(async () => {
  // Clean up
  await testDb.destroy()
})

beforeEach(async () => {
  // Clear tables before each test
  await testDb.query("TRUNCATE TABLE product CASCADE")
})
```

---

### Q: Tests are slow. How can I speed them up?

**A:** Several optimization strategies:

**1. Run tests in parallel:**
```bash
yarn test --maxWorkers=4
```

**2. Run only changed tests:**
```bash
yarn test --onlyChanged
```

**3. Use test.only during development:**
```typescript
test.only("should create product", () => {
  // Only this test runs
})
```

**4. Mock expensive operations:**
```typescript
// Mock external API calls
jest.mock("axios")

// Mock database queries
jest.mock("../repository")
```

**5. Use in-memory database:**
```typescript
// Use SQLite in memory for tests
const testDb = new DataSource({
  type: "sqlite",
  database: ":memory:",
})
```

---

## Deployment

### Q: How do I deploy Medusa to production?

**A:** Key steps:

**1. Environment variables:**
```bash
# .env.production
DATABASE_URL=postgresql://user:pass@host:5432/medusa
JWT_SECRET=your-secret-key
COOKIE_SECRET=your-cookie-secret
REDIS_URL=redis://host:6379
NODE_ENV=production
```

**2. Build for production:**
```bash
yarn build
```

**3. Run migrations:**
```bash
yarn medusa db:migrate
```

**4. Start server:**
```bash
yarn start
```

**Platforms:**
- **Railway:** Easy deployment, good for small projects
- **Heroku:** Traditional PaaS
- **AWS/GCP:** Full control
- **Docker:** Containerized deployment

See deployment guides for specific platforms.

---

### Q: What environment variables are required?

**A:** Essential variables:

```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# Secrets (generate with openssl rand -hex 32)
JWT_SECRET=your-jwt-secret
COOKIE_SECRET=your-cookie-secret

# Redis (optional but recommended)
REDIS_URL=redis://host:6379

# Admin CORS
ADMIN_CORS=https://your-admin-domain.com

# Store CORS
STORE_CORS=https://your-store-domain.com

# Environment
NODE_ENV=production
```

---

### Q: How do I run migrations in production?

**A:** Carefully! Follow this process:

**1. Backup database:**
```bash
pg_dump medusa > backup-$(date +%Y%m%d).sql
```

**2. Test migration locally:**
```bash
# On local copy of production data
yarn medusa db:migrate
```

**3. Run in production:**
```bash
# During maintenance window
yarn medusa db:migrate

# If something fails
yarn medusa db:rollback
```

**4. Verify:**
```bash
psql -d medusa -c "\d product"  # Check schema
```

**Best practices:**
- Run during low-traffic period
- Have rollback plan ready
- Test on staging first
- Monitor application after migration

---

### Q: How should I scale Medusa?

**A:** Scaling strategies:

**Vertical scaling (easier):**
- Increase server resources (CPU, RAM)
- Good for most use cases
- Simple to implement

**Horizontal scaling (complex):**
- Multiple server instances
- Load balancer in front
- Shared Redis for sessions
- Shared PostgreSQL database

**Example architecture:**
```
Load Balancer
  ├── Medusa Instance 1
  ├── Medusa Instance 2
  └── Medusa Instance 3
       ↓
  PostgreSQL (primary + replicas)
  Redis (cluster)
```

**Database optimization:**
- Read replicas for queries
- Connection pooling
- Indexes on common queries

**Caching:**
- Redis for sessions
- CDN for static assets
- API response caching

---

## Contributing

### Q: Where do I start contributing?

**A:** Follow these steps:

1. **Read the contributing guide:**
   `/home/user/medusa/CONTRIBUTING.md`

2. **Find a good first issue:**
   Look for `good-first-issue` label on GitHub

3. **Join Discord:**
   Ask questions in #contributions channel

4. **Start small:**
   Fix a typo, add an example, write a test

See `/home/user/medusa/docs/learning/FIRST_CONTRIBUTIONS.md` for detailed guide.

---

### Q: How do I submit a pull request?

**A:** Follow this process:

```bash
# 1. Fork and clone
git clone https://github.com/YOUR_USERNAME/medusa.git

# 2. Create branch
git checkout -b fix/my-fix

# 3. Make changes
# ... edit files ...

# 4. Test
yarn test
yarn lint

# 5. Commit
git commit -m "fix: description of fix"

# 6. Push
git push origin fix/my-fix

# 7. Open PR on GitHub
```

**PR checklist:**
- [ ] Tests pass
- [ ] Linting passes
- [ ] Changes documented
- [ ] PR description complete
- [ ] Linked to issue

---

### Q: What's the code review process?

**A:** Typical flow:

1. **Submit PR** - You open a pull request
2. **CI checks** - Automated tests run (10-15 min)
3. **Review** - Maintainer reviews code (1-3 days)
4. **Feedback** - Requested changes if needed
5. **Update** - You address feedback
6. **Approval** - Maintainer approves
7. **Merge** - PR merged to main

**Tips:**
- Be patient - maintainers are busy
- Respond to feedback promptly
- Ask questions if unclear
- Keep PR scope focused

---

### Q: What's the style guide?

**A:** Key conventions:

**TypeScript:**
- Use TypeScript, not JavaScript
- Strict mode enabled
- Proper types, avoid `any`

**Formatting:**
- 2 spaces for indentation
- Semicolons required
- Double quotes for strings
- Prettier for formatting

**Naming:**
- camelCase for variables/functions
- PascalCase for classes/types
- UPPER_CASE for constants

**Code style:**
```typescript
// Good
export async function createProduct(
  data: CreateProductInput
): Promise<Product> {
  const product = await productService.createProduct(data)
  return product
}

// Avoid
export async function CreateProduct(data: any) {
  let product = await productService.createProduct(data)
  return product
}
```

---

## Troubleshooting

### Q: I'm getting TypeScript errors. How do I fix them?

**A:** Common solutions:

**1. Rebuild:**
```bash
yarn build
```

**2. Check TypeScript version:**
```bash
yarn why typescript
# Should be ^5.6.2
```

**3. Clear cache:**
```bash
rm -rf .medusa
rm -rf dist
yarn build
```

**4. Check imports:**
```typescript
// Use relative imports for local files
import { ProductService } from "./services/product"

// Use package imports for dependencies
import { MedusaService } from "@medusajs/framework/utils"
```

---

### Q: Server won't start. What do I check?

**A:** Troubleshooting checklist:

**1. Check port:**
```bash
lsof -i :9000  # Is port in use?
```

**2. Check database:**
```bash
psql -d medusa -c "SELECT 1"  # Can connect?
```

**3. Check logs:**
```bash
yarn dev  # Look for error messages
```

**4. Check environment:**
```bash
cat .env  # Are variables set?
```

**5. Rebuild:**
```bash
yarn build
```

**6. Check Node version:**
```bash
node --version  # Should be 20+
```

---

### Q: Getting "ECONNREFUSED" errors. What's wrong?

**A:** Connection refused means the service isn't running:

**For database:**
```bash
# Check if PostgreSQL is running
pg_isready

# Start PostgreSQL
# macOS:
brew services start postgresql

# Linux:
sudo systemctl start postgresql
```

**For Redis:**
```bash
# Check if Redis is running
redis-cli ping

# Start Redis
# macOS:
brew services start redis

# Linux:
sudo systemctl start redis
```

**For Medusa:**
```bash
# Make sure server is running
yarn dev
```

---

### Q: I found a bug. How do I report it?

**A:** Create a good bug report:

**1. Search existing issues first**

**2. Create new issue with:**
- Clear title
- Steps to reproduce
- Expected behavior
- Actual behavior
- Medusa version
- Environment (OS, Node version)
- Screenshots if applicable

**Example:**
```markdown
## Bug Report

**Describe the bug:**
Product creation fails when SKU contains spaces

**To Reproduce:**
1. Go to Products > New Product
2. Add variant with SKU "PROD 001"
3. Click Save
4. See error

**Expected:**
Product should be created with SKU "PROD 001"

**Actual:**
Error: "Invalid SKU format"

**Environment:**
- Medusa: 2.11.3
- Node: 20.10.0
- OS: macOS 14.0
```

**Include:**
- Error messages
- Stack traces
- Console logs

---

### Q: Where can I get help?

**A:** Multiple resources available:

**Discord (fastest):**
- https://discord.gg/medusajs
- Channels: #help, #development, #contributions

**GitHub:**
- Issues: Report bugs
- Discussions: Ask questions, share ideas

**Documentation:**
- `/home/user/medusa/docs/learning/`
- https://docs.medusajs.com

**Stack Overflow:**
- Tag: `medusajs`

**Twitter/X:**
- @medusajs

**Email:**
- hello@medusajs.com (general inquiries)

---

## Additional Resources

### Documentation
- Getting Started: `/home/user/medusa/docs/learning/GETTING_STARTED.md`
- Architecture: `/home/user/medusa/docs/learning/ARCHITECTURE_OVERVIEW.md`
- How-To Guide: `/home/user/medusa/docs/learning/HOW_TO_GUIDE.md`
- Exercises: `/home/user/medusa/docs/learning/EXERCISES.md`

### Community
- Discord: https://discord.gg/medusajs
- GitHub: https://github.com/medusajs/medusa
- Twitter: https://twitter.com/medusajs

### Tools
- Admin Dashboard: http://localhost:9000/app
- API Docs: https://docs.medusajs.com/api
- Postman Collection: Available in Discord

---

**Still have questions?** Ask in Discord or open a GitHub Discussion!

---

**Last Updated:** November 19, 2025

This FAQ is continuously updated based on common community questions. If you have a question not covered here, please ask in Discord and we'll add it!
