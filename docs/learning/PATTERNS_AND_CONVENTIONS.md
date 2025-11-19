# Patterns and Conventions Guide

**A comprehensive guide to Medusa's code patterns, naming conventions, and best practices**

**Last Updated:** November 19, 2025
**Version:** 2.11.3

---

## Table of Contents

1. [Naming Conventions](#naming-conventions)
2. [Directory Structure Conventions](#directory-structure-conventions)
3. [Dependency Injection Patterns](#dependency-injection-patterns)
4. [Workflow Patterns](#workflow-patterns)
5. [Module Patterns](#module-patterns)
6. [API Design Patterns](#api-design-patterns)
7. [Database Patterns](#database-patterns)
8. [Error Handling Patterns](#error-handling-patterns)
9. [Testing Patterns](#testing-patterns)
10. [TypeScript Patterns](#typescript-patterns)

---

## Naming Conventions

### Files and Directories

#### Module Files
```
✅ DO THIS:
kebab-case for directories and files
product-module.ts
product-service.ts
create-products.ts

❌ NOT THAT:
ProductModule.ts (PascalCase - only for classes)
product_module.ts (snake_case - avoid)
productModule.ts (camelCase - only for variables)
```

#### Test Files
```
✅ DO THIS:
product.spec.ts                    # Integration tests
product-module-service.spec.ts     # Unit tests
__tests__/product.spec.ts          # Test directory pattern

❌ NOT THAT:
product.test.ts (use .spec.ts)
product-test.ts (inconsistent)
```

**Mnemonic:** "K-Files" = **K**ebab-case for **Files**

### Variables and Functions

```typescript
// ✅ DO THIS:
// camelCase for variables and functions
const productService = container.resolve("productService")
const createdProducts = await service.createProducts(data)

function transformProductData(input: ProductInput) {
  // ...
}

// ❌ NOT THAT:
const ProductService = container.resolve("productService") // Too much like a class
const created_products = await service.createProducts(data) // snake_case
```

**Mnemonic:** "CAVE" = **C**amelCase for **A**ll **V**ariables & **E**xecutables

### Classes and Interfaces

```typescript
// ✅ DO THIS:
// PascalCase for classes, interfaces, types
class ProductModuleService {
  // ...
}

interface CreateProductDTO {
  title: string
  // ...
}

type WorkflowInput = {
  products: Product[]
}

// ❌ NOT THAT:
class productModuleService {} // camelCase
interface createProductDTO {} // camelCase
```

**Mnemonic:** "PIT" = **P**ascalCase for **I**nterfaces & **T**ypes

### Constants

```typescript
// ✅ DO THIS:
// UPPER_SNAKE_CASE for true constants
const MAX_RETRY_ATTEMPTS = 3
const DEFAULT_CURRENCY_CODE = "usd"

// camelCase for config objects
const productConfig = {
  enableInventory: true,
  defaultStatus: "draft"
}

// ❌ NOT THAT:
const maxRetryAttempts = 3 // Use UPPER_SNAKE_CASE for constants
const PRODUCT_CONFIG = {} // Config objects are camelCase
```

### Step and Workflow IDs

```typescript
// ✅ DO THIS:
// kebab-case with descriptive names
export const createProductsStepId = "create-products"
export const createProductsWorkflowId = "create-products"

// For nested workflows
export const updateProductVariantsStepId = "update-product-variants"

// ❌ NOT THAT:
export const CREATE_PRODUCTS = "createProducts" // Inconsistent
export const step1 = "step1" // Not descriptive
```

**Mnemonic:** "SWID" = **S**tep/**W**orkflow **ID**s = kebab-case

### Model ID Prefixes

```typescript
// ✅ DO THIS:
// Use consistent prefixes for entity IDs
const Product = model.define("Product", {
  id: model.id({ prefix: "prod" }).primaryKey(), // prod_01ABCD...
  // ...
})

const ProductVariant = model.define("ProductVariant", {
  id: model.id({ prefix: "variant" }).primaryKey(), // variant_01ABCD...
  // ...
})

// Common prefixes:
// prod_ = Product
// variant_ = ProductVariant
// order_ = Order
// cart_ = Cart
// cust_ = Customer
// usr_ = User
```

**Mnemonic:** "PREFIX" = **P**refixes **R**ead **E**asily, **F**ind **I**Ds e**X**actly

---

## Directory Structure Conventions

### Module Structure

Every module follows this standard structure:

```
packages/modules/product/
├── src/
│   ├── index.ts                 # Module export (Module() wrapper)
│   ├── joiner-config.ts         # Link configuration
│   ├── models/                  # Database entities
│   │   ├── product.ts
│   │   ├── product-variant.ts
│   │   └── index.ts
│   ├── services/                # Business logic
│   │   ├── product-module.ts    # Main module service
│   │   ├── index.ts
│   │   └── __tests__/
│   ├── repositories/            # Data access (if custom needed)
│   ├── migrations/              # Database migrations
│   ├── schema/                  # Validation schemas
│   └── types/                   # TypeScript types
├── integration-tests/           # Integration tests
│   ├── __tests__/
│   └── __fixtures__/
└── package.json
```

**Real Example:**
```
/home/user/medusa/packages/modules/product/src/
├── index.ts                      # Exports Module(Modules.PRODUCT, {...})
├── joiner-config.ts
├── models/
│   ├── product.ts
│   ├── product-variant.ts
│   ├── product-option.ts
│   ├── product-image.ts
│   └── ...
├── services/
│   ├── product-module-service.ts # The main service
│   └── product-category.ts       # Helper service
└── migrations/
    ├── InitialSetup20240401153642.ts
    └── Migration20240601111544.ts
```

### Workflow Structure

```
packages/core/core-flows/src/product/
├── workflows/                   # Workflow definitions
│   ├── create-products.ts
│   ├── update-products.ts
│   └── delete-products.ts
└── steps/                       # Step implementations
    ├── create-products.ts
    ├── update-products.ts
    └── validate-products.ts
```

**Pattern:**
- `workflows/` = Orchestration (what to do)
- `steps/` = Implementation (how to do it)

**Mnemonic:** "WS" = **W**orkflows orchestrate **S**teps

### API Route Structure

```
packages/medusa/src/api/
├── admin/                       # Admin API routes
│   ├── products/
│   │   ├── route.ts            # List (GET) & Create (POST)
│   │   ├── middlewares.ts      # Route-specific middleware
│   │   ├── validators.ts       # Zod schemas
│   │   ├── query-config.ts     # Query configuration
│   │   └── [id]/               # Dynamic route
│   │       ├── route.ts        # Get/Update/Delete single
│   │       └── variants/
│   │           └── route.ts
│   └── ...
├── store/                       # Storefront API routes
│   └── products/
│       └── route.ts
└── middlewares.ts               # Global middlewares
```

**Mnemonic:** "RMVQ" = **R**oute + **M**iddleware + **V**alidator + **Q**uery config

---

## Dependency Injection Patterns

Medusa uses **Awilix** for dependency injection.

### Service Resolution Pattern

```typescript
// ✅ DO THIS:
// In steps - resolve from container
import { Modules } from "@medusajs/framework/utils"

export const createProductsStep = createStep(
  "create-products",
  async (data: ProductTypes.CreateProductDTO[], { container }) => {
    const service = container.resolve<IProductModuleService>(Modules.PRODUCT)

    const created = await service.createProducts(data)
    return new StepResponse(created, created.map(p => p.id))
  },
  async (createdIds, { container }) => {
    // Compensation
    const service = container.resolve<IProductModuleService>(Modules.PRODUCT)
    await service.deleteProducts(createdIds)
  }
)

// ❌ NOT THAT:
import ProductService from "../services/product" // Direct import
const service = new ProductService() // Manual instantiation
```

**Key Points:**
1. Always use `container.resolve()`
2. Use module constants from `Modules` enum
3. Type the resolved service with interface (e.g., `IProductModuleService`)

### Module Constants

```typescript
// ✅ DO THIS:
import { Modules } from "@medusajs/framework/utils"

const productService = container.resolve(Modules.PRODUCT)    // "product"
const cartService = container.resolve(Modules.CART)          // "cart"
const orderService = container.resolve(Modules.ORDER)        // "order"

// ❌ NOT THAT:
const productService = container.resolve("product") // Magic string
const productService = container.resolve("productService") // Wrong name
```

**Available Modules:**
```typescript
Modules.PRODUCT          // "product"
Modules.CART            // "cart"
Modules.ORDER           // "order"
Modules.CUSTOMER        // "customer"
Modules.PAYMENT         // "payment"
Modules.FULFILLMENT     // "fulfillment"
Modules.INVENTORY       // "inventory"
Modules.PRICING         // "pricing"
Modules.PROMOTION       // "promotion"
// ... and 25+ more
```

**Mnemonic:** "CRM" = **C**ontainer **R**esolves **M**odules (not direct imports)

### Injected Dependencies Pattern

```typescript
// ✅ DO THIS:
// Define InjectedDependencies type
type InjectedDependencies = {
  logger: Logger
}

export class SendgridNotificationService
  extends AbstractNotificationProviderService {

  static identifier = "notification-sendgrid"
  protected logger_: Logger

  constructor(
    { logger }: InjectedDependencies,
    options: SendgridNotificationServiceOptions
  ) {
    super()
    this.logger_ = logger
    // ...
  }
}

// ❌ NOT THAT:
constructor(logger: Logger, options: Options) {
  // Missing type definition
  // Wrong parameter order
}
```

**Pattern:**
1. First parameter: `InjectedDependencies` (destructured)
2. Second parameter: `Options` (config)
3. Suffix injected properties with `_` (e.g., `logger_`, `config_`)

**Real Example:** `/home/user/medusa/packages/modules/providers/notification-sendgrid/src/services/sendgrid.ts`

---

## Workflow Patterns

### Basic Workflow Structure

```typescript
// ✅ DO THIS:
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"

export const createProductsWorkflowId = "create-products"

export const createProductsWorkflow = createWorkflow(
  createProductsWorkflowId,
  (input: WorkflowData<CreateProductsInput>) => {
    // 1. Validate input
    validateProductInputStep({ products: input.products })

    // 2. Create entities
    const createdProducts = createProductsStep(input.products)

    // 3. Handle relations
    associateProductsWithSalesChannelsStep({
      links: input.salesChannels
    })

    // 4. Return response
    return new WorkflowResponse(createdProducts)
  }
)
```

**Real Example:** `/home/user/medusa/packages/core/core-flows/src/product/workflows/create-products.ts`

### Step Creation Pattern

```typescript
// ✅ DO THIS:
export const createProductsStepId = "create-products"

export const createProductsStep = createStep(
  createProductsStepId,
  async (data: ProductTypes.CreateProductDTO[], { container }) => {
    // 1. Resolve service
    const service = container.resolve<IProductModuleService>(Modules.PRODUCT)

    // 2. Perform action
    const created = await service.createProducts(data)

    // 3. Return with compensation data
    return new StepResponse(
      created,                          // Output data
      created.map(product => product.id) // Compensation data
    )
  },
  async (createdIds, { container }) => {
    // 4. Compensation (rollback)
    if (!createdIds?.length) return

    const service = container.resolve<IProductModuleService>(Modules.PRODUCT)
    await service.deleteProducts(createdIds)
  }
)
```

**Real Example:** `/home/user/medusa/packages/core/core-flows/src/product/steps/create-products.ts`

### Transform Pattern

Use `transform` to manipulate data between steps:

```typescript
import { transform } from "@medusajs/framework/workflows-sdk"

// ✅ DO THIS:
const variantsInput = transform(
  { input, createdProducts },
  (data) => {
    const productVariants = []

    data.createdProducts.forEach((product, i) => {
      const inputProduct = data.input.products[i]

      for (const inputVariant of inputProduct.variants || []) {
        productVariants.push({
          product_id: product.id,
          ...inputVariant,
        })
      }
    })

    return { product_variants: productVariants }
  }
)

// Now use transformed data
createProductVariantsStep(variantsInput)

// ❌ NOT THAT:
// Don't try to manipulate step outputs directly
createdProducts.map(...) // Won't work - it's a proxy!
```

**Key Point:** Step outputs are proxies. Use `transform()` to access their data.

### Workflow Hooks Pattern

```typescript
import { createHook } from "@medusajs/framework/workflows-sdk"

// ✅ DO THIS:
export const createProductsWorkflow = createWorkflow(
  "create-products",
  (input: WorkflowData<CreateProductsInput>) => {
    const createdProducts = createProductsStep(input.products)

    // Create hook for extensibility
    const productsCreated = createHook("productsCreated", {
      products: createdProducts,
      additional_data: input.additional_data,
    })

    return new WorkflowResponse(createdProducts, {
      hooks: [productsCreated]
    })
  }
)
```

**Use Case:** Allow plugins/customizations to hook into workflows.

**Mnemonic:** "VTCH" = **V**alidate → **T**ransform → **C**reate → **H**ook

---

## Module Patterns

### Module Definition Pattern

```typescript
// ✅ DO THIS:
// packages/modules/product/src/index.ts
import { Module, Modules } from "@medusajs/framework/utils"
import { ProductModuleService } from "@services"

export default Module(Modules.PRODUCT, {
  service: ProductModuleService,
})
```

**Pattern:**
1. Import `Module` wrapper
2. Pass module constant and service
3. Export as default

**Real Example:** `/home/user/medusa/packages/modules/product/src/index.ts`

### Module Service Pattern

```typescript
// ✅ DO THIS:
export class ProductModuleService extends ModulesSdkUtils.MedusaService({
  Product,
  ProductVariant,
  ProductOption,
  // ... all entities
}) {
  // Inherits CRUD operations for all entities

  // Custom methods
  async customMethod(id: string) {
    // ...
  }
}
```

**What You Get:**
- `createProducts()` / `createProduct()`
- `updateProducts()` / `updateProduct()`
- `deleteProducts()` / `deleteProduct()`
- `listProducts()` / `retrieveProduct()`
- `listAndCountProducts()`
- Same for all entities!

**Mnemonic:** "CRUD-L" = **C**reate, **R**etrieve, **U**pdate, **D**elete, **L**ist

### Joiner Config Pattern

Defines how modules link together:

```typescript
// ✅ DO THIS:
// packages/modules/product/src/joiner-config.ts
import { defineJoinerConfig } from "@medusajs/framework/utils"
import Product from "./models/product"

export default defineJoinerConfig("product", {
  models: [Product],
  linkableKeys: {
    product_id: Product.name,
    variant_id: "ProductVariant",
  },
})
```

**Use Case:** Enables cross-module queries (e.g., fetch product with order details).

---

## API Design Patterns

### Route Handler Pattern

```typescript
// ✅ DO THIS:
// packages/medusa/src/api/admin/products/route.ts
import { createProductsWorkflow } from "@medusajs/core-flows"
import {
  AuthenticatedMedusaRequest,
  MedusaResponse,
  refetchEntity,
} from "@medusajs/framework/http"

export const POST = async (
  req: AuthenticatedMedusaRequest<HttpTypes.AdminCreateProduct>,
  res: MedusaResponse<HttpTypes.AdminProductResponse>
) => {
  // 1. Extract and validate body
  const { additional_data, ...products } = req.validatedBody

  // 2. Run workflow
  const { result } = await createProductsWorkflow(req.scope).run({
    input: { products: [products], additional_data },
  })

  // 3. Refetch with relations
  const product = await refetchEntity({
    entity: "product",
    idOrFilter: result[0].id,
    scope: req.scope,
    fields: req.queryConfig.fields ?? [],
  })

  // 4. Return response
  res.status(200).json({ product })
}
```

**Real Example:** `/home/user/medusa/packages/medusa/src/api/admin/products/route.ts`

### HTTP Method Naming Convention

```
✅ DO THIS:
export const GET = async (req, res) => {}     // List/Retrieve
export const POST = async (req, res) => {}    // Create
export const PUT = async (req, res) => {}     // Replace (rarely used)
export const PATCH = async (req, res) => {}   // Update
export const DELETE = async (req, res) => {}  // Delete

❌ NOT THAT:
export const listProducts = async () => {}    // Use GET
export const createProduct = async () => {}   // Use POST
```

### Validator Pattern

```typescript
// ✅ DO THIS:
// packages/medusa/src/api/admin/products/validators.ts
import { z } from "zod"
import { createSelectParams } from "../../utils/validators"

export const AdminCreateProduct = z.object({
  title: z.string(),
  subtitle: z.string().optional(),
  description: z.string().optional(),
  is_giftcard: z.boolean().default(false),
  discountable: z.boolean().default(true),
  images: z.array(z.object({
    url: z.string(),
  })).optional(),
  // ...
})

export type AdminCreateProductType = z.infer<typeof AdminCreateProduct>
```

**Mnemonic:** "ZITS" = **Z**od **I**nfers **T**ype**S**

### Middleware Pattern

```typescript
// ✅ DO THIS:
// packages/medusa/src/api/admin/products/middlewares.ts
import { MiddlewareRoute } from "@medusajs/framework/http"
import { validateAndTransformBody } from "../../utils/validate-body"
import { AdminCreateProduct, AdminUpdateProduct } from "./validators"

export const adminProductRoutesMiddlewares: MiddlewareRoute[] = [
  {
    method: ["POST"],
    matcher: "/admin/products",
    middlewares: [
      validateAndTransformBody(AdminCreateProduct),
    ],
  },
  {
    method: ["POST"],
    matcher: "/admin/products/:id",
    middlewares: [
      validateAndTransformBody(AdminUpdateProduct),
    ],
  },
]
```

**Pattern:** Middleware = Validation + Authentication + Transformation

---

## Database Patterns

### Model Definition Pattern

```typescript
// ✅ DO THIS:
import { model } from "@medusajs/framework/utils"

const Product = model
  .define("Product", {
    // Primary key with prefix
    id: model.id({ prefix: "prod" }).primaryKey(),

    // Text fields
    title: model.text().searchable(),
    handle: model.text(),
    subtitle: model.text().searchable().nullable(),
    description: model.text().searchable().nullable(),

    // Boolean with default
    is_giftcard: model.boolean().default(false),
    discountable: model.boolean().default(true),

    // Enum
    status: model
      .enum(ProductUtils.ProductStatus)
      .default(ProductUtils.ProductStatus.DRAFT),

    // Nullable fields
    thumbnail: model.text().nullable(),
    external_id: model.text().nullable(),

    // JSON
    metadata: model.json().nullable(),

    // Relationships
    variants: model.hasMany(() => ProductVariant, {
      mappedBy: "product",
    }),

    type: model
      .belongsTo(() => ProductType, {
        mappedBy: "products",
      })
      .nullable(),

    tags: model.manyToMany(() => ProductTag, {
      mappedBy: "products",
      pivotTable: "product_tags",
    }),
  })
  .cascades({
    delete: ["variants", "options", "images"],
  })
  .indexes([
    {
      name: "IDX_product_handle_unique",
      on: ["handle"],
      unique: true,
      where: "deleted_at IS NULL",
    },
  ])

export default Product
```

**Real Example:** `/home/user/medusa/packages/modules/product/src/models/product.ts`

### Relationship Patterns

```typescript
// ✅ One-to-Many (hasMany)
variants: model.hasMany(() => ProductVariant, {
  mappedBy: "product",  // Property name on the other side
})

// ✅ Many-to-One (belongsTo)
product: model.belongsTo(() => Product, {
  mappedBy: "variants",
})

// ✅ Many-to-Many
tags: model.manyToMany(() => ProductTag, {
  mappedBy: "products",
  pivotTable: "product_tags",  // Join table name
})
```

**Mnemonic:** "HBM" = **H**asMany, **B**elongsTo, **M**anyToMany

### Migration Pattern

```typescript
// ✅ DO THIS:
import { Migration } from "@medusajs/framework/mikro-orm/migrations"

export class Migration20240601111544 extends Migration {
  async up(): Promise<void> {
    this.addSql(
      'alter table if exists "product_variant" ' +
      'alter column "variant_rank" type integer ' +
      'using ("variant_rank"::integer);'
    )
  }

  async down(): Promise<void> {
    this.addSql(
      'alter table if exists "product_variant" ' +
      'alter column "variant_rank" type numeric ' +
      'using ("variant_rank"::numeric);'
    )
  }
}
```

**Real Example:** `/home/user/medusa/packages/modules/product/src/migrations/Migration20240601111544.ts`

**Pattern:**
- Class name: `Migration{YYYYMMDDHHMMSS}`
- Extend `Migration`
- Implement `up()` and `down()`
- Use `this.addSql()` for raw SQL

**Mnemonic:** "UD-SQL" = **U**p/**D**own with **SQL**

---

## Error Handling Patterns

### MedusaError Pattern

```typescript
import { MedusaError } from "@medusajs/framework/utils"

// ✅ DO THIS:
if (!notification) {
  throw new MedusaError(
    MedusaError.Types.INVALID_DATA,
    `No notification information provided`
  )
}

if (!productOptions.length) {
  throw new MedusaError(
    MedusaError.Types.INVALID_DATA,
    `Product options are not provided for: [${productTitles.join(", ")}].`
  )
}

// ❌ NOT THAT:
throw new Error("Invalid data") // Generic error
if (!notification) return null   // Silent failure
```

**Error Types:**
```typescript
MedusaError.Types.INVALID_DATA        // 400
MedusaError.Types.NOT_FOUND           // 404
MedusaError.Types.UNAUTHORIZED        // 401
MedusaError.Types.NOT_ALLOWED         // 403
MedusaError.Types.DUPLICATE_ERROR     // 409
MedusaError.Types.DB_ERROR            // 500
```

### Validation Pattern

```typescript
// ✅ DO THIS:
const missingOptionsProductTitles = products
  .filter((product) => !product.options?.length)
  .map((product) => product.title)

if (missingOptionsProductTitles.length) {
  throw new MedusaError(
    MedusaError.Types.INVALID_DATA,
    `Product options are not provided for: [${missingOptionsProductTitles.join(", ")}].`
  )
}
```

**Pattern:** Collect all errors, report together (better UX).

---

## Testing Patterns

### Integration Test Pattern

```typescript
// ✅ DO THIS:
import { moduleIntegrationTestRunner } from "@medusajs/test-utils"
import { Modules } from "@medusajs/framework/utils"

jest.setTimeout(300000)

moduleIntegrationTestRunner<IProductModuleService>({
  moduleName: Modules.PRODUCT,
  testSuite: ({ service }) => {
    describe("Product Module", () => {
      it("should create a product", async () => {
        const product = await service.createProducts({
          title: "Test Product",
          options: [{ title: "Size", values: ["S", "M"] }],
        })

        expect(product.title).toBe("Test Product")
        expect(product.options).toHaveLength(1)
      })
    })
  },
})
```

**Real Example:** `/home/user/medusa/packages/modules/product/integration-tests/__tests__/product.spec.ts`

### Test File Organization

```
integration-tests/
├── __tests__/
│   ├── product.spec.ts
│   └── product-category.spec.ts
└── __fixtures__/
    └── product/
        ├── data.ts              # Test data
        └── index.ts             # Helper functions
```

**Mnemonic:** "TDF" = **T**ests, **D**ata, **F**ixtures

---

## TypeScript Patterns

### Type Definition Pattern

```typescript
// ✅ DO THIS:
// Use interfaces for object shapes
interface CreateProductDTO {
  title: string
  description?: string
  options?: CreateProductOptionDTO[]
}

// Use types for unions, intersections
type ProductStatus = "draft" | "published" | "archived"

type CreateProductInput = CreateProductDTO & {
  sales_channels?: { id: string }[]
}

// ❌ NOT THAT:
type CreateProductDTO = {  // Use interface
  title: string
}

interface ProductStatus {  // Use type for unions
  // ...
}
```

### Generic Type Pattern

```typescript
// ✅ DO THIS:
type WorkflowData<T> = T | WorkflowDataProxy<T>

type StepResponse<TOutput, TCompensateInput> = {
  output: TOutput
  compensateInput?: TCompensateInput
}

// Use generics for reusable patterns
function createStep<TInput, TOutput, TCompensateInput>(
  id: string,
  invokeFn: InvokeFn<TInput, TOutput, TCompensateInput>,
  compensateFn?: CompensateFn<TCompensateInput>
) {
  // ...
}
```

### Utility Types Pattern

```typescript
// ✅ DO THIS:
// Pick specific fields
type ProductSummary = Pick<Product, "id" | "title" | "handle">

// Omit fields
type ProductWithoutRelations = Omit<Product, "variants" | "options">

// Make all optional
type PartialProduct = Partial<Product>

// Make all required
type RequiredProduct = Required<Product>

// Combine
type UpdateProductDTO = Partial<Omit<CreateProductDTO, "id">>
```

**Mnemonic:** "POPR" = **P**ick, **O**mit, **P**artial, **R**equired

---

## Quick Reference Cheat Sheet

```
FILES:           kebab-case          product-module.ts
VARIABLES:       camelCase           productService
CLASSES:         PascalCase          ProductService
CONSTANTS:       UPPER_SNAKE_CASE    MAX_RETRIES
IDs:             kebab-case          "create-products"
PREFIXES:        underscore          prod_01ABC

DIRECTORIES:
  Module:        models/ services/ migrations/
  Workflow:      workflows/ steps/
  API:           route.ts middlewares.ts validators.ts

DI:              container.resolve(Modules.PRODUCT)
WORKFLOW:        createWorkflow → createStep → StepResponse
ERROR:           throw new MedusaError(Types.INVALID_DATA, message)
MODEL:           model.define().cascades().indexes()
TEST:            moduleIntegrationTestRunner
```

---

## Common Pitfalls

### ❌ Pitfall 1: Direct Imports
```typescript
// ❌ WRONG
import ProductService from "../services/product"
const service = new ProductService()

// ✅ CORRECT
const service = container.resolve(Modules.PRODUCT)
```

### ❌ Pitfall 2: Manipulating Step Outputs
```typescript
// ❌ WRONG
const products = createProductsStep(input)
const ids = products.map(p => p.id) // Won't work!

// ✅ CORRECT
const ids = transform({ products }, (data) =>
  data.products.map(p => p.id)
)
```

### ❌ Pitfall 3: Missing Compensation
```typescript
// ❌ WRONG
export const createProductsStep = createStep(
  "create-products",
  async (data, { container }) => {
    const service = container.resolve(Modules.PRODUCT)
    const created = await service.createProducts(data)
    return created // Missing StepResponse!
  }
  // Missing compensation function!
)

// ✅ CORRECT
export const createProductsStep = createStep(
  "create-products",
  async (data, { container }) => {
    const service = container.resolve(Modules.PRODUCT)
    const created = await service.createProducts(data)
    return new StepResponse(created, created.map(p => p.id))
  },
  async (createdIds, { container }) => {
    if (!createdIds?.length) return
    const service = container.resolve(Modules.PRODUCT)
    await service.deleteProducts(createdIds)
  }
)
```

### ❌ Pitfall 4: Inconsistent Naming
```typescript
// ❌ WRONG
const createProduct = "createProducts"      // camelCase ID
const CreateProductsStep = createStep(...)  // PascalCase variable

// ✅ CORRECT
const createProductsStepId = "create-products"  // kebab-case ID
const createProductsStep = createStep(...)      // camelCase variable
```

---

**Next Steps:**
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step task recipes
- [CODE_TOURS.md](./CODE_TOURS.md) - Guided code walkthroughs
- [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Daily development workflow
