# Hands-On Learning Exercises

**Progressive exercises to master Medusa development through practice**

**Last Updated:** November 19, 2025
**Version:** 2.11.3
**Total Time:** 3 weeks (1-2 hours per day)

---

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [Beginner Exercises (Week 1)](#beginner-exercises-week-1)
3. [Intermediate Exercises (Week 2)](#intermediate-exercises-week-2)
4. [Advanced Exercises (Week 3)](#advanced-exercises-week-3)
5. [Bonus Challenges](#bonus-challenges)

---

## How to Use This Guide

**Philosophy:** Learning by doing. Each exercise builds on previous ones.

**Format:**
- **Objective:** What you'll accomplish
- **Prerequisites:** What you need to know/complete first
- **Time Estimate:** How long it should take
- **Instructions:** Step-by-step guide
- **Verification:** How to confirm success
- **Solution/Hints:** Help when stuck
- **What You Learned:** Key takeaways
- **Next Steps:** Where to go from here

**Tips:**
- Don't skip exercises - each builds important skills
- Try solving before looking at hints
- Take notes on what you learn
- Ask for help in Discord if stuck
- Celebrate small wins

---

## Beginner Exercises (Week 1)

### Exercise 1: Explore the Admin Dashboard

**Objective:** Get comfortable with the Medusa admin interface and understand the product data model.

**Prerequisites:**
- Medusa installed and running locally
- See `/home/user/medusa/docs/learning/GETTING_STARTED.md`

**Time Estimate:** 30 minutes

**Instructions:**

1. **Start the admin dashboard:**
   ```bash
   cd /home/user/medusa
   yarn dev
   # Admin opens at http://localhost:9000/app
   ```

2. **Create your first product:**
   - Navigate to Products > New Product
   - Fill in the form:
     - Title: "Test T-Shirt"
     - Handle: "test-t-shirt"
     - Description: "A comfortable cotton t-shirt"
     - Price: $25.00
     - Add at least 2 variants (e.g., Small, Medium)
   - Click "Publish"

3. **Explore the product structure:**
   - Edit the product you created
   - Notice the tabs: General, Variants, Attributes, Media, etc.
   - Add an image to the product
   - Add inventory quantities to variants

4. **Create other entities:**
   - Create a customer (Customers > New)
   - Create a discount code (Promotions > New)
   - Browse orders (if any exist from seed data)

**Verification:**

```bash
# Check the product exists in the database
psql -d medusa -c "SELECT id, title, handle FROM product LIMIT 5;"
```

You should see your "Test T-Shirt" in the results.

**What You Learned:**
- How to navigate the admin dashboard
- Product data structure (products, variants, inventory)
- Relationship between products and other entities
- How data flows from UI to database

**Next Steps:**
- Explore the Settings page to understand configuration
- Try creating a full product catalog (5-10 products)
- Experiment with product collections and categories

---

### Exercise 2: Make Your First API Call

**Objective:** Learn to interact with Medusa's REST API using cURL and understand authentication.

**Prerequisites:**
- Exercise 1 completed
- Medusa server running
- cURL installed (`curl --version`)

**Time Estimate:** 20 minutes

**Instructions:**

1. **List products (public endpoint):**
   ```bash
   curl http://localhost:9000/store/products | jq
   ```

   You should see a JSON response with your products.

2. **Get admin API token:**
   ```bash
   # Login as admin (default: admin@medusa-test.com / supersecret)
   curl -X POST http://localhost:9000/auth/user/emailpass \
     -H "Content-Type: application/json" \
     -d '{
       "email": "admin@medusa-test.com",
       "password": "supersecret"
     }' | jq
   ```

   Save the token from the response.

3. **Use authenticated endpoint:**
   ```bash
   # Replace YOUR_TOKEN with the token from step 2
   export MEDUSA_TOKEN="YOUR_TOKEN"

   curl http://localhost:9000/admin/products \
     -H "Authorization: Bearer $MEDUSA_TOKEN" | jq
   ```

4. **Create a product via API:**
   ```bash
   curl -X POST http://localhost:9000/admin/products \
     -H "Authorization: Bearer $MEDUSA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "API Test Product",
       "handle": "api-test-product",
       "description": "Created via API",
       "status": "published"
     }' | jq
   ```

5. **Query specific product:**
   ```bash
   # Get product by ID (use ID from previous response)
   curl http://localhost:9000/store/products/PRODUCT_ID | jq
   ```

**Verification:**

Check the admin dashboard - you should see "API Test Product" in the products list.

**Hints:**
- If you get authentication errors, the token may have expired - generate a new one
- Use `jq` to format JSON output (install with `brew install jq` or `apt-get install jq`)
- Check `/home/user/medusa/docs/learning/API_DOCUMENTATION.md` for more endpoints

**What You Learned:**
- How to authenticate with Medusa API
- Difference between store and admin APIs
- Making GET and POST requests
- How the API returns data

**Next Steps:**
- Try UPDATE and DELETE operations
- Explore other endpoints (orders, customers, regions)
- Use Postman or Insomnia for easier API testing

---

### Exercise 3: Explore the Database Schema

**Objective:** Understand the database structure and learn to query Medusa's data directly.

**Prerequisites:**
- PostgreSQL installed and accessible
- Medusa database created
- Basic SQL knowledge

**Time Estimate:** 30 minutes

**Instructions:**

1. **Connect to the database:**
   ```bash
   psql -d medusa
   ```

2. **List all tables:**
   ```sql
   \dt
   ```

   You'll see 100+ tables. Key ones to explore:
   - `product` - Product information
   - `product_variant` - SKUs, pricing
   - `order` - Customer orders
   - `cart` - Shopping carts
   - `customer` - Customer data

3. **Explore product structure:**
   ```sql
   -- See product columns
   \d product

   -- Query products
   SELECT id, title, handle, status, created_at
   FROM product
   LIMIT 5;

   -- See product with variants
   SELECT
     p.title as product_name,
     pv.title as variant_name,
     pv.sku,
     pm.amount as price
   FROM product p
   JOIN product_variant pv ON p.id = pv.product_id
   JOIN price_set ps ON pv.price_set_id = ps.id
   JOIN price pm ON ps.id = pm.price_set_id
   LIMIT 10;
   ```

4. **Explore relationships:**
   ```sql
   -- Products with their categories
   SELECT
     p.title,
     pc.name as category
   FROM product p
   LEFT JOIN product_category_product pcp ON p.id = pcp.product_id
   LEFT JOIN product_category pc ON pcp.product_category_id = pc.id
   LIMIT 10;

   -- Orders with customer information
   SELECT
     o.id as order_id,
     o.email,
     o.total,
     o.status,
     o.created_at
   FROM order o
   ORDER BY o.created_at DESC
   LIMIT 5;
   ```

5. **Understand inventory:**
   ```sql
   -- Inventory levels for variants
   SELECT
     pv.title as variant,
     pv.sku,
     il.stocked_quantity,
     il.reserved_quantity,
     il.stocked_quantity - il.reserved_quantity as available
   FROM product_variant pv
   LEFT JOIN inventory_item ii ON pv.id = ii.id
   LEFT JOIN inventory_level il ON ii.id = il.inventory_item_id
   LIMIT 10;
   ```

6. **Create a useful view:**
   ```sql
   -- Create a view for easy product querying
   CREATE VIEW product_summary AS
   SELECT
     p.id,
     p.title,
     p.handle,
     p.status,
     COUNT(DISTINCT pv.id) as variant_count,
     p.created_at
   FROM product p
   LEFT JOIN product_variant pv ON p.id = pv.product_id
   GROUP BY p.id, p.title, p.handle, p.status, p.created_at;

   -- Use the view
   SELECT * FROM product_summary;
   ```

**Verification:**

Run this query - it should return data without errors:

```sql
SELECT
  COUNT(*) as total_products,
  COUNT(DISTINCT status) as status_types
FROM product;
```

**Hints:**
- Use `\?` for PostgreSQL help
- Use `\q` to exit psql
- See `/home/user/medusa/docs/learning/DATABASE_SCHEMA.md` for full schema documentation
- Medusa uses MikroORM - the schema is auto-generated from models

**What You Learned:**
- Medusa's database structure
- How products, variants, and pricing relate
- How to write queries for commerce data
- The link module pattern (join tables)

**Next Steps:**
- Explore other tables (cart, fulfillment, payment)
- Write more complex JOIN queries
- Learn about MikroORM and how models map to tables
- Try database migrations

---

## Intermediate Exercises (Week 2)

### Exercise 4: Create a Simple Module

**Objective:** Build a custom "Notes" module with full CRUD operations.

**Prerequisites:**
- Exercises 1-3 completed
- TypeScript basics
- Understanding of modules from `/home/user/medusa/docs/learning/BACKEND_ARCHITECTURE.md`

**Time Estimate:** 2 hours

**Instructions:**

1. **Create module structure:**
   ```bash
   cd /home/user/medusa/packages/modules
   mkdir -p note/src/{models,services,migrations}
   cd note
   ```

2. **Initialize package.json:**
   ```json
   {
     "name": "@medusajs/note",
     "version": "0.0.1",
     "main": "dist/index.js",
     "types": "dist/index.d.ts",
     "scripts": {
       "build": "tsc"
     },
     "dependencies": {
       "@medusajs/framework": "workspace:^",
       "@medusajs/utils": "workspace:^"
     },
     "devDependencies": {
       "typescript": "^5.6.2"
     }
   }
   ```

3. **Create the model:**

   File: `src/models/note.ts`
   ```typescript
   import { model } from "@medusajs/framework/utils"

   const Note = model.define("note", {
     id: model.id({ prefix: "note" }).primaryKey(),
     title: model.text(),
     content: model.text(),
     tags: model.array().nullable(),
     metadata: model.json().nullable(),
   })

   export default Note
   ```

4. **Create the service:**

   File: `src/services/note.ts`
   ```typescript
   import { MedusaService } from "@medusajs/framework/utils"
   import Note from "../models/note"

   export default class NoteService extends MedusaService({ Note }) {}
   ```

5. **Create module definition:**

   File: `src/index.ts`
   ```typescript
   import { Module } from "@medusajs/framework/utils"
   import NoteService from "./services/note"

   export const NOTE_MODULE = "note"

   export default Module(NOTE_MODULE, {
     service: NoteService,
   })
   ```

6. **Add TypeScript config:**

   File: `tsconfig.json`
   ```json
   {
     "extends": "../../tsconfig.base.json",
     "compilerOptions": {
       "outDir": "./dist"
     },
     "include": ["src"],
     "exclude": ["node_modules", "dist"]
   }
   ```

7. **Register the module:**

   Edit `/home/user/medusa/packages/medusa/src/medusa-config.ts`:
   ```typescript
   import { NOTE_MODULE } from "@medusajs/note"

   // In the modules section:
   modules: [
     // ... existing modules
     {
       resolve: "@medusajs/note",
       key: NOTE_MODULE,
     },
   ]
   ```

8. **Build and run:**
   ```bash
   # Build the module
   cd /home/user/medusa/packages/modules/note
   yarn build

   # Build and run Medusa
   cd /home/user/medusa
   yarn build
   yarn dev
   ```

**Verification:**

Check the module is loaded:

```bash
# Check logs for "note module loaded" or similar
# Check database - a "note" table should be created
psql -d medusa -c "\d note"
```

**Hints:**
- If the table doesn't appear, check migrations ran successfully
- Look at existing modules in `/home/user/medusa/packages/modules` for examples
- Check the console for error messages

**What You Learned:**
- Module structure and organization
- How to create models with the model builder
- How MedusaService provides CRUD operations
- Module registration and dependency injection

**Next Steps:**
- Add API endpoints to expose the module (Exercise 5)
- Add relationships to other modules
- Add custom methods to the service
- Write tests for the module

---

### Exercise 5: Add an API Endpoint

**Objective:** Create a GET /admin/stats endpoint that returns database statistics.

**Prerequisites:**
- Exercise 4 completed
- Understanding of API routes
- Express.js basics

**Time Estimate:** 1 hour

**Instructions:**

1. **Create route file:**

   File: `/home/user/medusa/packages/medusa/src/api/admin/stats/route.ts`
   ```typescript
   import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
   import { MedusaModule } from "@medusajs/framework/modules-sdk"

   export async function GET(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const productModule = req.scope.resolve("productModuleService")
     const orderModule = req.scope.resolve("orderModuleService")
     const customerModule = req.scope.resolve("customerModuleService")

     const [products, orders, customers] = await Promise.all([
       productModule.listProducts({}, { take: 1 }),
       orderModule.listOrders({}, { take: 1 }),
       customerModule.listCustomers({}, { take: 1 }),
     ])

     const stats = {
       products: products[1], // count
       orders: orders[1],
       customers: customers[1],
       timestamp: new Date().toISOString(),
     }

     res.json({ stats })
   }
   ```

2. **Add validation (optional but recommended):**

   File: `/home/user/medusa/packages/medusa/src/api/admin/stats/validators.ts`
   ```typescript
   import { z } from "zod"

   export const GetStatsQueryParams = z.object({
     detailed: z.boolean().optional().default(false),
   })

   export type GetStatsQueryParamsType = z.infer<typeof GetStatsQueryParams>
   ```

3. **Update route with validation:**
   ```typescript
   import { validateAndTransformQuery } from "@medusajs/framework"
   import { GetStatsQueryParams } from "./validators"

   export async function GET(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const validated = await validateAndTransformQuery(
       GetStatsQueryParams,
       req.query
     )

     // ... rest of the code
   }
   ```

4. **Test the endpoint:**
   ```bash
   # Restart the server
   cd /home/user/medusa
   yarn dev

   # Test the endpoint
   curl http://localhost:9000/admin/stats \
     -H "Authorization: Bearer YOUR_TOKEN" | jq
   ```

5. **Add to the Notes module:**

   Create a note endpoint:

   File: `/home/user/medusa/packages/medusa/src/api/admin/notes/route.ts`
   ```typescript
   import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

   export async function GET(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const noteService = req.scope.resolve("noteModuleService")

     const [notes, count] = await noteService.listNotes()

     res.json({ notes, count })
   }

   export async function POST(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const noteService = req.scope.resolve("noteModuleService")

     const note = await noteService.createNotes(req.body)

     res.json({ note })
   }
   ```

6. **Create note via API:**
   ```bash
   curl -X POST http://localhost:9000/admin/notes \
     -H "Authorization: Bearer $MEDUSA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "My First Note",
       "content": "Created via API",
       "tags": ["test", "api"]
     }' | jq
   ```

**Verification:**

```bash
# List notes
curl http://localhost:9000/admin/notes \
  -H "Authorization: Bearer $MEDUSA_TOKEN" | jq

# Check database
psql -d medusa -c "SELECT * FROM note;"
```

**What You Learned:**
- How to create API endpoints
- File-based routing in Medusa
- Dependency injection with req.scope.resolve
- Input validation with Zod
- Async/await patterns for database queries

**Next Steps:**
- Add UPDATE and DELETE endpoints
- Add pagination and filtering
- Add more complex validation rules
- Add error handling

---

### Exercise 6: Create a Workflow

**Objective:** Build a product approval workflow with steps and compensation logic.

**Prerequisites:**
- Exercises 4-5 completed
- Understanding of workflows from `/home/user/medusa/docs/learning/BACKEND_ARCHITECTURE.md`

**Time Estimate:** 1.5 hours

**Instructions:**

1. **Create workflow directory:**
   ```bash
   mkdir -p /home/user/medusa/packages/core/workflows-sdk/src/approval
   cd /home/user/medusa/packages/core/workflows-sdk/src/approval
   ```

2. **Create workflow steps:**

   File: `steps/validate-product.ts`
   ```typescript
   import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

   export const validateProductStep = createStep(
     "validate-product",
     async ({ productId }: { productId: string }, { container }) => {
       const productService = container.resolve("productModuleService")

       const product = await productService.retrieveProduct(productId)

       if (!product.title || product.title.length < 3) {
         throw new Error("Product title must be at least 3 characters")
       }

       if (!product.description) {
         throw new Error("Product description is required")
       }

       return new StepResponse({ valid: true })
     }
   )
   ```

   File: `steps/approve-product.ts`
   ```typescript
   import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

   export const approveProductStep = createStep(
     "approve-product",
     async ({ productId }: { productId: string }, { container }) => {
       const productService = container.resolve("productModuleService")

       // Update product status to "published"
       const updatedProduct = await productService.updateProducts(productId, {
         status: "published",
       })

       return new StepResponse(
         { approved: true, productId },
         { previousStatus: "draft" } // For compensation
       )
     },
     async ({ previousStatus }, { container }) => {
       // Compensation: revert to previous status
       const productService = container.resolve("productModuleService")

       await productService.updateProducts(productId, {
         status: previousStatus,
       })
     }
   )
   ```

   File: `steps/notify-approval.ts`
   ```typescript
   import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

   export const notifyApprovalStep = createStep(
     "notify-approval",
     async ({ productId, email }: { productId: string, email: string }) => {
       console.log(`Product ${productId} approved! Notification sent to ${email}`)

       // In real implementation, send email via notification module
       // const notificationService = container.resolve("notificationModuleService")
       // await notificationService.send({ to: email, template: "product-approved" })

       return new StepResponse({ notified: true })
     }
   )
   ```

3. **Create the workflow:**

   File: `approve-product-workflow.ts`
   ```typescript
   import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"
   import { validateProductStep } from "./steps/validate-product"
   import { approveProductStep } from "./steps/approve-product"
   import { notifyApprovalStep } from "./steps/notify-approval"

   type ApproveProductInput = {
     productId: string
     email: string
   }

   export const approveProductWorkflow = createWorkflow(
     "approve-product",
     (input: ApproveProductInput) => {
       // Step 1: Validate product
       const validated = validateProductStep({ productId: input.productId })

       // Step 2: Approve product
       const approved = approveProductStep({ productId: input.productId })

       // Step 3: Send notification
       const notified = notifyApprovalStep({
         productId: input.productId,
         email: input.email,
       })

       return new WorkflowResponse({
         approved: approved.approved,
         notified: notified.notified,
       })
     }
   )
   ```

4. **Export the workflow:**

   File: `index.ts`
   ```typescript
   export * from "./approve-product-workflow"
   ```

5. **Use the workflow in an API endpoint:**

   File: `/home/user/medusa/packages/medusa/src/api/admin/products/[id]/approve/route.ts`
   ```typescript
   import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
   import { approveProductWorkflow } from "@medusajs/workflows/approval"

   export async function POST(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const { id } = req.params
     const { email } = req.body

     const { result } = await approveProductWorkflow(req.scope).run({
       input: {
         productId: id,
         email: email || "admin@example.com",
       },
     })

     res.json({ result })
   }
   ```

6. **Test the workflow:**
   ```bash
   # Create a draft product first
   PRODUCT_ID=$(curl -X POST http://localhost:9000/admin/products \
     -H "Authorization: Bearer $MEDUSA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "Workflow Test Product",
       "description": "Testing the approval workflow",
       "status": "draft"
     }' | jq -r '.product.id')

   # Approve the product
   curl -X POST http://localhost:9000/admin/products/$PRODUCT_ID/approve \
     -H "Authorization: Bearer $MEDUSA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"email": "test@example.com"}' | jq

   # Verify status changed
   curl http://localhost:9000/admin/products/$PRODUCT_ID \
     -H "Authorization: Bearer $MEDUSA_TOKEN" | jq '.product.status'
   ```

**Verification:**

The product status should be "published" and you should see the notification log in the console.

**What You Learned:**
- How to create workflow steps
- Compensation logic for rollbacks
- Chaining steps together
- Using workflows in API endpoints
- Error handling in workflows

**Next Steps:**
- Add more complex validation logic
- Implement parallel steps
- Add conditional logic
- Use hooks for side effects

---

### Exercise 7: Add a Database Field and Migration

**Objective:** Add a SKU prefix field to products with a database migration.

**Prerequisites:**
- Exercise 4 completed
- Understanding of MikroORM
- PostgreSQL knowledge

**Time Estimate:** 1 hour

**Instructions:**

1. **Update the product model:**

   Find the product model (typically in `@medusajs/product`):

   File: `/home/user/medusa/packages/modules/product/src/models/product.ts`

   Add the new field:
   ```typescript
   // Add to the Product model definition
   sku_prefix: model.text().nullable().default("PROD"),
   ```

2. **Generate migration:**
   ```bash
   cd /home/user/medusa

   # This will detect the model change and create a migration
   yarn medusa db:generate product
   ```

3. **Review the migration:**

   Look in `/home/user/medusa/packages/modules/product/src/migrations/` for the new migration file. It should look like:

   ```typescript
   import { Migration } from '@mikro-orm/migrations'

   export class Migration20251119000000 extends Migration {
     async up(): Promise<void> {
       this.addSql('alter table "product" add column "sku_prefix" text null default \'PROD\';')
     }

     async down(): Promise<void> {
       this.addSql('alter table "product" drop column "sku_prefix";')
     }
   }
   ```

4. **Run the migration:**
   ```bash
   yarn medusa db:migrate product
   ```

5. **Verify the change:**
   ```sql
   psql -d medusa -c "\d product"
   # Should show sku_prefix column

   psql -d medusa -c "SELECT id, title, sku_prefix FROM product LIMIT 5;"
   ```

6. **Use the new field:**

   Update a product to set the prefix:
   ```bash
   curl -X POST http://localhost:9000/admin/products/PRODUCT_ID \
     -H "Authorization: Bearer $MEDUSA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"sku_prefix": "TSH"}' | jq
   ```

7. **Update service to use the prefix:**

   Create a method that generates SKUs with the prefix:

   ```typescript
   // In a custom service or workflow step
   async function generateSKU(productId: string, variantNumber: number) {
     const product = await productService.retrieveProduct(productId)
     const prefix = product.sku_prefix || "PROD"
     const sku = `${prefix}-${productId.slice(-6)}-${variantNumber}`
     return sku
   }
   ```

**Verification:**

```bash
# Check the database directly
psql -d medusa -c "
  SELECT
    p.title,
    p.sku_prefix,
    pv.sku
  FROM product p
  LEFT JOIN product_variant pv ON p.id = pv.product_id
  LIMIT 5;
"
```

**Hints:**
- Always review migrations before running them
- Test migrations in development first
- Keep migrations small and focused
- Add default values for existing rows

**What You Learned:**
- How to modify data models
- Migration generation with MikroORM
- Running and reverting migrations
- Best practices for schema changes

**Next Steps:**
- Add indexes for new fields
- Create more complex migrations (multiple tables)
- Learn about migration rollback
- Add validation for new fields

---

## Advanced Exercises (Week 3)

### Exercise 8: Build an Event Subscriber Integration

**Objective:** Create an event subscriber that sends notifications when orders are placed.

**Prerequisites:**
- All intermediate exercises completed
- Understanding of event bus
- Knowledge of subscribers

**Time Estimate:** 2 hours

**Instructions:**

1. **Create subscriber directory:**
   ```bash
   mkdir -p /home/user/medusa/packages/medusa/src/subscribers
   cd /home/user/medusa/packages/medusa/src/subscribers
   ```

2. **Create the subscriber:**

   File: `order-notification.ts`
   ```typescript
   import { SubscriberArgs, type SubscriberConfig } from "@medusajs/framework"

   export default async function orderPlacedHandler({
     event: { data },
     container,
   }: SubscriberArgs<{ id: string }>) {
     const orderService = container.resolve("orderModuleService")
     const logger = container.resolve("logger")

     const order = await orderService.retrieveOrder(data.id, {
       relations: ["items", "customer"],
     })

     logger.info(`Order placed: ${order.id}`)
     logger.info(`Customer: ${order.email}`)
     logger.info(`Total: ${order.total}`)
     logger.info(`Items: ${order.items.length}`)

     // In production, send actual notifications:
     // const notificationService = container.resolve("notificationModuleService")
     // await notificationService.send({
     //   to: order.email,
     //   template: "order-confirmation",
     //   data: { order }
     // })

     // Send to admin
     // await notificationService.send({
     //   to: "admin@store.com",
     //   template: "new-order-admin",
     //   data: { order }
     // })
   }

   export const config: SubscriberConfig = {
     event: "order.placed",
   }
   ```

3. **Create a more complex subscriber with multiple events:**

   File: `order-tracking.ts`
   ```typescript
   import { SubscriberArgs, type SubscriberConfig } from "@medusajs/framework"

   export default async function orderTrackingHandler({
     event: { name, data },
     container,
   }: SubscriberArgs<{ id: string }>) {
     const logger = container.resolve("logger")

     const eventMessages = {
       "order.placed": "Order has been placed",
       "order.fulfillment_created": "Order is being prepared",
       "order.shipment_created": "Order has been shipped",
       "order.completed": "Order has been delivered",
     }

     const message = eventMessages[name] || "Order status updated"
     logger.info(`[Order ${data.id}] ${message}`)

     // Track in database
     const noteService = container.resolve("noteModuleService")
     await noteService.createNotes({
       title: `Order Event: ${name}`,
       content: `Order ${data.id}: ${message}`,
       tags: ["order", "tracking"],
       metadata: { orderId: data.id, event: name },
     })
   }

   export const config: SubscriberConfig = {
     event: [
       "order.placed",
       "order.fulfillment_created",
       "order.shipment_created",
       "order.completed",
     ],
   }
   ```

4. **Create a custom event emitter:**

   File: `/home/user/medusa/packages/medusa/src/api/admin/products/[id]/feature/route.ts`
   ```typescript
   import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

   export async function POST(
     req: MedusaRequest,
     res: MedusaResponse
   ) {
     const { id } = req.params
     const eventBus = req.scope.resolve("eventBusModuleService")
     const productService = req.scope.resolve("productModuleService")

     const product = await productService.retrieveProduct(id)

     // Emit custom event
     await eventBus.emit("product.featured", {
       id: product.id,
       title: product.title,
       timestamp: new Date(),
     })

     res.json({ message: "Product featured event emitted" })
   }
   ```

5. **Create subscriber for custom event:**

   File: `product-featured.ts`
   ```typescript
   import { SubscriberArgs, type SubscriberConfig } from "@medusajs/framework"

   export default async function productFeaturedHandler({
     event: { data },
     container,
   }: SubscriberArgs<{ id: string; title: string }>) {
     const logger = container.resolve("logger")

     logger.info(`Product featured: ${data.title} (${data.id})`)

     // Update homepage, send to marketing team, etc.
   }

   export const config: SubscriberConfig = {
     event: "product.featured",
   }
   ```

6. **Test the subscribers:**
   ```bash
   # Restart server to load subscribers
   yarn dev

   # Create an order (or use store API)
   # Check logs for subscriber output

   # Trigger custom event
   curl -X POST http://localhost:9000/admin/products/PRODUCT_ID/feature \
     -H "Authorization: Bearer $MEDUSA_TOKEN" | jq

   # Check logs for "Product featured" message
   # Check notes table for tracking entries
   psql -d medusa -c "SELECT * FROM note WHERE tags @> '{\"order\"}' ORDER BY created_at DESC LIMIT 5;"
   ```

**Verification:**

Check the server logs - you should see the subscriber messages when events are emitted.

**What You Learned:**
- How to create event subscribers
- Subscribing to multiple events
- Emitting custom events
- Using the event bus for decoupled architecture
- Async event handling

**Next Steps:**
- Add error handling and retries
- Create subscribers for batch processing
- Implement rate limiting for events
- Add event monitoring and logging

---

### Exercise 9: Add an Admin Dashboard Page

**Objective:** Create a custom analytics dashboard page in the admin UI.

**Prerequisites:**
- React knowledge
- Understanding of admin UI structure
- TanStack Query experience helpful

**Time Estimate:** 2 hours

**Instructions:**

1. **Create the dashboard page:**

   File: `/home/user/medusa/packages/admin/dashboard/src/routes/analytics/page.tsx`
   ```tsx
   import { Container, Heading } from "@medusajs/ui"
   import { useEffect, useState } from "react"

   type Stats = {
     products: number
     orders: number
     customers: number
     timestamp: string
   }

   export default function AnalyticsPage() {
     const [stats, setStats] = useState<Stats | null>(null)
     const [loading, setLoading] = useState(true)

     useEffect(() => {
       fetch("/admin/stats", {
         credentials: "include",
       })
         .then((res) => res.json())
         .then((data) => {
           setStats(data.stats)
           setLoading(false)
         })
         .catch((err) => {
           console.error("Failed to fetch stats:", err)
           setLoading(false)
         })
     }, [])

     if (loading) {
       return <div>Loading...</div>
     }

     return (
       <Container>
         <Heading level="h1">Analytics Dashboard</Heading>

         <div className="grid grid-cols-3 gap-4 mt-6">
           <StatCard
             title="Products"
             value={stats?.products || 0}
             color="blue"
           />
           <StatCard
             title="Orders"
             value={stats?.orders || 0}
             color="green"
           />
           <StatCard
             title="Customers"
             value={stats?.customers || 0}
             color="purple"
           />
         </div>

         <div className="mt-6 text-sm text-gray-500">
           Last updated: {stats?.timestamp}
         </div>
       </Container>
     )
   }

   function StatCard({ title, value, color }: {
     title: string
     value: number
     color: string
   }) {
     const colorClasses = {
       blue: "bg-blue-100 text-blue-800",
       green: "bg-green-100 text-green-800",
       purple: "bg-purple-100 text-purple-800",
     }

     return (
       <div className={`p-6 rounded-lg ${colorClasses[color]}`}>
         <div className="text-sm font-medium">{title}</div>
         <div className="text-3xl font-bold mt-2">{value}</div>
       </div>
     )
   }
   ```

2. **Add navigation item:**

   File: `/home/user/medusa/packages/admin/dashboard/src/routes/analytics/analytics-nav.tsx`
   ```tsx
   import { ChartBar } from "@medusajs/icons"
   import { defineRouteConfig } from "@medusajs/admin-sdk"

   export default defineRouteConfig({
     label: "Analytics",
     icon: ChartBar,
   })
   ```

3. **Create a chart component:**

   Install chart library:
   ```bash
   cd /home/user/medusa/packages/admin/dashboard
   yarn add recharts
   ```

   Update the page with charts:
   ```tsx
   import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts'

   // Add to component
   const [chartData, setChartData] = useState([])

   // Fetch chart data
   useEffect(() => {
     // Simulate daily orders data
     const data = Array.from({ length: 7 }, (_, i) => ({
       day: `Day ${i + 1}`,
       orders: Math.floor(Math.random() * 50) + 10,
     }))
     setChartData(data)
   }, [])

   // Add to JSX
   <div className="mt-8">
     <Heading level="h2">Orders This Week</Heading>
     <ResponsiveContainer width="100%" height={300}>
       <LineChart data={chartData}>
         <CartesianGrid strokeDasharray="3 3" />
         <XAxis dataKey="day" />
         <YAxis />
         <Tooltip />
         <Line type="monotone" dataKey="orders" stroke="#8884d8" />
       </LineChart>
     </ResponsiveContainer>
   </div>
   ```

4. **Add a data table:**
   ```tsx
   import { Table } from "@medusajs/ui"

   function RecentOrders() {
     const [orders, setOrders] = useState([])

     useEffect(() => {
       fetch("/admin/orders?limit=5", {
         credentials: "include",
       })
         .then((res) => res.json())
         .then((data) => setOrders(data.orders))
     }, [])

     return (
       <div className="mt-8">
         <Heading level="h2">Recent Orders</Heading>
         <Table>
           <Table.Header>
             <Table.Row>
               <Table.HeaderCell>Order ID</Table.HeaderCell>
               <Table.HeaderCell>Customer</Table.HeaderCell>
               <Table.HeaderCell>Total</Table.HeaderCell>
               <Table.HeaderCell>Status</Table.HeaderCell>
             </Table.Row>
           </Table.Header>
           <Table.Body>
             {orders.map((order) => (
               <Table.Row key={order.id}>
                 <Table.Cell>{order.display_id}</Table.Cell>
                 <Table.Cell>{order.email}</Table.Cell>
                 <Table.Cell>${(order.total / 100).toFixed(2)}</Table.Cell>
                 <Table.Cell>{order.status}</Table.Cell>
               </Table.Row>
             ))}
           </Table.Body>
         </Table>
       </div>
     )
   }
   ```

5. **Build and test:**
   ```bash
   cd /home/user/medusa
   yarn build
   yarn dev

   # Navigate to http://localhost:9000/app/analytics
   ```

**Verification:**

You should see the Analytics page in the admin sidebar with stats, charts, and tables.

**What You Learned:**
- Admin UI routing and page creation
- Using Medusa UI components
- Fetching data from API endpoints
- Building data visualizations
- Navigation configuration

**Next Steps:**
- Add real-time updates with WebSockets
- Create more complex charts
- Add filters and date ranges
- Implement data export functionality

---

### Exercise 10: Full Feature Implementation - Product Reviews

**Objective:** Build a complete "Product Reviews" feature from scratch including module, API, workflows, and UI.

**Prerequisites:**
- All previous exercises completed
- Full understanding of Medusa architecture

**Time Estimate:** 4-6 hours

**Instructions:**

This is a comprehensive exercise. You'll build:
1. Review module (data models, services)
2. API endpoints (CRUD operations)
3. Workflows (review approval, spam detection)
4. Admin UI (review management page)
5. Store API (submit reviews, display reviews)

**Part 1: Create the Module (1 hour)**

1. Create module structure:
   ```bash
   cd /home/user/medusa/packages/modules
   mkdir -p review/src/{models,services,migrations}
   ```

2. Create models:

   File: `review/src/models/review.ts`
   ```typescript
   import { model } from "@medusajs/framework/utils"

   const Review = model.define("review", {
     id: model.id({ prefix: "rev" }).primaryKey(),
     product_id: model.text(),
     customer_id: model.text().nullable(),
     rating: model.number(), // 1-5
     title: model.text(),
     content: model.text(),
     verified_purchase: model.boolean().default(false),
     status: model.enum(["pending", "approved", "rejected"]).default("pending"),
     helpful_count: model.number().default(0),
     metadata: model.json().nullable(),
   })

   export default Review
   ```

3. Create service with custom methods:

   File: `review/src/services/review.ts`
   ```typescript
   import { MedusaService } from "@medusajs/framework/utils"
   import Review from "../models/review"

   export default class ReviewService extends MedusaService({ Review }) {
     async getProductReviews(productId: string) {
       return await this.listReviews({
         product_id: productId,
         status: "approved",
       })
     }

     async getAverageRating(productId: string) {
       const [reviews] = await this.listReviews({
         product_id: productId,
         status: "approved",
       })

       if (reviews.length === 0) return 0

       const sum = reviews.reduce((acc, review) => acc + review.rating, 0)
       return sum / reviews.length
     }
   }
   ```

**Part 2: Create API Endpoints (1 hour)**

File: `/home/user/medusa/packages/medusa/src/api/store/products/[id]/reviews/route.ts`
```typescript
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const { id } = req.params
  const reviewService = req.scope.resolve("reviewModuleService")

  const [reviews, count] = await reviewService.getProductReviews(id)
  const averageRating = await reviewService.getAverageRating(id)

  res.json({ reviews, count, averageRating })
}

export async function POST(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const { id } = req.params
  const { rating, title, content } = req.body
  const customerId = req.auth?.actor_id

  const reviewService = req.scope.resolve("reviewModuleService")

  const review = await reviewService.createReviews({
    product_id: id,
    customer_id: customerId,
    rating,
    title,
    content,
    status: "pending", // Will be approved via workflow
  })

  res.json({ review })
}
```

**Part 3: Create Workflows (1.5 hours)**

Create review approval workflow with spam detection:

```typescript
import { createWorkflow, createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

const detectSpamStep = createStep(
  "detect-spam",
  async ({ reviewId }) => {
    // Simple spam detection (in production, use ML/external service)
    const spamWords = ["spam", "fake", "scam"]
    const review = await reviewService.retrieveReview(reviewId)

    const isSpam = spamWords.some(word =>
      review.content.toLowerCase().includes(word)
    )

    return new StepResponse({ isSpam })
  }
)

const approveReviewStep = createStep(
  "approve-review",
  async ({ reviewId }, { container }) => {
    const reviewService = container.resolve("reviewModuleService")

    await reviewService.updateReviews(reviewId, {
      status: "approved",
    })

    return new StepResponse({ approved: true })
  }
)

export const reviewApprovalWorkflow = createWorkflow(
  "review-approval",
  ({ reviewId }) => {
    const spamCheck = detectSpamStep({ reviewId })

    // Only approve if not spam
    const approved = when({ spamCheck }, ({ spamCheck }) => {
      return !spamCheck.isSpam
    }).then(() => {
      return approveReviewStep({ reviewId })
    })

    return new WorkflowResponse({ approved })
  }
)
```

**Part 4: Admin UI (1.5 hours)**

File: `/home/user/medusa/packages/admin/dashboard/src/routes/reviews/page.tsx`

```tsx
import { Container, Heading, Table, Badge, Button } from "@medusajs/ui"
import { useEffect, useState } from "react"

export default function ReviewsPage() {
  const [reviews, setReviews] = useState([])

  useEffect(() => {
    fetch("/admin/reviews", { credentials: "include" })
      .then(res => res.json())
      .then(data => setReviews(data.reviews))
  }, [])

  const approveReview = async (id: string) => {
    await fetch(`/admin/reviews/${id}/approve`, {
      method: "POST",
      credentials: "include",
    })
    // Refresh list
  }

  return (
    <Container>
      <Heading level="h1">Product Reviews</Heading>

      <Table>
        <Table.Header>
          <Table.Row>
            <Table.HeaderCell>Product</Table.HeaderCell>
            <Table.HeaderCell>Rating</Table.HeaderCell>
            <Table.HeaderCell>Title</Table.HeaderCell>
            <Table.HeaderCell>Status</Table.HeaderCell>
            <Table.HeaderCell>Actions</Table.HeaderCell>
          </Table.Row>
        </Table.Header>
        <Table.Body>
          {reviews.map((review) => (
            <Table.Row key={review.id}>
              <Table.Cell>{review.product_id}</Table.Cell>
              <Table.Cell>{"⭐".repeat(review.rating)}</Table.Cell>
              <Table.Cell>{review.title}</Table.Cell>
              <Table.Cell>
                <Badge color={review.status === "approved" ? "green" : "yellow"}>
                  {review.status}
                </Badge>
              </Table.Cell>
              <Table.Cell>
                {review.status === "pending" && (
                  <Button onClick={() => approveReview(review.id)}>
                    Approve
                  </Button>
                )}
              </Table.Cell>
            </Table.Row>
          ))}
        </Table.Body>
      </Table>
    </Container>
  )
}
```

**Part 5: Integration & Testing (1 hour)**

1. Register module
2. Run migrations
3. Test all APIs
4. Test workflows
5. Test UI

**Verification:**

Complete end-to-end test:
```bash
# 1. Submit review via store API
curl -X POST http://localhost:9000/store/products/PRODUCT_ID/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "rating": 5,
    "title": "Great product!",
    "content": "Really happy with this purchase"
  }'

# 2. Check admin UI - should see pending review

# 3. Approve via admin

# 4. Check store API - should see approved review
curl http://localhost:9000/store/products/PRODUCT_ID/reviews
```

**What You Learned:**
- Full-stack feature development
- Module creation and architecture
- API design and implementation
- Workflow orchestration
- Admin UI development
- Integration and testing

**Congratulations!** You've completed the advanced exercises and built a production-ready feature.

---

## Bonus Challenges

Once you've completed all exercises, try these additional challenges:

1. **Add Search Functionality**
   - Implement full-text search for products
   - Add search API endpoint
   - Create search UI in admin

2. **Build a Loyalty Program**
   - Module for points and rewards
   - Workflow for earning/redeeming points
   - Integration with orders

3. **Create a Custom Dashboard**
   - Real-time sales dashboard
   - WebSocket integration
   - Advanced charts and analytics

4. **Implement Multi-Currency**
   - Currency conversion module
   - Price display in multiple currencies
   - Currency selector in admin

5. **Build a Mobile App**
   - React Native storefront
   - Use Medusa Store API
   - Cart and checkout flow

---

## Getting Help

- **Discord:** Join the Medusa Discord community
- **Documentation:** `/home/user/medusa/docs/learning/`
- **GitHub Issues:** Report bugs or ask questions
- **Stack Overflow:** Tag questions with `medusajs`

---

## Next Steps After Exercises

1. **Read the full documentation** in `/home/user/medusa/docs/learning/`
2. **Contribute to Medusa** - See `FIRST_CONTRIBUTIONS.md`
3. **Build a real project** - Start your own e-commerce store
4. **Join the community** - Share what you've built

---

**Happy Learning!** Remember: The best way to learn is by building. Don't be afraid to experiment and make mistakes.
