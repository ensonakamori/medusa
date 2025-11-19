# Database Architecture - MikroORM, Entities & Migrations

> **Version**: ✅ CURRENT - Medusa 2.11.3 (Updated: Nov 2025)

A comprehensive deep-dive into Medusa's database layer, powered by MikroORM 6.4.16 with PostgreSQL, featuring a database-per-module architecture.

## Table of Contents

- [Tech Stack Overview](#tech-stack-overview)
- [Why MikroORM?](#why-mikroorm)
- [Entity Definition Patterns](#entity-definition-patterns)
- [Repository Pattern](#repository-pattern)
- [Relationships](#relationships)
- [Migration System](#migration-system)
- [Database Per Module](#database-per-module)
- [Transactions & Unit of Work](#transactions--unit-of-work)
- [Querying Patterns](#querying-patterns)
- [Mental Models](#mental-models)
- [Comparison: Prisma/TypeORM vs MikroORM](#comparison-prismatypeorm-vs-mikroorm)

---

## Tech Stack Overview

### Core Dependencies ✅ CURRENT

**Location**: `/home/user/medusa/packages/deps/package.json`

```json
{
  "dependencies": {
    "@mikro-orm/cli": "6.4.16",
    "@mikro-orm/core": "6.4.16",
    "@mikro-orm/knex": "6.4.16",
    "@mikro-orm/migrations": "6.4.16",
    "@mikro-orm/postgresql": "6.4.16",
    "pg": "8.16.3"
  }
}
```

**Framework Re-exports**: `/home/user/medusa/packages/core/framework/package.json`

```json
{
  "exports": {
    "./mikro-orm/cli": "./dist/deps/mikro-orm-cli.js",
    "./mikro-orm/core": "./dist/deps/mikro-orm-core.js",
    "./mikro-orm/knex": "./dist/deps/mikro-orm-knex.js",
    "./mikro-orm/migrations": "./dist/deps/mikro-orm-migrations.js",
    "./mikro-orm/postgresql": "./dist/deps/mikro-orm-postgresql.js",
    "./pg": "./dist/deps/pg.js"
  }
}
```

### Version Notes

- **MikroORM 6.4.16**: Latest v6 stable, identity map pattern, Unit of Work
- **PostgreSQL (pg 8.16.3)**: Primary database, supports JSONB, full-text search
- **Knex**: Query builder used internally by MikroORM for SQL generation

---

## Why MikroORM?

### MikroORM vs Alternatives

| Feature | MikroORM | Prisma | TypeORM |
|---------|----------|--------|---------|
| **Identity Map** | ✅ Yes | ❌ No | ⚠️ Partial |
| **Unit of Work** | ✅ Yes | ❌ No | ⚠️ Limited |
| **Type Safety** | ✅ Full | ✅ Full | ⚠️ Partial |
| **Schema First** | ✅ Code | ❌ Schema | ✅ Code |
| **Migrations** | ✅ Auto-generated | ✅ Auto-generated | ⚠️ Manual |
| **Multi-DB** | ✅ Yes | ⚠️ Limited | ✅ Yes |
| **Performance** | ✅ Excellent | ⚠️ Good | ⚠️ Variable |

### Key Advantages for Medusa

1. **Identity Map**: Ensures entity uniqueness per request (prevents duplicate instances)
2. **Unit of Work**: Automatic change tracking, optimized batch updates
3. **Database Per Module**: Each module can have its own database schema
4. **Advanced Relationships**: Supports complex many-to-many via link modules
5. **Migration Safety**: Auto-generates migrations from entity changes

🧠 **Mental Model**: MikroORM is like a **smart cache** between your code and database:
- It tracks which entities you've loaded (Identity Map)
- It remembers what you've changed (Unit of Work)
- It batches all changes into optimized SQL (flush)

**Mnemonic**: "**M**anages **I**dentity, **K**eeps **R**ecords, **O**rchestrates **O**perations, **R**elational **M**agic"

---

## Entity Definition Patterns

Medusa uses a **model builder pattern** for defining entities:

### Basic Entity Structure

**File**: `/home/user/medusa/packages/modules/cart/src/models/cart.ts:1`

```typescript
import { model } from "@medusajs/framework/utils"
import Address from "./address"
import LineItem from "./line-item"
import ShippingMethod from "./shipping-method"

const Cart = model
  .define("Cart", {
    // Primary key with auto-generated prefix
    id: model.id({ prefix: "cart" }).primaryKey(),

    // Scalar fields
    region_id: model.text().nullable(),
    customer_id: model.text().nullable(),
    sales_channel_id: model.text().nullable(),
    email: model.text().nullable(),
    currency_code: model.text(),
    metadata: model.json().nullable(),
    completed_at: model.dateTime().nullable(),

    // One-to-one relationships
    shipping_address: model
      .hasOne(() => Address, {
        mappedBy: undefined,
        foreignKey: true,
      })
      .nullable(),

    billing_address: model
      .hasOne(() => Address, {
        mappedBy: undefined,
        foreignKey: true,
      })
      .nullable(),

    // One-to-many relationships
    items: model.hasMany(() => LineItem, {
      mappedBy: "cart",
    }),

    shipping_methods: model.hasMany(() => ShippingMethod, {
      mappedBy: "cart",
    }),
  })
  .cascades({
    // Cascade delete related entities
    delete: [
      "items",
      "shipping_methods",
      "shipping_address",
      "billing_address",
    ],
  })
  .indexes([
    {
      name: "IDX_cart_region_id",
      on: ["region_id"],
      where: "deleted_at IS NULL AND region_id IS NOT NULL",
    },
    {
      name: "IDX_cart_customer_id",
      on: ["customer_id"],
      where: "deleted_at IS NULL AND customer_id IS NOT NULL",
    },
  ])

export default Cart
```

💡 **Aha Moment**: The `model` builder provides:
- **Type inference** - TypeScript types auto-generated
- **Validation** - Runtime checks for data integrity
- **Soft deletes** - Automatic `deleted_at` column (filtered in indexes)
- **Audit timestamps** - Auto `created_at` / `updated_at`

### Field Type Reference

```typescript
// Text fields
model.text()                    // VARCHAR
model.text().nullable()         // VARCHAR NULL

// Numbers
model.number()                  // INTEGER
model.bigNumber()               // BIGINT (for currency amounts)

// Booleans
model.boolean()                 // BOOLEAN
model.boolean().default(true)   // BOOLEAN DEFAULT true

// Dates
model.dateTime()                // TIMESTAMP
model.dateTime().nullable()     // TIMESTAMP NULL

// JSON
model.json()                    // JSONB (PostgreSQL)
model.json().nullable()         // JSONB NULL

// IDs with prefix
model.id({ prefix: "cart" })    // cart_01HQXYZ...
```

🧠 **Mental Model - Field Types**:
- `text()` → strings
- `number()` → integers
- `bigNumber()` → big integers (use for money!)
- `boolean()` → true/false
- `dateTime()` → timestamps
- `json()` → objects/arrays

**Mnemonic**: "**T**ext **N**umbers **B**ooleans **D**ates **J**SON" = TNBDJ field types

### Complex Entity Example

**File**: `/home/user/medusa/packages/modules/cart/src/models/line-item.ts:1`

```typescript
import { model } from "@medusajs/framework/utils"
import Cart from "./cart"
import LineItemAdjustment from "./line-item-adjustment"
import LineItemTaxLine from "./line-item-tax-line"

const LineItem = model
  .define(
    { name: "LineItem", tableName: "cart_line_item" },
    {
      id: model.id({ prefix: "cali" }).primaryKey(),
      title: model.text(),
      subtitle: model.text().nullable(),
      thumbnail: model.text().nullable(),
      quantity: model.number(),

      // Product references
      variant_id: model.text().nullable(),
      product_id: model.text().nullable(),
      product_title: model.text().nullable(),

      // Pricing
      unit_price: model.bigNumber(),
      compare_at_unit_price: model.bigNumber().nullable(),

      // Flags
      requires_shipping: model.boolean().default(true),
      is_discountable: model.boolean().default(true),
      is_giftcard: model.boolean().default(false),
      is_tax_inclusive: model.boolean().default(false),

      // JSON data
      variant_option_values: model.json().nullable(),
      metadata: model.json().nullable(),

      // Relationships
      adjustments: model.hasMany(() => LineItemAdjustment, {
        mappedBy: "item",
      }),
      tax_lines: model.hasMany(() => LineItemTaxLine, {
        mappedBy: "item",
      }),
      cart: model.belongsTo(() => Cart, {
        mappedBy: "items",
      }),
    }
  )
  .indexes([
    {
      name: "IDX_cart_line_item_cart_id",
      on: ["cart_id"],
      where: "deleted_at IS NULL",
    },
    {
      name: "IDX_line_item_variant_id",
      on: ["variant_id"],
      where: "deleted_at IS NULL AND variant_id IS NOT NULL",
    },
  ])
  .cascades({
    delete: ["adjustments", "tax_lines"],
  })

export default LineItem
```

💡 **Aha Moment**: Notice `bigNumber()` for pricing fields - never use `number()` for money! JavaScript's `Number` loses precision for large values.

---

## Repository Pattern

Medusa modules use **repositories** to access the database:

### Repository Structure

```typescript
import { DAL } from "@medusajs/framework/types"
import { SqlEntityManager } from "@mikro-orm/postgresql"

export class CartRepository extends DALUtils.MikroOrmBaseRepository {
  constructor({
    manager,
  }: {
    manager: SqlEntityManager
  }) {
    super(manager, [Cart, LineItem, Address, ShippingMethod])
  }

  async findByCustomerId(customerId: string): Promise<Cart[]> {
    return await this.find({
      customer_id: customerId,
      completed_at: null, // Only active carts
    })
  }

  async findWithItems(cartId: string): Promise<Cart | null> {
    return await this.findOne(
      { id: cartId },
      {
        populate: ["items", "shipping_address"],
      }
    )
  }
}
```

### Service Using Repository

```typescript
export class CartModuleService extends ModulesSdkUtils.MedusaService({
  Cart,
  LineItem,
  Address,
}) {
  constructor(
    @InjectManager("baseRepository") readonly baseRepository_: CartRepository
  ) {
    super(...arguments)
  }

  async retrieveWithItems(
    id: string,
    config?: FindConfig<Cart>
  ): Promise<Cart> {
    return await this.baseRepository_.findWithItems(id)
  }
}
```

---

## Relationships

### One-to-One (1:1)

```typescript
// Cart has one shipping address
shipping_address: model
  .hasOne(() => Address, {
    mappedBy: undefined,      // Not bidirectional
    foreignKey: true,         // Create FK column
  })
  .nullable()
```

Generated SQL:
```sql
ALTER TABLE cart
ADD COLUMN shipping_address_id TEXT,
ADD FOREIGN KEY (shipping_address_id) REFERENCES address(id);
```

### One-to-Many (1:N)

```typescript
// Cart has many line items
// In Cart model:
items: model.hasMany(() => LineItem, {
  mappedBy: "cart",
})

// In LineItem model:
cart: model.belongsTo(() => Cart, {
  mappedBy: "items",
})
```

Generated SQL:
```sql
ALTER TABLE cart_line_item
ADD COLUMN cart_id TEXT NOT NULL,
ADD FOREIGN KEY (cart_id) REFERENCES cart(id);
```

### Many-to-Many (M:N) via Link Modules

Medusa uses **link modules** for M:N relationships (covered in INTEGRATION_GUIDE.md):

```typescript
// Product ←→ Sales Channel (many-to-many)
// Via link module: product_sales_channel

// No direct relationship in Product or SalesChannel models
// Instead, queried via remote links
```

🧠 **Mental Model - Relationship Types**:

```
One-to-One:   Cart [1] ←→ [1] Address
One-to-Many:  Cart [1] ←→ [N] LineItem
Many-to-Many: Product [N] ←→ [N] SalesChannel (via link table)
```

**Mnemonic**: "**H**as **O**ne, **H**as **M**any, **B**elongs **T**o, **L**ink **M**odule"

---

## Migration System

Each module manages its own migrations:

### Migration Directory Structure

```
packages/modules/cart/src/migrations/
├── Migration20240831125857.ts
├── Migration20241216183049.ts
├── Migration20241106085918.ts
└── Migration20250120115059.ts
```

### Migration Example

```typescript
import { Migration } from "@mikro-orm/migrations"

export class Migration20250120115059 extends Migration {
  async up(): Promise<void> {
    // Add new column
    this.addSql(
      'ALTER TABLE "cart" ADD COLUMN "metadata" JSONB NULL;'
    )

    // Create index
    this.addSql(
      'CREATE INDEX "IDX_cart_customer_id" ON "cart" ("customer_id") WHERE deleted_at IS NULL;'
    )
  }

  async down(): Promise<void> {
    // Rollback changes
    this.addSql('DROP INDEX "IDX_cart_customer_id";')
    this.addSql('ALTER TABLE "cart" DROP COLUMN "metadata";')
  }
}
```

### Generating Migrations

```bash
# Generate migration from entity changes
npx medusa-mikro-orm migration:create

# Run pending migrations
npx medusa-mikro-orm migration:up

# Rollback last migration
npx medusa-mikro-orm migration:down
```

💡 **Aha Moment**: MikroORM **auto-generates** migrations by comparing:
1. Current entity definitions (code)
2. Current database schema
3. Produces SQL to sync them

---

## Database Per Module

Medusa uses a **database-per-module** architecture:

```
PostgreSQL Database
├── Schema: cart
│   ├── cart
│   ├── cart_line_item
│   ├── cart_address
│   └── cart_shipping_method
├── Schema: product
│   ├── product
│   ├── product_variant
│   ├── product_option
│   └── product_image
├── Schema: order
│   ├── order
│   ├── order_item
│   └── order_fulfillment
└── Schema: link
    ├── product_sales_channel
    ├── product_variant_inventory_item
    └── order_cart
```

### Benefits

1. **Isolation**: Modules don't directly access each other's tables
2. **Versioning**: Each module has independent migrations
3. **Scalability**: Can split schemas into separate databases later
4. **Testing**: Test modules independently

🧠 **Mental Model**: Think of each module as a **microservice with its own database**, but physically co-located for convenience.

---

## Transactions & Unit of Work

### Unit of Work Pattern

MikroORM's Unit of Work tracks all changes in memory and flushes them as a single transaction:

```typescript
import { MikroORM } from "@mikro-orm/core"

export class CartService {
  async updateCart(cartId: string, data: UpdateCartInput) {
    const em = this.manager_ // EntityManager

    // 1. Load entity (tracked by Identity Map)
    const cart = await em.findOne(Cart, { id: cartId })

    // 2. Make changes (tracked by Unit of Work)
    cart.email = data.email
    cart.customer_id = data.customer_id

    // 3. Flush changes (single transaction)
    await em.flush()

    // ✅ Generates optimized SQL:
    // UPDATE cart SET email = ?, customer_id = ? WHERE id = ?
  }
}
```

### Explicit Transactions

```typescript
import { MikroORM, EntityManager } from "@mikro-orm/core"

async function transferItems(fromCartId: string, toCartId: string) {
  const em = this.manager_

  await em.transactional(async (tx) => {
    // All operations use the same transaction
    const fromCart = await tx.findOne(Cart, { id: fromCartId })
    const toCart = await tx.findOne(Cart, { id: toCartId })

    const items = await tx.find(LineItem, { cart_id: fromCartId })

    for (const item of items) {
      item.cart = toCart
    }

    await tx.flush()

    // ✅ All changes committed atomically
    // ❌ If any fails, all rollback
  })
}
```

💡 **Aha Moment**: You rarely write manual `BEGIN`/`COMMIT` - MikroORM handles it:
- **Implicit**: `flush()` wraps in transaction
- **Explicit**: `transactional()` for multi-step operations

---

## Querying Patterns

### Basic Queries

```typescript
// Find all carts
const carts = await em.find(Cart, {})

// Find with filters
const activeCarts = await em.find(Cart, {
  completed_at: null,
  customer_id: "cus_123",
})

// Find one
const cart = await em.findOne(Cart, { id: "cart_123" })

// Find one or fail
const cart = await em.findOneOrFail(Cart, { id: "cart_123" })
```

### Eager Loading (Populate)

```typescript
// Load cart with relationships
const cart = await em.findOne(
  Cart,
  { id: "cart_123" },
  {
    populate: ["items", "shipping_address"],
  }
)

// Nested populate
const cart = await em.findOne(
  Cart,
  { id: "cart_123" },
  {
    populate: ["items.adjustments", "items.tax_lines"],
  }
)
```

### Query Builder (Knex-style)

```typescript
const qb = em.createQueryBuilder(Cart, "c")

const carts = await qb
  .select("*")
  .where({ customer_id: "cus_123" })
  .andWhere({ completed_at: null })
  .orderBy({ created_at: "DESC" })
  .limit(10)
  .getResult()
```

### Pagination

```typescript
const [carts, count] = await em.findAndCount(
  Cart,
  { customer_id: "cus_123" },
  {
    limit: 20,
    offset: 0,
    orderBy: { created_at: "DESC" },
  }
)

// Result:
// carts: Cart[] (20 items)
// count: number (total matching)
```

### Filtering with Operators

```typescript
import { FilterQuery } from "@mikro-orm/core"

const filters: FilterQuery<Cart> = {
  created_at: {
    $gte: new Date("2025-01-01"), // Greater than or equal
    $lte: new Date("2025-12-31"), // Less than or equal
  },
  customer_id: {
    $ne: null, // Not null
  },
}

const carts = await em.find(Cart, filters)
```

💡 **Aha Moment**: MikroORM uses MongoDB-style operators:
- `$eq`: Equals
- `$ne`: Not equals
- `$gt`/`$gte`: Greater than (or equal)
- `$lt`/`$lte`: Less than (or equal)
- `$in`: In array
- `$nin`: Not in array

---

## Mental Models

### 🧠 Identity Map Mental Model

```
┌──────────────────────────────────────────┐
│ EntityManager (Request Scope)            │
├──────────────────────────────────────────┤
│                                          │
│ Identity Map:                            │
│ ┌────────────────────────────────────┐  │
│ │ "cart_123" → Cart { ... }          │  │
│ │ "cali_456" → LineItem { ... }      │  │
│ │ "cali_789" → LineItem { ... }      │  │
│ └────────────────────────────────────┘  │
│                                          │
│ // Second load uses cached instance      │
│ const cart1 = await em.find(Cart, "cart_123") │
│ const cart2 = await em.find(Cart, "cart_123") │
│ console.log(cart1 === cart2) // ✅ true  │
└──────────────────────────────────────────┘
```

### 🧠 Unit of Work Mental Model

```
┌──────────────────────────────────────────┐
│ Unit of Work (Change Tracker)            │
├──────────────────────────────────────────┤
│                                          │
│ 1. cart.email = "new@email.com"          │
│    → Mark "cart" as dirty                │
│                                          │
│ 2. lineItem.quantity = 5                 │
│    → Mark "lineItem" as dirty            │
│                                          │
│ 3. await em.flush()                      │
│    → Generate UPDATE for cart            │
│    → Generate UPDATE for lineItem        │
│    → Execute in single transaction       │
│                                          │
│ ✅ Optimized: Only changed fields        │
│ ✅ Batched: Single transaction           │
└──────────────────────────────────────────┘
```

### 🌉 Bridge: Prisma/TypeORM → MikroORM

| Prisma | TypeORM | MikroORM | Concept |
|--------|---------|----------|---------|
| `prisma.product.findMany()` | `repository.find()` | `em.find(Product, {})` | Query all |
| `prisma.product.findUnique()` | `repository.findOne()` | `em.findOne(Product, {})` | Query one |
| `prisma.product.create()` | `repository.save()` | `em.persistAndFlush()` | Create |
| `prisma.product.update()` | `repository.save()` | `em.flush()` | Update |
| `include: { variants: true }` | `relations: ["variants"]` | `populate: ["variants"]` | Eager load |
| Transaction API | `QueryRunner` | `em.transactional()` | Transaction |

### 💡 Key Aha Moments

1. **Identity Map = One Instance Per Entity**: No duplicate objects in memory
2. **Unit of Work = Auto Change Tracking**: MikroORM knows what changed
3. **flush() = Smart Batch Update**: Generates minimal SQL
4. **Database Per Module = Isolation**: Modules don't share tables

---

## Advanced Patterns

### Soft Deletes

All Medusa entities support soft deletes via `deleted_at`:

```typescript
// Soft delete (sets deleted_at)
await em.softRemove(cart)

// Hard delete (actually removes row)
await em.nativeDelete(Cart, { id: "cart_123" })

// Query excludes soft-deleted by default
const carts = await em.find(Cart, {}) // WHERE deleted_at IS NULL

// Include soft-deleted
const allCarts = await em.find(Cart, {}, { filters: false })
```

### Lifecycle Hooks

```typescript
@Entity()
class Cart {
  @Property()
  id: string

  @BeforeCreate()
  beforeCreate() {
    console.log("About to create cart")
  }

  @AfterCreate()
  afterCreate() {
    console.log("Cart created")
  }

  @BeforeUpdate()
  beforeUpdate() {
    console.log("About to update cart")
  }
}
```

### Custom Repository Methods

```typescript
export class CartRepository extends MikroOrmBaseRepository {
  async findActiveByCustomer(customerId: string): Promise<Cart[]> {
    const qb = this.createQueryBuilder("c")

    return await qb
      .where({ customer_id: customerId })
      .andWhere({ completed_at: null })
      .andWhere({ deleted_at: null })
      .orderBy({ updated_at: "DESC" })
      .getResult()
  }

  async countItemsByCart(cartId: string): Promise<number> {
    const qb = this.manager_.createQueryBuilder(LineItem, "li")

    return await qb
      .where({ cart_id: cartId })
      .andWhere({ deleted_at: null })
      .getCount()
  }
}
```

---

## Performance Optimizations

### 1. Selective Field Loading

```typescript
// ❌ Bad: Load all fields
const products = await em.find(Product, {})

// ✅ Good: Load only needed fields
const products = await em.find(
  Product,
  {},
  {
    fields: ["id", "title", "handle"],
  }
)
```

### 2. Batch Loading

```typescript
// ❌ Bad: N+1 query problem
for (const cart of carts) {
  const items = await em.find(LineItem, { cart_id: cart.id })
}

// ✅ Good: Populate relationship
const carts = await em.find(
  Cart,
  {},
  { populate: ["items"] }
)
```

### 3. Query Result Caching

```typescript
const products = await em.find(
  Product,
  { status: "published" },
  {
    cache: 60000, // Cache for 60 seconds
  }
)
```

---

## File References

All file paths mentioned:

1. **Cart Model**: `/home/user/medusa/packages/modules/cart/src/models/cart.ts:1`
2. **LineItem Model**: `/home/user/medusa/packages/modules/cart/src/models/line-item.ts:1`
3. **Cart Module**: `/home/user/medusa/packages/modules/cart/src/index.ts:1`
4. **Framework Package**: `/home/user/medusa/packages/core/framework/package.json`
5. **Deps Package**: `/home/user/medusa/packages/deps/package.json`

---

## Summary

Medusa's database layer is built on **MikroORM 6.4.16** with PostgreSQL and features:

- **Identity Map** - Entity uniqueness per request
- **Unit of Work** - Automatic change tracking
- **Model Builder** - Type-safe entity definitions
- **Database Per Module** - Isolated schemas per domain
- **Auto Migrations** - Generated from entity changes
- **Smart Relationships** - 1:1, 1:N, M:N via link modules
- **Soft Deletes** - Audit trail via `deleted_at`
- **Transaction Support** - Automatic and explicit transactions

**Key Principle**: Entities are objects, not SQL rows - MikroORM handles the mapping magic.
