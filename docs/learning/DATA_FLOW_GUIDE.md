# Medusa Data Flow Guide

**Version:** 2.11.3
**Last Updated:** November 18, 2025
**For:** Understanding how data flows through the Medusa system

---

## Table of Contents

1. [Overview](#overview)
2. [Request Lifecycle](#request-lifecycle)
3. [API Request Flow](#api-request-flow)
4. [Workflow Execution Flow](#workflow-execution-flow)
5. [Event Bus Flow](#event-bus-flow)
6. [Module Communication](#module-communication)
7. [Real-World Examples](#real-world-examples)
8. [Database Transaction Flow](#database-transaction-flow)
9. [Troubleshooting Data Flow](#troubleshooting-data-flow)

---

## Overview

Understanding data flow in Medusa requires tracing requests through multiple layers:

```mermaid
graph TB
    Client[Client Request]
    Router[Express Router]
    MW[Middleware Chain]
    Route[Route Handler]
    WF[Workflow Engine]
    Modules[Commerce Modules]
    DB[(PostgreSQL)]
    Cache[(Redis Cache)]
    Events[Event Bus]

    Client -->|HTTP Request| Router
    Router -->|Match route| MW
    MW -->|Validate/Auth| Route
    Route -->|Execute| WF
    WF -->|Call| Modules
    Modules -->|Query| DB
    Modules -->|Cache| Cache
    WF -->|Emit| Events
    Events -->|Notify| Modules
    Route -->|Response| Client

    style Client fill:#61dafb
    style Router fill:#68a063
    style WF fill:#a8e6cf
    style Modules fill:#e1f5ff
    style DB fill:#4ecdc4
    style Events fill:#ffd93d
```

### Key Concepts

🧠 **Mental Model:** Data flows like a river through channels

1. **HTTP Layer** - The entry point (river source)
2. **Middleware** - Filters and validators (water purification)
3. **Route Handlers** - Traffic controllers (dam operators)
4. **Workflows** - Orchestrators (irrigation systems)
5. **Modules** - Workers (water wheels)
6. **Database** - Storage (reservoir)
7. **Event Bus** - Notifications (river tributaries)

---

## Request Lifecycle

### Complete Request Flow

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Express
    participant Middleware
    participant RouteHandler
    participant Workflow
    participant Module
    participant Database
    participant EventBus

    Client->>Express: POST /store/carts
    Express->>Middleware: Process request
    Middleware->>Middleware: Parse body
    Middleware->>Middleware: Authenticate
    Middleware->>Middleware: Validate schema
    Middleware->>RouteHandler: Pass request
    RouteHandler->>Workflow: Execute createCartWorkflow
    Workflow->>Module: Find region
    Module->>Database: SELECT region
    Database-->>Module: Region data
    Module-->>Workflow: Region object
    Workflow->>Module: Create cart
    Module->>Database: INSERT cart
    Database-->>Module: Cart ID
    Module-->>Workflow: Cart object
    Workflow->>EventBus: Emit cart.created
    EventBus-->>Workflow: Event emitted
    Workflow-->>RouteHandler: Workflow result
    RouteHandler->>Module: Refetch cart with relations
    Module->>Database: SELECT cart with joins
    Database-->>Module: Full cart data
    Module-->>RouteHandler: Cart object
    RouteHandler-->>Client: 200 { cart }
```

### Lifecycle Phases

| Phase | Duration | Purpose | Key Components |
|-------|----------|---------|----------------|
| **1. Request Reception** | <1ms | Receive HTTP request | Express server |
| **2. Middleware Processing** | 5-20ms | Auth, validation, parsing | Middleware chain |
| **3. Route Resolution** | <1ms | Find handler | File-based router |
| **4. Workflow Execution** | 50-500ms | Orchestrate operations | Workflow engine |
| **5. Module Operations** | 10-200ms | Business logic + DB | Commerce modules |
| **6. Database Queries** | 5-100ms | Data persistence | MikroORM + PostgreSQL |
| **7. Event Emission** | 1-5ms | Publish events | Event bus |
| **8. Response Formation** | 5-20ms | Serialize response | Route handler |
| **9. Response Transmission** | Network dependent | Send to client | Express |

**Total:** ~80-850ms (depends on complexity and network)

---

## API Request Flow

### Example: Create Cart

**API Endpoint:** `POST /store/carts`

#### 1. File-Based Route

**File:** `/packages/medusa/src/api/store/carts/route.ts`

```typescript
import { createCartWorkflow } from "@medusajs/core-flows"
import {
  AuthenticatedMedusaRequest,
  MedusaResponse,
} from "@medusajs/framework/http"
import { HttpTypes } from "@medusajs/framework/types"

export const POST = async (
  req: AuthenticatedMedusaRequest<
    HttpTypes.StoreCreateCart,
    HttpTypes.SelectParams
  >,
  res: MedusaResponse<HttpTypes.StoreCartResponse>
) => {
  // Step 1: Prepare workflow input
  const workflowInput = {
    ...req.validatedBody,  // Validated by middleware
    customer_id: req.auth_context?.actor_id,  // From auth middleware
  }

  // Step 2: Execute workflow
  const { result } = await createCartWorkflow(req.scope).run({
    input: workflowInput,
  })

  // Step 3: Refetch with requested fields
  const cart = await refetchCart(
    result.id,
    req.scope,
    req.queryConfig.fields
  )

  // Step 4: Send response
  res.status(200).json({ cart })
}
```

#### 2. Request Object Anatomy

```typescript
// AuthenticatedMedusaRequest properties:
{
  // Standard Express properties
  body: { /* request body */ },
  params: { /* route params */ },
  query: { /* query string */ },
  headers: { /* HTTP headers */ },

  // Medusa additions
  validatedBody: { /* Zod-validated body */ },
  validatedQuery: { /* Zod-validated query */ },
  auth_context: {
    actor_id: "cus_123",     // Authenticated user ID
    auth_identity_id: "...",  // Auth identity ID
    app_metadata: { /* ... */ }
  },
  scope: Container,           // DI container (Awilix)
  queryConfig: {
    fields: ["id", "email", "items.*"],  // Requested fields
    pagination: { limit: 20, offset: 0 }
  }
}
```

#### 3. Middleware Chain

```mermaid
graph LR
    Request[Raw Request] --> Parse[Body Parser]
    Parse --> CORS[CORS Handler]
    CORS --> Auth[Authentication]
    Auth --> Validate[Zod Validation]
    Validate --> Inject[Inject DI Container]
    Inject --> Handler[Route Handler]

    style Request fill:#f9f9f9
    style Handler fill:#e8f5e9
```

**Middleware Example:**

**File:** `/packages/medusa/src/api/middlewares.ts`

```typescript
import { authenticate } from "@medusajs/framework/http"
import { z } from "zod"

// Global middlewares
export const config = {
  routes: {
    "/store/carts": {
      method: ["POST"],
      middlewares: [
        authenticate("customer", ["session", "bearer"]),  // Auth
        validateBody(z.object({
          region_id: z.string().optional(),
          items: z.array(z.object({
            variant_id: z.string(),
            quantity: z.number().int().positive(),
          })).optional(),
        })),
      ],
    },
  },
}
```

---

## Workflow Execution Flow

### Workflow Anatomy

**File:** `/packages/core/core-flows/src/cart/workflows/create-carts.ts`

```typescript
import {
  createWorkflow,
  WorkflowData,
  WorkflowResponse,
  parallelize,
  transform,
} from "@medusajs/framework/workflows-sdk"

export const createCartWorkflow = createWorkflow(
  "create-cart",  // Workflow ID
  (input: WorkflowData<CreateCartInput>) => {
    // Step 1: Parallel operations
    const [salesChannel, region, customerData] = parallelize(
      findSalesChannelStep({ salesChannelId: input.sales_channel_id }),
      findOneOrAnyRegionStep({ regionId: input.region_id }),
      findOrCreateCustomerStep({ customerId: input.customer_id })
    )

    // Step 2: Data transformation
    const cartInput = transform({ input, region, customerData }, (data) => {
      return {
        ...data.input,
        currency_code: data.region.currency_code,
        region_id: data.region.id,
        customer_id: data.customerData.customer?.id,
      }
    })

    // Step 3: Create cart
    const carts = createCartsStep([cartInput])
    const cart = transform({ carts }, (data) => data.carts[0])

    // Step 4: Emit event
    emitEventStep({
      eventName: CartWorkflowEvents.CREATED,
      data: { id: cart.id },
    })

    // Step 5: Return result
    return new WorkflowResponse(cart)
  }
)
```

### Workflow Execution Sequence

```mermaid
sequenceDiagram
    autonumber
    participant Route as Route Handler
    participant WF as Workflow Engine
    participant Step1 as findSalesChannelStep
    participant Step2 as findRegionStep
    participant Step3 as findCustomerStep
    participant Step4 as createCartsStep
    participant Step5 as emitEventStep
    participant DB as Database

    Route->>WF: createCartWorkflow.run({ input })
    WF->>WF: Initialize workflow context

    par Parallel Execution
        WF->>Step1: Execute
        Step1->>DB: Query sales_channel
        DB-->>Step1: SalesChannel
        Step1-->>WF: Result

        WF->>Step2: Execute
        Step2->>DB: Query region
        DB-->>Step2: Region
        Step2-->>WF: Result

        WF->>Step3: Execute
        Step3->>DB: Query/Create customer
        DB-->>Step3: Customer
        Step3-->>WF: Result
    end

    WF->>WF: Transform data

    WF->>Step4: Execute
    Step4->>DB: INSERT cart
    DB-->>Step4: Cart ID
    Step4-->>WF: Cart object

    WF->>Step5: Execute
    Step5->>Step5: Publish event
    Step5-->>WF: Event emitted

    WF-->>Route: Workflow result { cart }
```

### Step Anatomy

**File:** `/packages/core/core-flows/src/cart/steps/create-carts.ts`

```typescript
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"
import { Modules } from "@medusajs/framework/utils"

export const createCartsStep = createStep(
  "create-carts",  // Step ID
  async (input: CreateCartInput[], { container }) => {
    // Resolve cart module from DI container
    const cartService = container.resolve(Modules.CART)

    // Execute business logic
    const carts = await cartService.create(input)

    // Return result with compensation function
    return new StepResponse(
      carts,
      // Compensation: runs if workflow fails later
      async () => {
        await cartService.delete(carts.map((c) => c.id))
      }
    )
  }
)
```

### Workflow Compensation (Rollback)

```mermaid
graph TB
    Input[Workflow Input]
    Step1[Step 1: Find Region]
    Step2[Step 2: Create Cart]
    Step3[Step 3: Add Items]
    Step4[Step 4: Calculate Tax]
    Error[Error Occurs!]

    Comp4[Compensate Step 4:<br/>Clear tax calculations]
    Comp3[Compensate Step 3:<br/>Delete line items]
    Comp2[Compensate Step 2:<br/>Delete cart]
    Comp1[Compensate Step 1:<br/>No action needed]
    ErrorResponse[Error Response]

    Input --> Step1
    Step1 --> Step2
    Step2 --> Step3
    Step3 --> Step4
    Step4 --> Error

    Error -.->|Rollback| Comp4
    Comp4 -.-> Comp3
    Comp3 -.-> Comp2
    Comp2 -.-> Comp1
    Comp1 -.-> ErrorResponse

    style Step1 fill:#e8f5e9
    style Step2 fill:#e8f5e9
    style Step3 fill:#e8f5e9
    style Step4 fill:#e8f5e9
    style Error fill:#ffebee
    style Comp1 fill:#fff9c4
    style Comp2 fill:#fff9c4
    style Comp3 fill:#fff9c4
    style Comp4 fill:#fff9c4
```

💡 **Aha Moment:** Workflows automatically rollback all completed steps if any step fails - like database transactions but across multiple modules!

---

## Event Bus Flow

### Event-Driven Communication

```mermaid
graph TB
    subgraph "Cart Module"
        CartCreate[Create Cart]
        EmitEvent[Emit cart.created]
    end

    subgraph "Event Bus (Redis/Local)"
        EventBus[Event Bus]
    end

    subgraph "Subscribers"
        Analytics[Analytics Subscriber]
        Email[Email Subscriber]
        Inventory[Inventory Subscriber]
    end

    CartCreate --> EmitEvent
    EmitEvent --> EventBus
    EventBus -->|Publish| Analytics
    EventBus -->|Publish| Email
    EventBus -->|Publish| Inventory

    Analytics --> TrackEvent[Track event in analytics]
    Email --> SendWelcome[Send welcome email]
    Inventory --> CheckStock[Check stock levels]

    style EmitEvent fill:#ffd93d
    style EventBus fill:#ffd93d
    style Analytics fill:#e1f5ff
    style Email fill:#e1f5ff
    style Inventory fill:#e1f5ff
```

### Publishing Events

**In a Workflow:**

```typescript
import { emitEventStep } from "@medusajs/core-flows"
import { CartWorkflowEvents } from "@medusajs/framework/utils"

// Inside workflow definition
emitEventStep({
  eventName: CartWorkflowEvents.CREATED,
  data: {
    id: cart.id,
    customer_id: cart.customer_id,
    total: cart.total,
  },
})
```

### Subscribing to Events

**File:** `/packages/medusa/src/subscribers/cart-created.ts`

```typescript
import { SubscriberArgs } from "@medusajs/framework"

export default async function handleCartCreated({
  event,
  container,
}: SubscriberArgs<{ id: string }>) {
  const logger = container.resolve("logger")
  const analyticsService = container.resolve("analyticsService")

  logger.info(`Cart created: ${event.data.id}`)

  // Track in analytics
  await analyticsService.track({
    event: "cart_created",
    cartId: event.data.id,
    timestamp: event.metadata.occurred_at,
  })
}

export const config = {
  event: "cart.created",  // Event to subscribe to
}
```

### Event Flow Sequence

```mermaid
sequenceDiagram
    autonumber
    participant Workflow
    participant EventBus
    participant Sub1 as Analytics Subscriber
    participant Sub2 as Email Subscriber
    participant Sub3 as Custom Subscriber

    Workflow->>EventBus: emitEvent("cart.created", { id })
    EventBus->>EventBus: Store event

    par Parallel Execution
        EventBus->>Sub1: Trigger subscriber
        Sub1->>Sub1: Track analytics
        Sub1-->>EventBus: Done

        EventBus->>Sub2: Trigger subscriber
        Sub2->>Sub2: Send email
        Sub2-->>EventBus: Done

        EventBus->>Sub3: Trigger subscriber
        Sub3->>Sub3: Custom logic
        Sub3-->>EventBus: Done
    end

    EventBus-->>Workflow: Events published
```

### Common Event Types

| Event Name | Emitter | Data | Use Cases |
|------------|---------|------|-----------|
| `cart.created` | Cart Module | `{ id, customer_id }` | Analytics, welcome email |
| `order.placed` | Order Module | `{ id, total, items }` | Inventory, fulfillment, email |
| `payment.captured` | Payment Module | `{ id, amount }` | Accounting, notifications |
| `product.created` | Product Module | `{ id, title }` | Search indexing, cache invalidation |
| `customer.created` | Customer Module | `{ id, email }` | Email campaigns, analytics |

---

## Module Communication

Modules communicate through three patterns:

### 1. Direct Module Access (Synchronous)

```typescript
// In a workflow or route handler
const productService = container.resolve(Modules.PRODUCT)
const pricingService = container.resolve(Modules.PRICING)

// Direct method calls
const variants = await productService.listProductVariants({ id: variantIds })
const prices = await pricingService.calculatePrices(
  { variantId: variants.map((v) => v.id) },
  { context: { region_id: regionId } }
)
```

**Flow:**

```mermaid
graph LR
    Workflow[Workflow] -->|resolve| Container[DI Container]
    Container -->|inject| ProductService[Product Service]
    Container -->|inject| PricingService[Pricing Service]

    Workflow -->|call| ProductService
    Workflow -->|call| PricingService

    ProductService -->|query| DB[(Database)]
    PricingService -->|query| DB

    style Workflow fill:#a8e6cf
    style Container fill:#ffd93d
    style ProductService fill:#e1f5ff
    style PricingService fill:#e1f5ff
```

### 2. Link Modules (Relational)

**Purpose:** Define relationships between modules without tight coupling.

**Example:** Cart Line Items → Product Variants

```typescript
// Define link (automatic in framework)
defineLink(
  { linkable: Modules.CART, field: "cart.items.variant_id" },
  { linkable: Modules.PRODUCT, field: "productVariant.id" }
)

// Query across modules
const cart = await query.graph({
  entity: "cart",
  fields: ["id", "email", "items.*", "items.variant.*"],
  filters: { id: "cart_123" },
})

// Result includes variant data in line items!
{
  id: "cart_123",
  email: "customer@example.com",
  items: [
    {
      id: "item_1",
      quantity: 2,
      variant: {  // ← Joined from Product module!
        id: "var_123",
        title: "Red T-Shirt",
        price: 1999
      }
    }
  ]
}
```

**Link Module Flow:**

```mermaid
graph TB
    Query[Query: Get cart with items and variants]
    CartModule[Cart Module]
    LinkModule[Link Module]
    ProductModule[Product Module]
    DB[(Database)]

    Query -->|Request| CartModule
    CartModule -->|Get cart & items| DB
    CartModule -->|Need variants| LinkModule
    LinkModule -->|Resolve link| ProductModule
    ProductModule -->|Get variants| DB
    LinkModule -->|Merge data| CartModule
    CartModule -->|Return complete cart| Query

    style LinkModule fill:#ffd93d
    style CartModule fill:#e1f5ff
    style ProductModule fill:#e1f5ff
```

### 3. Event Bus (Asynchronous)

**Use Case:** One module needs to notify others without waiting.

```typescript
// Order module places an order
await emitEventStep({
  eventName: "order.placed",
  data: { id: order.id, items: order.items },
})

// Inventory module reserves stock
export default async function handleOrderPlaced({ event, container }) {
  const inventoryService = container.resolve(Modules.INVENTORY)
  await inventoryService.reserveQuantity(event.data.items)
}

// Fulfillment module creates shipment
export default async function handleOrderPlaced({ event, container }) {
  const fulfillmentService = container.resolve(Modules.FULFILLMENT)
  await fulfillmentService.createShipment(event.data.id)
}
```

---

## Real-World Examples

### Example 1: Complete Cart Creation

**Request:** `POST /store/carts`

**Body:**
```json
{
  "region_id": "reg_01JF8B",
  "items": [
    {
      "variant_id": "var_01JF8C",
      "quantity": 2
    }
  ],
  "email": "customer@example.com"
}
```

**Complete Flow:**

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Router
    participant Middleware
    participant Handler
    participant Workflow
    participant Region
    participant Product
    participant Pricing
    participant Cart
    participant Events
    participant DB

    Client->>Router: POST /store/carts {body}
    Router->>Middleware: Process request
    Middleware->>Middleware: Validate body (Zod)
    Middleware->>Handler: Validated request

    Handler->>Workflow: createCartWorkflow.run()

    par Parallel Steps
        Workflow->>Region: findOneOrAnyRegionStep
        Region->>DB: SELECT region
        DB-->>Region: reg_01JF8B
        Region-->>Workflow: Region object

        Workflow->>Product: getVariantsStep
        Product->>DB: SELECT variants
        DB-->>Product: var_01JF8C
        Product-->>Workflow: Variant object
    end

    Workflow->>Pricing: calculatePrices
    Pricing->>DB: Query price rules
    DB-->>Pricing: Price rules
    Pricing-->>Workflow: Calculated prices

    Workflow->>Cart: createCartsStep
    Cart->>DB: BEGIN TRANSACTION
    Cart->>DB: INSERT cart
    Cart->>DB: INSERT line_items (2x)
    Cart->>DB: COMMIT
    DB-->>Cart: Cart ID
    Cart-->>Workflow: Cart object

    Workflow->>Events: emitEventStep("cart.created")
    Events->>Events: Publish to subscribers
    Events-->>Workflow: Done

    Workflow-->>Handler: { result: cart }
    Handler->>Cart: refetchCart (with relations)
    Cart->>DB: SELECT cart with joins
    DB-->>Cart: Full cart data
    Cart-->>Handler: Cart object

    Handler-->>Client: 200 { cart }
```

**Database Queries Executed:**

```sql
-- 1. Find region
SELECT * FROM region WHERE id = 'reg_01JF8B';

-- 2. Find variant
SELECT * FROM product_variant WHERE id = 'var_01JF8C';

-- 3. Calculate prices
SELECT * FROM price_rule
WHERE region_id = 'reg_01JF8B'
  AND variant_id = 'var_01JF8C';

-- 4. Create cart (transaction)
BEGIN;
INSERT INTO cart (id, region_id, email, currency_code)
VALUES ('cart_01JF8D', 'reg_01JF8B', 'customer@example.com', 'USD');

INSERT INTO line_item (id, cart_id, variant_id, quantity, unit_price)
VALUES ('item_01JF8E', 'cart_01JF8D', 'var_01JF8C', 2, 1999);
COMMIT;

-- 5. Refetch with relations
SELECT
  c.*,
  li.*,
  v.*,
  p.*
FROM cart c
LEFT JOIN line_item li ON li.cart_id = c.id
LEFT JOIN product_variant v ON v.id = li.variant_id
LEFT JOIN product p ON p.id = v.product_id
WHERE c.id = 'cart_01JF8D';
```

**Response:**

```json
{
  "cart": {
    "id": "cart_01JF8D",
    "email": "customer@example.com",
    "region_id": "reg_01JF8B",
    "currency_code": "USD",
    "items": [
      {
        "id": "item_01JF8E",
        "variant_id": "var_01JF8C",
        "quantity": 2,
        "unit_price": 1999,
        "total": 3998,
        "variant": {
          "id": "var_01JF8C",
          "title": "Red T-Shirt - Medium",
          "product": {
            "id": "prod_01JF8A",
            "title": "Red T-Shirt"
          }
        }
      }
    ],
    "total": 3998
  }
}
```

---

### Example 2: Place Order from Cart

**Request:** `POST /store/carts/:id/complete`

**Flow:**

```mermaid
graph TB
    Request[POST /store/carts/:id/complete]
    Validate[Validate cart can be completed]
    Payment[Authorize payment]
    Order[Create order from cart]
    Inventory[Reserve inventory]
    Fulfillment[Create fulfillment]
    Events[Emit order.placed event]
    Email[Send confirmation email]
    Response[Return order]

    Request --> Validate
    Validate -->|Cart valid| Payment
    Payment -->|Payment authorized| Order
    Order --> Inventory
    Inventory --> Fulfillment
    Fulfillment --> Events
    Events --> Email
    Events --> Response

    Validate -.->|Cart invalid| Error[400 Error]
    Payment -.->|Payment failed| Error2[402 Error]

    style Request fill:#61dafb
    style Order fill:#e8f5e9
    style Events fill:#ffd93d
    style Error fill:#ffebee
    style Error2 fill:#ffebee
```

**Workflow Steps:**

1. **Validate Cart:** Check items in stock, prices current
2. **Authorize Payment:** Charge payment method
3. **Create Order:** Transform cart → order
4. **Reserve Inventory:** Decrement stock levels
5. **Create Fulfillment:** Prepare for shipping
6. **Emit Events:** Notify subscribers
7. **Send Email:** Confirmation to customer

**Code:**

```typescript
export const completeCartWorkflow = createWorkflow(
  "complete-cart",
  (input: { cart_id: string }) => {
    // Step 1: Retrieve and validate cart
    const cart = retrieveCartStep({ id: input.cart_id })
    validateCartStep({ cart })

    // Step 2: Authorize payment
    const payment = authorizePaymentStep({
      cartId: cart.id,
      amount: cart.total,
    })

    // Step 3: Create order
    const order = createOrderFromCartStep({
      cart,
      paymentId: payment.id,
    })

    // Step 4: Reserve inventory
    reserveInventoryStep({
      items: order.items,
    })

    // Step 5: Create fulfillment
    const fulfillment = createFulfillmentStep({
      orderId: order.id,
    })

    // Step 6: Emit event
    emitEventStep({
      eventName: "order.placed",
      data: { id: order.id },
    })

    return new WorkflowResponse(order)
  }
)
```

---

## Database Transaction Flow

### Transaction Management

**MikroORM Unit of Work Pattern:**

```typescript
// Workflow step with transaction
export const createProductStep = createStep(
  "create-product",
  async (input: CreateProductInput, { container }) => {
    const productService = container.resolve(Modules.PRODUCT)

    // Automatic transaction!
    const product = await productService.create({
      title: input.title,
      variants: input.variants,  // Creates variants in same transaction
    })

    return new StepResponse(product, async () => {
      // Compensation: delete product
      await productService.delete(product.id)
    })
  }
)
```

**Transaction Flow:**

```mermaid
sequenceDiagram
    autonumber
    participant Step
    participant Service
    participant EntityManager
    participant DB

    Step->>Service: create(data)
    Service->>EntityManager: BEGIN TRANSACTION
    Service->>EntityManager: persist(product)
    Service->>EntityManager: persist(variants)
    Service->>EntityManager: flush()
    EntityManager->>DB: INSERT product
    EntityManager->>DB: INSERT variants
    EntityManager->>DB: COMMIT
    DB-->>EntityManager: Success
    EntityManager-->>Service: Entities
    Service-->>Step: Product with variants
```

### Rollback Example

```mermaid
graph TB
    Start[Start Transaction]
    Op1[Operation 1: Create Product]
    Op2[Operation 2: Create Variants]
    Op3[Operation 3: Update Inventory]
    Error[Error: Inventory check failed!]
    Rollback[ROLLBACK Transaction]
    End[Transaction Aborted]

    Start --> Op1
    Op1 -->|Success| Op2
    Op2 -->|Success| Op3
    Op3 --> Error
    Error -.-> Rollback
    Rollback -.-> End

    style Op1 fill:#e8f5e9
    style Op2 fill:#e8f5e9
    style Error fill:#ffebee
    style Rollback fill:#fff9c4
```

**Database State:**
- ✅ Before error: No changes committed
- ✅ After rollback: Database unchanged
- ✅ Atomic: All or nothing

---

## Troubleshooting Data Flow

### Common Issues & Solutions

#### Issue 1: Request Times Out

**Symptom:** Request takes >30 seconds, times out

**Possible Causes:**
1. Slow database query (missing index)
2. External API call hanging
3. Infinite loop in workflow

**Debug Steps:**

```typescript
// Add logging to workflow
export const myWorkflow = createWorkflow("my-workflow", (input) => {
  console.time("step-1")
  const result1 = step1(input)
  console.timeEnd("step-1")

  console.time("step-2")
  const result2 = step2(result1)
  console.timeEnd("step-2")

  // ...
})
```

**Check Database:**
```sql
-- Find slow queries
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;
```

---

#### Issue 2: Data Inconsistency

**Symptom:** Cart shows wrong total, inventory incorrect

**Possible Causes:**
1. Event bus message lost
2. Cache not invalidated
3. Workflow compensation didn't run

**Debug Steps:**

```typescript
// Check event bus
const eventBus = container.resolve("eventBus")
const events = await eventBus.listEvents({ type: "cart.created" })
console.log("Events:", events)

// Check cache
const cacheService = container.resolve("cacheService")
const cached = await cacheService.get("cart:cart_123")
console.log("Cached cart:", cached)

// Force refetch from database
const cart = await cartService.retrieve("cart_123", { reload: true })
```

---

#### Issue 3: Workflow Fails Silently

**Symptom:** Workflow doesn't throw error, but doesn't complete

**Debug Steps:**

```typescript
// Check workflow execution
const { result, errors } = await myWorkflow(container).run({
  input: data,
  throwOnError: false,  // Don't throw, return errors
})

if (errors) {
  console.error("Workflow errors:", errors)
}
```

---

## Key Takeaways

### ✅ Remember These Flows

1. **HTTP Request** → Middleware → Route Handler → Workflow → Modules → Database
2. **Workflow Step** → Compensation function (for rollback)
3. **Event Emission** → Event Bus → Subscribers (parallel)
4. **Module Communication** → Direct call, Link modules, or Events
5. **Transactions** → Automatic per step, manual rollback via compensation

### 🧠 Mental Models

- **Request = River flow** through layers
- **Workflow = Orchestration** with automatic rollback
- **Event Bus = Broadcast** to multiple listeners
- **Modules = Islands** connected by bridges (links) and signals (events)

### 🎯 Debugging Strategy

1. **Add logging** at each layer
2. **Check database** for slow queries
3. **Inspect events** in event bus
4. **Validate cache** is fresh
5. **Test workflows** in isolation

---

## Next Steps

1. Trace a request through the codebase using a debugger
2. Add custom logging to a workflow
3. Create a new event subscriber
4. Write a workflow with compensation logic
5. Optimize a slow database query

---

## Additional Resources

- **Workflow SDK Docs:** https://docs.medusajs.com/resources/workflows
- **Event Bus Guide:** https://docs.medusajs.com/resources/event-bus
- **Module SDK:** https://docs.medusajs.com/resources/module-sdk
- **MikroORM Transactions:** https://mikro-orm.io/docs/transactions

---

**File Path:** `/home/user/medusa/docs/learning/DATA_FLOW_GUIDE.md`
**Related Files:**
- `ARCHITECTURE_OVERVIEW.md` - System architecture
- `PROJECT_STRUCTURE.md` - Codebase navigation
- `TECH_STACK_GUIDE.md` - Technology details
