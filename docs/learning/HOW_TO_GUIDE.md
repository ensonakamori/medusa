# How-To Guide

**Step-by-step recipes for common Medusa development tasks**

**Last Updated:** November 19, 2025
**Version:** 2.11.3

---

## Table of Contents

1. [How to Create a New Module](#how-to-create-a-new-module)
2. [How to Add an API Endpoint](#how-to-add-an-api-endpoint)
3. [How to Create a Workflow](#how-to-create-a-workflow)
4. [How to Add a Database Field/Migration](#how-to-add-a-database-fieldmigration)
5. [How to Integrate a Provider](#how-to-integrate-a-provider)
6. [How to Add a Route to Admin Dashboard](#how-to-add-a-route-to-admin-dashboard)
7. [How to Debug a Module](#how-to-debug-a-module)
8. [How to Write Tests](#how-to-write-tests)
9. [How to Use the CLI](#how-to-use-the-cli)

---

## How to Create a New Module

**Goal:** Create a custom "Reviews" module for product reviews.

**Time:** 15-20 minutes

### Step 1: Create Directory Structure

```bash
cd /home/user/medusa/packages/modules
mkdir -p review/src/{models,services,migrations,types}
cd review
```

### Step 2: Initialize package.json

```bash
# Create package.json
cat > package.json << 'EOF'
{
  "name": "@medusajs/review",
  "version": "0.0.1",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "test": "jest"
  },
  "dependencies": {
    "@medusajs/framework": "workspace:^",
    "@medusajs/utils": "workspace:^"
  },
  "devDependencies": {
    "typescript": "^5.6.2"
  }
}
EOF
```

### Step 3: Create the Model

**File:** `/home/user/medusa/packages/modules/review/src/models/review.ts`

```typescript
import { model } from "@medusajs/framework/utils"

const Review = model
  .define("Review", {
    id: model.id({ prefix: "rev" }).primaryKey(),
    product_id: model.text(),
    customer_id: model.text(),
    rating: model.number(), // 1-5
    title: model.text(),
    content: model.text().nullable(),
    verified_purchase: model.boolean().default(false),
    metadata: model.json().nullable(),
  })
  .indexes([
    {
      name: "IDX_review_product_id",
      on: ["product_id"],
    },
    {
      name: "IDX_review_customer_id",
      on: ["customer_id"],
    },
  ])

export default Review
```

**File:** `/home/user/medusa/packages/modules/review/src/models/index.ts`

```typescript
export { default as Review } from "./review"
```

### Step 4: Create the Service

**File:** `/home/user/medusa/packages/modules/review/src/services/review-module.ts`

```typescript
import { ModulesSdkUtils } from "@medusajs/framework/utils"
import { Review } from "../models"

export class ReviewModuleService extends ModulesSdkUtils.MedusaService({
  Review,
}) {
  // Inherits:
  // - createReviews() / createReview()
  // - updateReviews() / updateReview()
  // - deleteReviews() / deleteReview()
  // - listReviews() / retrieveReview()

  // Add custom methods
  async getAverageRating(productId: string): Promise<number> {
    const reviews = await this.listReviews({ product_id: productId })

    if (reviews.length === 0) return 0

    const sum = reviews.reduce((acc, review) => acc + review.rating, 0)
    return sum / reviews.length
  }
}
```

**File:** `/home/user/medusa/packages/modules/review/src/services/index.ts`

```typescript
export { ReviewModuleService } from "./review-module"
```

### Step 5: Create Module Definition

**File:** `/home/user/medusa/packages/modules/review/src/index.ts`

```typescript
import { Module } from "@medusajs/framework/utils"
import { ReviewModuleService } from "./services"

export const REVIEW_MODULE = "review"

export default Module(REVIEW_MODULE, {
  service: ReviewModuleService,
})
```

### Step 6: Create TypeScript Config

**File:** `/home/user/medusa/packages/modules/review/tsconfig.json`

```json
{
  "extends": "../../../_tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "**/__tests__", "**/__mocks__"]
}
```

### Step 7: Register Module in Medusa Config

**File:** `/home/user/medusa/medusa-config.js` (or `.ts`)

```javascript
module.exports = {
  // ... other config
  modules: {
    review: {
      resolve: "@medusajs/review",
    },
  },
}
```

### Step 8: Build and Test

```bash
# Build the module
cd /home/user/medusa/packages/modules/review
yarn build

# Run migrations (creates tables)
cd /home/user/medusa
yarn medusa db:migrate

# Start dev server
yarn dev
```

### ✅ Quick Check

```bash
# Check if module is loaded
curl http://localhost:9000/admin/modules

# Should see "review" in the list
```

### ⚠️ Common Pitfalls

1. **Missing Module Registration:** Module won't load unless added to `medusa-config.js`
2. **Wrong Model Export:** Must use `export default` for models
3. **Missing Indexes:** Add indexes for foreign keys (product_id, customer_id)

---

## How to Add an API Endpoint

**Goal:** Add admin endpoints to manage reviews.

**Time:** 10-15 minutes

### Step 1: Create Route File

**File:** `/home/user/medusa/packages/medusa/src/api/admin/reviews/route.ts`

```typescript
import {
  AuthenticatedMedusaRequest,
  MedusaResponse,
} from "@medusajs/framework/http"
import { HttpTypes } from "@medusajs/framework/types"

// GET /admin/reviews - List reviews
export const GET = async (
  req: AuthenticatedMedusaRequest,
  res: MedusaResponse
) => {
  const reviewService = req.scope.resolve("review")

  const { data: reviews, metadata } = await reviewService.listAndCountReviews(
    req.filterableFields,
    {
      skip: req.queryConfig.pagination?.offset || 0,
      take: req.queryConfig.pagination?.limit || 20,
    }
  )

  res.json({
    reviews,
    count: metadata.count,
    offset: metadata.skip,
    limit: metadata.take,
  })
}

// POST /admin/reviews - Create review
export const POST = async (
  req: AuthenticatedMedusaRequest<CreateReviewInput>,
  res: MedusaResponse
) => {
  const reviewService = req.scope.resolve("review")

  const review = await reviewService.createReview(req.validatedBody)

  res.status(200).json({ review })
}
```

### Step 2: Create Single Item Routes

**File:** `/home/user/medusa/packages/medusa/src/api/admin/reviews/[id]/route.ts`

```typescript
import {
  AuthenticatedMedusaRequest,
  MedusaResponse,
} from "@medusajs/framework/http"

// GET /admin/reviews/:id - Get single review
export const GET = async (
  req: AuthenticatedMedusaRequest,
  res: MedusaResponse
) => {
  const reviewService = req.scope.resolve("review")

  const review = await reviewService.retrieveReview(req.params.id)

  res.json({ review })
}

// POST /admin/reviews/:id - Update review
export const POST = async (
  req: AuthenticatedMedusaRequest<UpdateReviewInput>,
  res: MedusaResponse
) => {
  const reviewService = req.scope.resolve("review")

  const review = await reviewService.updateReview(
    req.params.id,
    req.validatedBody
  )

  res.json({ review })
}

// DELETE /admin/reviews/:id - Delete review
export const DELETE = async (
  req: AuthenticatedMedusaRequest,
  res: MedusaResponse
) => {
  const reviewService = req.scope.resolve("review")

  await reviewService.deleteReview(req.params.id)

  res.status(200).json({
    id: req.params.id,
    deleted: true,
  })
}
```

### Step 3: Create Validators

**File:** `/home/user/medusa/packages/medusa/src/api/admin/reviews/validators.ts`

```typescript
import { z } from "zod"

export const AdminCreateReview = z.object({
  product_id: z.string(),
  customer_id: z.string(),
  rating: z.number().min(1).max(5),
  title: z.string(),
  content: z.string().optional(),
  verified_purchase: z.boolean().default(false),
  metadata: z.record(z.unknown()).optional(),
})

export const AdminUpdateReview = z.object({
  rating: z.number().min(1).max(5).optional(),
  title: z.string().optional(),
  content: z.string().optional(),
  metadata: z.record(z.unknown()).optional(),
})

export const AdminGetReviewParams = z.object({
  id: z.string(),
})

export type AdminCreateReviewType = z.infer<typeof AdminCreateReview>
export type AdminUpdateReviewType = z.infer<typeof AdminUpdateReview>
```

### Step 4: Create Middleware

**File:** `/home/user/medusa/packages/medusa/src/api/admin/reviews/middlewares.ts`

```typescript
import { MiddlewareRoute } from "@medusajs/framework/http"
import { validateAndTransformBody } from "../../utils/validate-body"
import { validateAndTransformQuery } from "../../utils/validate-query"
import { AdminCreateReview, AdminUpdateReview } from "./validators"

export const adminReviewRoutesMiddlewares: MiddlewareRoute[] = [
  {
    method: ["POST"],
    matcher: "/admin/reviews",
    middlewares: [validateAndTransformBody(AdminCreateReview)],
  },
  {
    method: ["POST"],
    matcher: "/admin/reviews/:id",
    middlewares: [validateAndTransformBody(AdminUpdateReview)],
  },
]
```

### Step 5: Register Middleware

**File:** `/home/user/medusa/packages/medusa/src/api/middlewares.ts`

Add to the existing file:

```typescript
import { adminReviewRoutesMiddlewares } from "./admin/reviews/middlewares"

// ... existing imports

export const middlewares: MiddlewareRoute[] = [
  // ... existing middlewares
  ...adminReviewRoutesMiddlewares,
]
```

### ✅ Quick Check

```bash
# Test endpoints
curl -X GET http://localhost:9000/admin/reviews \
  -H "Authorization: Bearer YOUR_TOKEN"

curl -X POST http://localhost:9000/admin/reviews \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "prod_123",
    "customer_id": "cust_456",
    "rating": 5,
    "title": "Great product!"
  }'
```

### ⚠️ Common Pitfalls

1. **Missing Middleware Registration:** Validators won't run
2. **Wrong HTTP Method:** Use POST for updates, not PUT
3. **Missing Authentication:** Add `AuthenticatedMedusaRequest` type

---

## How to Create a Workflow

**Goal:** Create a workflow to approve reviews.

**Time:** 15 minutes

### Step 1: Create Steps Directory

```bash
mkdir -p /home/user/medusa/packages/core/core-flows/src/review/{workflows,steps}
```

### Step 2: Create Validation Step

**File:** `/home/user/medusa/packages/core/core-flows/src/review/steps/validate-review.ts`

```typescript
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"
import { MedusaError } from "@medusajs/framework/utils"

export const validateReviewStepId = "validate-review"

export const validateReviewStep = createStep(
  validateReviewStepId,
  async (data: { rating: number; content?: string }) => {
    // Validate rating
    if (data.rating < 1 || data.rating > 5) {
      throw new MedusaError(
        MedusaError.Types.INVALID_DATA,
        `Rating must be between 1 and 5. Got: ${data.rating}`
      )
    }

    // Validate content length
    if (data.content && data.content.length > 1000) {
      throw new MedusaError(
        MedusaError.Types.INVALID_DATA,
        `Review content too long (max 1000 characters)`
      )
    }

    return new StepResponse(data)
  }
)
```

### Step 3: Create Review Step

**File:** `/home/user/medusa/packages/core/core-flows/src/review/steps/create-review.ts`

```typescript
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

export const createReviewStepId = "create-review"

export interface CreateReviewInput {
  product_id: string
  customer_id: string
  rating: number
  title: string
  content?: string
}

export const createReviewStep = createStep(
  createReviewStepId,
  async (data: CreateReviewInput, { container }) => {
    const reviewService = container.resolve("review")

    const review = await reviewService.createReview(data)

    return new StepResponse(review, review.id)
  },
  async (reviewId, { container }) => {
    // Compensation: Delete the review
    if (!reviewId) return

    const reviewService = container.resolve("review")
    await reviewService.deleteReview(reviewId)
  }
)
```

### Step 4: Create Notification Step

**File:** `/home/user/medusa/packages/core/core-flows/src/review/steps/send-review-notification.ts`

```typescript
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"
import { Modules } from "@medusajs/framework/utils"

export const sendReviewNotificationStepId = "send-review-notification"

export const sendReviewNotificationStep = createStep(
  sendReviewNotificationStepId,
  async (data: { review: any; customer_email: string }, { container }) => {
    const notificationService = container.resolve(Modules.NOTIFICATION)

    await notificationService.createNotifications({
      to: data.customer_email,
      channel: "email",
      template: "review-created",
      data: {
        review_id: data.review.id,
        product_id: data.review.product_id,
      },
    })

    return new StepResponse({ sent: true })
  }
)
```

### Step 5: Create Workflow

**File:** `/home/user/medusa/packages/core/core-flows/src/review/workflows/create-review.ts`

```typescript
import {
  createWorkflow,
  WorkflowData,
  WorkflowResponse,
  transform,
} from "@medusajs/framework/workflows-sdk"
import { Modules } from "@medusajs/framework/utils"
import {
  validateReviewStep,
  createReviewStep,
  sendReviewNotificationStep,
} from "../steps"

export interface CreateReviewWorkflowInput {
  product_id: string
  customer_id: string
  rating: number
  title: string
  content?: string
}

export const createReviewWorkflowId = "create-review"

export const createReviewWorkflow = createWorkflow(
  createReviewWorkflowId,
  (input: WorkflowData<CreateReviewWorkflowInput>) => {
    // Step 1: Validate review data
    validateReviewStep({
      rating: input.rating,
      content: input.content,
    })

    // Step 2: Create review
    const review = createReviewStep(input)

    // Step 3: Get customer email
    const customerEmail = transform({ input }, async (data, { container }) => {
      const customerService = container.resolve(Modules.CUSTOMER)
      const customer = await customerService.retrieveCustomer(data.input.customer_id)
      return customer.email
    })

    // Step 4: Send notification
    sendReviewNotificationStep({
      review,
      customer_email: customerEmail,
    })

    // Return response
    return new WorkflowResponse(review)
  }
)
```

### Step 6: Export Workflow

**File:** `/home/user/medusa/packages/core/core-flows/src/review/index.ts`

```typescript
export * from "./workflows/create-review"
export * from "./steps/create-review"
export * from "./steps/validate-review"
export * from "./steps/send-review-notification"
```

### Step 7: Use Workflow in API

Update the POST route to use the workflow:

```typescript
// /home/user/medusa/packages/medusa/src/api/admin/reviews/route.ts
import { createReviewWorkflow } from "@medusajs/core-flows/review"

export const POST = async (
  req: AuthenticatedMedusaRequest<CreateReviewInput>,
  res: MedusaResponse
) => {
  const { result } = await createReviewWorkflow(req.scope).run({
    input: req.validatedBody,
  })

  res.status(200).json({ review: result })
}
```

### ✅ Quick Check

```bash
# Create review via workflow
curl -X POST http://localhost:9000/admin/reviews \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "prod_123",
    "customer_id": "cust_456",
    "rating": 5,
    "title": "Great!",
    "content": "Loved it!"
  }'

# Check logs for workflow execution
```

### ⚠️ Common Pitfalls

1. **Missing Compensation:** Always add compensation functions to steps
2. **Direct Data Access:** Use `transform()` to access step outputs
3. **Missing StepResponse:** Always return `new StepResponse()`

---

## How to Add a Database Field/Migration

**Goal:** Add `is_approved` field to Review model.

**Time:** 5-10 minutes

### Step 1: Update Model

**File:** `/home/user/medusa/packages/modules/review/src/models/review.ts`

```typescript
const Review = model
  .define("Review", {
    // ... existing fields
    is_approved: model.boolean().default(false), // NEW FIELD
    approved_at: model.dateTime().nullable(),    // NEW FIELD
    approved_by: model.text().nullable(),        // NEW FIELD
  })
  // ... rest of model
```

### Step 2: Generate Migration

```bash
cd /home/user/medusa
yarn medusa db:generate review "AddApprovalFields"
```

This creates a file like:
`/home/user/medusa/packages/modules/review/src/migrations/Migration20251119120000.ts`

### Step 3: Review Generated Migration

```typescript
import { Migration } from "@medusajs/framework/mikro-orm/migrations"

export class Migration20251119120000 extends Migration {
  async up(): Promise<void> {
    this.addSql(
      'alter table "review" add column "is_approved" boolean not null default false;'
    )
    this.addSql(
      'alter table "review" add column "approved_at" timestamptz null;'
    )
    this.addSql(
      'alter table "review" add column "approved_by" text null;'
    )
  }

  async down(): Promise<void> {
    this.addSql('alter table "review" drop column "is_approved";')
    this.addSql('alter table "review" drop column "approved_at";')
    this.addSql('alter table "review" drop column "approved_by";')
  }
}
```

### Step 4: Run Migration

```bash
# Run migration
yarn medusa db:migrate

# If issues, rollback
yarn medusa db:migrate:rollback
```

### ✅ Quick Check

```bash
# Connect to database
psql medusa_db

# Check table structure
\d review

# Should see new columns
```

### ⚠️ Common Pitfalls

1. **Wrong Module Name:** Use correct module name in `db:generate`
2. **Missing down():** Always implement rollback
3. **Data Loss:** Be careful with NOT NULL on existing data

---

## How to Integrate a Provider

**Goal:** Integrate a custom email provider (e.g., Postmark).

**Time:** 20 minutes

### Step 1: Create Provider Directory

```bash
mkdir -p /home/user/medusa/packages/modules/providers/notification-postmark/src/services
cd /home/user/medusa/packages/modules/providers/notification-postmark
```

### Step 2: Create package.json

```json
{
  "name": "@medusajs/notification-postmark",
  "version": "0.0.1",
  "main": "dist/index.js",
  "dependencies": {
    "@medusajs/framework": "workspace:^",
    "postmark": "^4.0.0"
  }
}
```

### Step 3: Create Provider Service

**File:** `src/services/postmark.ts`

```typescript
import {
  Logger,
  NotificationTypes,
} from "@medusajs/framework/types"
import {
  AbstractNotificationProviderService,
  MedusaError,
} from "@medusajs/framework/utils"
import { Client } from "postmark"

type InjectedDependencies = {
  logger: Logger
}

interface PostmarkConfig {
  apiKey: string
  from: string
}

export class PostmarkNotificationService
  extends AbstractNotificationProviderService {

  static identifier = "notification-postmark"

  protected config_: PostmarkConfig
  protected logger_: Logger
  protected client_: Client

  constructor(
    { logger }: InjectedDependencies,
    options: { api_key: string; from: string }
  ) {
    super()

    this.config_ = {
      apiKey: options.api_key,
      from: options.from,
    }
    this.logger_ = logger
    this.client_ = new Client(this.config_.apiKey)
  }

  async send(
    notification: NotificationTypes.ProviderSendNotificationDTO
  ): Promise<NotificationTypes.ProviderSendNotificationResultsDTO> {
    if (!notification) {
      throw new MedusaError(
        MedusaError.Types.INVALID_DATA,
        "No notification provided"
      )
    }

    const from = notification.from || this.config_.from

    try {
      const result = await this.client_.sendEmail({
        From: from,
        To: notification.to,
        Subject: notification.content?.subject || "",
        HtmlBody: notification.content?.html || "",
        TextBody: notification.content?.text || "",
      })

      return { id: result.MessageID }
    } catch (error) {
      this.logger_.error(`Postmark send failed: ${error.message}`)
      throw error
    }
  }
}
```

### Step 4: Create Module Export

**File:** `src/index.ts`

```typescript
import { ModuleProvider, Modules } from "@medusajs/framework/utils"
import { PostmarkNotificationService } from "./services/postmark"

export default ModuleProvider(Modules.NOTIFICATION, {
  services: [PostmarkNotificationService],
})
```

### Step 5: Configure in medusa-config.js

```javascript
module.exports = {
  modules: {
    notification: {
      resolve: "@medusajs/notification",
      options: {
        providers: [
          {
            resolve: "@medusajs/notification-postmark",
            id: "postmark",
            options: {
              channels: ["email"],
              api_key: process.env.POSTMARK_API_KEY,
              from: process.env.POSTMARK_FROM_EMAIL,
            },
          },
        ],
      },
    },
  },
}
```

### Step 6: Use Provider

```typescript
// In any workflow/route
const notificationService = container.resolve(Modules.NOTIFICATION)

await notificationService.createNotifications({
  to: "customer@example.com",
  channel: "email",
  template: "order-confirmation",
  data: {
    order_id: "order_123",
  },
})
```

### ✅ Quick Check

```bash
# Test sending email
curl -X POST http://localhost:9000/admin/notifications \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "to": "test@example.com",
    "channel": "email",
    "template": "test"
  }'
```

### ⚠️ Common Pitfalls

1. **Missing Environment Variables:** Set API keys in `.env`
2. **Wrong Identifier:** Use consistent `static identifier`
3. **Error Handling:** Always catch and log provider errors

---

## How to Add a Route to Admin Dashboard

**Goal:** Add a reviews management page to admin.

**Time:** 20-30 minutes

### Step 1: Create Route Directory

```bash
cd /home/user/medusa/packages/admin/dashboard/src/routes
mkdir -p reviews/reviews-list
```

### Step 2: Create List Page

**File:** `reviews/reviews-list/reviews-list.tsx`

```tsx
import { SingleColumnPage } from "../../../components/layout/pages"
import { ReviewsTable } from "./components/reviews-table"

export const ReviewsList = () => {
  return (
    <SingleColumnPage>
      <ReviewsTable />
    </SingleColumnPage>
  )
}
```

### Step 3: Create Table Component

**File:** `reviews/reviews-list/components/reviews-table.tsx`

```tsx
import { useQuery } from "@tanstack/react-query"
import { sdk } from "../../../lib/client"
import { DataTable } from "../../../components/table/data-table"

export const ReviewsTable = () => {
  const { data, isLoading } = useQuery({
    queryKey: ["reviews"],
    queryFn: async () => {
      const response = await sdk.client.fetch("/admin/reviews")
      return response.json()
    },
  })

  const columns = [
    {
      header: "Product",
      accessorKey: "product_id",
    },
    {
      header: "Customer",
      accessorKey: "customer_id",
    },
    {
      header: "Rating",
      accessorKey: "rating",
      cell: ({ row }) => `${row.original.rating}/5`,
    },
    {
      header: "Title",
      accessorKey: "title",
    },
    {
      header: "Approved",
      accessorKey: "is_approved",
      cell: ({ row }) => (row.original.is_approved ? "Yes" : "No"),
    },
  ]

  if (isLoading) {
    return <div>Loading...</div>
  }

  return (
    <DataTable
      columns={columns}
      data={data?.reviews || []}
      pageSize={20}
    />
  )
}
```

### Step 4: Add Route Configuration

**File:** `reviews/reviews-list/loader.ts`

```typescript
export const reviewsListLoader = async () => {
  return {}
}
```

### Step 5: Register Route

**File:** Add to your routing configuration (check existing routes for pattern)

```tsx
import { ReviewsList } from "./reviews/reviews-list/reviews-list"

// In your router
{
  path: "/reviews",
  element: <ReviewsList />,
  loader: reviewsListLoader,
}
```

### Step 6: Add Navigation Link

**File:** Update sidebar navigation (location varies by version)

```tsx
{
  label: "Reviews",
  to: "/reviews",
  icon: <StarIcon />,
}
```

### ✅ Quick Check

1. Start dev server: `yarn dev`
2. Navigate to http://localhost:7001/app/reviews
3. Should see reviews table

### ⚠️ Common Pitfalls

1. **Missing Query Key:** Use unique keys for TanStack Query
2. **Wrong API Path:** Match backend route exactly
3. **CORS Issues:** Ensure admin can access API

---

## How to Debug a Module

**Goal:** Debug a module that's not working correctly.

**Time:** Varies

### Method 1: Using Console Logs

```typescript
// In your service
export class ReviewModuleService {
  async createReview(data: CreateReviewInput) {
    console.log("🔍 Creating review with data:", JSON.stringify(data, null, 2))

    const review = await this.create(data)

    console.log("✅ Created review:", review.id)

    return review
  }
}
```

### Method 2: Using VSCode Debugger

**Step 1:** Create `.vscode/launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Medusa",
      "runtimeExecutable": "yarn",
      "runtimeArgs": ["dev"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"],
      "env": {
        "NODE_ENV": "development"
      }
    }
  ]
}
```

**Step 2:** Add breakpoints in your code

**Step 3:** Press F5 to start debugging

### Method 3: Using Logger

```typescript
type InjectedDependencies = {
  logger: Logger
}

export class ReviewModuleService {
  protected logger_: Logger

  constructor({ logger }: InjectedDependencies) {
    this.logger_ = logger
  }

  async createReview(data: CreateReviewInput) {
    this.logger_.info("Creating review", { data })

    try {
      const review = await this.create(data)
      this.logger_.info("Review created successfully", { id: review.id })
      return review
    } catch (error) {
      this.logger_.error("Failed to create review", { error })
      throw error
    }
  }
}
```

### Method 4: Database Queries

```bash
# Watch database queries
psql medusa_db

# In another terminal
tail -f /var/log/postgresql/postgresql.log

# Or enable query logging in PostgreSQL config
ALTER DATABASE medusa_db SET log_statement = 'all';
```

### Method 5: Network Debugging

```bash
# Use curl with verbose output
curl -v http://localhost:9000/admin/reviews

# Or use httpie
http -v :9000/admin/reviews

# Or browser DevTools → Network tab
```

### ✅ Quick Check

```typescript
// Add comprehensive logging
this.logger_.debug("Step 1: Validating input")
this.logger_.debug("Step 2: Fetching from DB")
this.logger_.debug("Step 3: Transforming data")
this.logger_.info("Operation completed successfully")
```

---

## How to Write Tests

**Goal:** Write integration tests for Review module.

**Time:** 15-20 minutes

### Step 1: Create Test Directory

```bash
mkdir -p /home/user/medusa/packages/modules/review/integration-tests/__tests__
```

### Step 2: Create Test File

**File:** `integration-tests/__tests__/review.spec.ts`

```typescript
import { moduleIntegrationTestRunner } from "@medusajs/test-utils"

jest.setTimeout(30000)

moduleIntegrationTestRunner({
  moduleName: "review",
  testSuite: ({ service }) => {
    describe("Review Module", () => {
      it("should create a review", async () => {
        const review = await service.createReview({
          product_id: "prod_123",
          customer_id: "cust_456",
          rating: 5,
          title: "Great product!",
          content: "I love it!",
        })

        expect(review).toBeDefined()
        expect(review.id).toBeDefined()
        expect(review.rating).toBe(5)
        expect(review.title).toBe("Great product!")
      })

      it("should list reviews by product", async () => {
        // Create test data
        await service.createReview({
          product_id: "prod_123",
          customer_id: "cust_1",
          rating: 5,
          title: "Review 1",
        })

        await service.createReview({
          product_id: "prod_123",
          customer_id: "cust_2",
          rating: 4,
          title: "Review 2",
        })

        // Test
        const reviews = await service.listReviews({
          product_id: "prod_123",
        })

        expect(reviews).toHaveLength(2)
      })

      it("should calculate average rating", async () => {
        // Create test data
        await service.createReview({
          product_id: "prod_456",
          customer_id: "cust_1",
          rating: 5,
          title: "Perfect",
        })

        await service.createReview({
          product_id: "prod_456",
          customer_id: "cust_2",
          rating: 3,
          title: "Okay",
        })

        // Test
        const avgRating = await service.getAverageRating("prod_456")

        expect(avgRating).toBe(4) // (5 + 3) / 2
      })

      it("should update a review", async () => {
        const review = await service.createReview({
          product_id: "prod_789",
          customer_id: "cust_1",
          rating: 3,
          title: "Initial review",
        })

        const updated = await service.updateReview(review.id, {
          rating: 5,
          title: "Updated review",
        })

        expect(updated.rating).toBe(5)
        expect(updated.title).toBe("Updated review")
      })

      it("should delete a review", async () => {
        const review = await service.createReview({
          product_id: "prod_111",
          customer_id: "cust_1",
          rating: 5,
          title: "To be deleted",
        })

        await service.deleteReview(review.id)

        await expect(
          service.retrieveReview(review.id)
        ).rejects.toThrow()
      })
    })
  },
})
```

### Step 3: Run Tests

```bash
cd /home/user/medusa/packages/modules/review
yarn test
```

### ✅ Quick Check

```bash
# Run specific test
yarn test review.spec.ts

# Run with coverage
yarn test --coverage

# Watch mode
yarn test --watch
```

### ⚠️ Common Pitfalls

1. **Missing Test Timeout:** Set `jest.setTimeout(30000)`
2. **Database State:** Tests should be isolated (use transactions)
3. **Async/Await:** Always await async operations

---

## How to Use the CLI

**Goal:** Master common Medusa CLI commands.

### Development

```bash
# Start development server (with hot reload)
yarn dev

# Build all packages
yarn build

# Build specific package
cd packages/medusa
yarn build
```

### Database

```bash
# Run migrations
yarn medusa db:migrate

# Rollback last migration
yarn medusa db:migrate:rollback

# Generate migration
yarn medusa db:generate MODULE_NAME "MigrationName"

# Seed database
yarn medusa db:seed

# Reset database (drop & recreate)
yarn medusa db:reset
```

### User Management

```bash
# Create admin user
yarn medusa user -e admin@example.com -p supersecret

# Create user interactively
yarn medusa user
```

### Module Management

```bash
# List loaded modules
curl http://localhost:9000/admin/modules

# Check module status
yarn medusa modules list
```

### Testing

```bash
# Run all tests
yarn test

# Run integration tests
yarn test:integration

# Run specific test file
yarn test path/to/test.spec.ts

# Run with coverage
yarn test --coverage
```

### Build & Deployment

```bash
# Build production
yarn build

# Start production server
yarn start

# Check build output
ls -la dist/
```

### Useful Flags

```bash
# Verbose output
yarn dev --verbose

# Specific port
yarn dev --port 9001

# Skip migrations
yarn dev --skip-migrations
```

### ✅ Quick Reference

```bash
# Common workflow
yarn dev                    # Start development
yarn medusa db:migrate      # Run migrations
yarn medusa user           # Create admin
yarn test                  # Run tests
yarn build                 # Build for production
```

---

**Next Steps:**
- [CODE_TOURS.md](./CODE_TOURS.md) - Guided walkthroughs of key features
- [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Daily development workflow
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Code patterns reference
