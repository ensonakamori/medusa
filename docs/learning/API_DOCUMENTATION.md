# API Documentation

Complete reference for Medusa's API structure, conventions, and usage patterns.

## Table of Contents

- [API Overview](#api-overview)
- [API Structure](#api-structure)
- [REST Conventions](#rest-conventions)
- [Authentication](#authentication)
- [Request & Response Format](#request--response-format)
- [Pagination](#pagination)
- [Filtering & Sorting](#filtering--sorting)
- [Error Responses](#error-responses)
- [Rate Limiting](#rate-limiting)
- [Versioning Strategy](#versioning-strategy)
- [API Examples](#api-examples)
- [Testing APIs](#testing-apis)
- [SDK Usage](#sdk-usage)

---

## API Overview

### API Architecture

Medusa provides RESTful APIs organized into three main categories:

```mermaid
graph TB
    A[Medusa API] --> B[Admin API]
    A --> C[Store API]
    A --> D[Auth API]

    B --> E[Products]
    B --> F[Orders]
    B --> G[Customers]
    B --> H[Inventory]

    C --> I[Products]
    C --> J[Cart]
    C --> K[Checkout]
    C --> L[Customers]

    D --> M[Email/Password]
    D --> N[OAuth]
    D --> O[Session]
```

### Base URLs

```
Admin API:  http://localhost:9000/admin/*
Store API:  http://localhost:9000/store/*
Auth API:   http://localhost:9000/auth/*
```

### Key Principles

✅ **RESTful**: Standard HTTP methods (GET, POST, PUT, DELETE, PATCH)
✅ **JSON**: All requests and responses use JSON
✅ **Stateless**: Each request contains all necessary information
✅ **Authenticated**: Admin routes require authentication
✅ **Versioned**: API versions for backward compatibility

---

## API Structure

### Directory Organization

```
packages/medusa/src/api/
├── admin/                 # Admin API routes
│   ├── products/
│   │   ├── route.ts      # GET /admin/products, POST /admin/products
│   │   ├── [id]/
│   │   │   └── route.ts  # GET /admin/products/:id, PUT, DELETE
│   │   ├── middlewares.ts
│   │   └── validators.ts
│   ├── orders/
│   ├── customers/
│   └── ...
├── store/                 # Store API routes
│   ├── products/
│   │   └── route.ts      # GET /store/products
│   ├── carts/
│   │   ├── route.ts      # POST /store/carts
│   │   └── [id]/
│   │       ├── route.ts  # GET /store/carts/:id
│   │       └── complete/
│   │           └── route.ts  # POST /store/carts/:id/complete
│   └── ...
└── auth/
    └── [actor_type]/
        └── [auth_provider]/
            └── route.ts  # POST /auth/customer/emailpass
```

### Route Structure

Each route file exports HTTP method handlers:

```typescript
// route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import { authenticate } from "@medusajs/framework/http"

// GET handler
export const GET = authenticate("admin")(
  async (
    req: MedusaRequest,
    res: MedusaResponse
  ) => {
    // Handle GET request
    const data = await service.list(req.filterableFields, req.queryConfig)
    res.json(data)
  }
)

// POST handler
export const POST = authenticate("admin")(
  async (req: MedusaRequest, res: MedusaResponse) => {
    // Handle POST request
    const created = await service.create(req.validatedBody)
    res.json(created)
  }
)
```

---

## REST Conventions

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource(s) | ✅ | ✅ |
| POST | Create new resource | ❌ | ❌ |
| PUT | Replace entire resource | ✅ | ❌ |
| PATCH | Partial update | ❌ | ❌ |
| DELETE | Remove resource | ✅ | ❌ |

### URL Patterns

```
# Collection operations
GET    /admin/products          # List products
POST   /admin/products          # Create product

# Resource operations
GET    /admin/products/:id      # Get single product
PUT    /admin/products/:id      # Update product (full)
PATCH  /admin/products/:id      # Update product (partial)
DELETE /admin/products/:id      # Delete product

# Sub-resource operations
GET    /admin/products/:id/variants       # List product variants
POST   /admin/products/:id/variants       # Create variant
DELETE /admin/products/:id/variants/:vid  # Delete variant

# Actions
POST   /store/carts/:id/complete          # Complete cart (create order)
POST   /admin/orders/:id/cancel           # Cancel order
POST   /admin/products/:id/variants/batch # Batch operations
```

### Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH, DELETE |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE with no response body |
| 400 | Bad Request | Validation error, invalid input |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Valid auth but insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource conflict (duplicate, constraint) |
| 422 | Unprocessable Entity | Semantic errors in request |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |

---

## Authentication

### Authentication Methods

#### 1. Bearer Token (JWT)

```bash
# Login to get token
curl -X POST http://localhost:9000/auth/user/emailpass \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@medusa.com",
    "password": "supersecret"
  }'

# Response
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

# Use token in subsequent requests
curl -X GET http://localhost:9000/admin/products \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### 2. Session Cookie

```bash
# Login with session
curl -X POST http://localhost:9000/auth/user/emailpass \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{
    "email": "admin@medusa.com",
    "password": "supersecret"
  }'

# Use session cookie
curl -X GET http://localhost:9000/admin/products \
  -b cookies.txt
```

#### 3. API Key

```bash
# Use API key
curl -X GET http://localhost:9000/admin/products \
  -H "x-api-key: your-api-key-here"
```

### Authentication Middleware

```typescript
import { authenticate } from "@medusajs/framework/http"

// Admin authentication
export const GET = authenticate("admin")(
  async (req, res) => {
    // req.auth_context contains authenticated user
    const userId = req.auth_context.actor_id
    // ...
  }
)

// Store (customer) authentication
export const GET = authenticate("store")(
  async (req, res) => {
    // req.auth_context contains authenticated customer
    const customerId = req.auth_context.actor_id
    // ...
  }
)

// Optional authentication
export const GET = authenticate("store", { allowUnregistered: true })(
  async (req, res) => {
    // Works for both authenticated and guest users
    const customerId = req.auth_context?.actor_id
    // ...
  }
)
```

### Auth Endpoints

```typescript
// POST /auth/:actor_type/:provider
// actor_type: "user" | "customer"
// provider: "emailpass" | "google" | "github"

// Email/Password login
POST /auth/user/emailpass
{
  "email": "admin@medusa.com",
  "password": "supersecret"
}

// Customer login
POST /auth/customer/emailpass
{
  "email": "customer@example.com",
  "password": "password123"
}

// OAuth callback
GET /auth/user/google/callback?code=...

// Logout
POST /auth/logout
```

---

## Request & Response Format

### Request Format

```typescript
// POST /admin/products
{
  "title": "Medusa T-Shirt",
  "handle": "medusa-tshirt",
  "description": "Comfortable cotton t-shirt",
  "status": "published",
  "variants": [
    {
      "title": "Small",
      "sku": "TSHIRT-S",
      "prices": [
        {
          "amount": 2999,
          "currency_code": "usd"
        }
      ]
    }
  ],
  "metadata": {
    "custom_field": "value"
  }
}
```

### Response Format

```typescript
// Success response
{
  "product": {
    "id": "prod_01234567890",
    "title": "Medusa T-Shirt",
    "handle": "medusa-tshirt",
    "status": "published",
    "created_at": "2024-11-19T10:00:00.000Z",
    "updated_at": "2024-11-19T10:00:00.000Z",
    "variants": [
      {
        "id": "variant_01234567890",
        "title": "Small",
        "sku": "TSHIRT-S",
        "prices": [...]
      }
    ]
  }
}
```

### Field Selection

Request specific fields to reduce payload size:

```bash
# Select specific fields
GET /admin/products?fields=id,title,handle

# Select nested fields
GET /admin/products?fields=id,title,variants.id,variants.sku

# Response
{
  "products": [
    {
      "id": "prod_123",
      "title": "T-Shirt",
      "handle": "tshirt",
      "variants": [
        {
          "id": "variant_123",
          "sku": "TSHIRT-S"
        }
      ]
    }
  ]
}
```

---

## Pagination

### Offset-Based Pagination

```bash
# Default pagination
GET /admin/products?limit=20&offset=0

# Page 2
GET /admin/products?limit=20&offset=20

# Response
{
  "products": [...],
  "count": 150,      # Total count
  "offset": 20,      # Current offset
  "limit": 20        # Items per page
}
```

### Cursor-Based Pagination

For better performance on large datasets:

```bash
# First page
GET /admin/products?limit=20

# Next page (using cursor from previous response)
GET /admin/products?limit=20&cursor=eyJpZCI6InByb2RfMTIzIn0=

# Response
{
  "products": [...],
  "has_more": true,
  "next_cursor": "eyJpZCI6InByb2RfNDU2In0="
}
```

### Default Limits

```typescript
// Default pagination values
const DEFAULT_LIMIT = 50
const MAX_LIMIT = 100

// Validation
const limit = Math.min(req.query.limit || DEFAULT_LIMIT, MAX_LIMIT)
const offset = req.query.offset || 0
```

---

## Filtering & Sorting

### Basic Filtering

```bash
# Filter by status
GET /admin/products?status=published

# Filter by multiple values
GET /admin/products?status[]=published&status[]=draft

# Filter by ID
GET /admin/products?id=prod_123

# Multiple IDs
GET /admin/products?id[]=prod_123&id[]=prod_456
```

### Advanced Filtering

```bash
# Date range filtering
GET /admin/orders?created_at[gte]=2024-01-01&created_at[lte]=2024-12-31

# Numeric comparisons
GET /admin/products?price[gt]=1000&price[lte]=5000

# String matching
GET /admin/products?title[$ilike]=%shirt%

# Search query
GET /admin/products?q=medusa
```

### Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `$eq` | Equals | `?status[$eq]=published` |
| `$ne` | Not equals | `?status[$ne]=draft` |
| `$gt` | Greater than | `?price[$gt]=1000` |
| `$gte` | Greater than or equal | `?created_at[$gte]=2024-01-01` |
| `$lt` | Less than | `?price[$lt]=5000` |
| `$lte` | Less than or equal | `?created_at[$lte]=2024-12-31` |
| `$like` | Pattern match (case-sensitive) | `?title[$like]=%Shirt%` |
| `$ilike` | Pattern match (case-insensitive) | `?title[$ilike]=%shirt%` |
| `$in` | In array | `?status[$in][]=published&status[$in][]=draft` |
| `$nin` | Not in array | `?status[$nin][]=rejected` |

### Sorting

```bash
# Sort ascending
GET /admin/products?order=created_at

# Sort descending
GET /admin/products?order=-created_at

# Multiple sort fields
GET /admin/products?order=status,-created_at

# Sort by nested field
GET /admin/products?order=variants.price
```

### Complex Queries

```bash
# Combine filters, sorting, and pagination
GET /admin/products?
  status=published&
  created_at[gte]=2024-01-01&
  price[gte]=1000&
  price[lte]=5000&
  order=-created_at&
  limit=20&
  offset=0
```

---

## Error Responses

### Error Format

```typescript
// Standard error response
{
  "type": "invalid_data",
  "message": "Product validation failed",
  "errors": [
    {
      "field": "title",
      "message": "Title is required"
    },
    {
      "field": "variants",
      "message": "At least one variant is required"
    }
  ]
}
```

### Error Types

```typescript
// MedusaError types
enum ErrorTypes {
  // 400 Bad Request
  INVALID_DATA = "invalid_data",
  INVALID_STATE = "invalid_state",

  // 401 Unauthorized
  UNAUTHORIZED = "unauthorized",

  // 403 Forbidden
  NOT_ALLOWED = "not_allowed",

  // 404 Not Found
  NOT_FOUND = "not_found",

  // 409 Conflict
  DUPLICATE_ERROR = "duplicate_error",

  // 422 Unprocessable Entity
  PAYMENT_AUTHORIZATION_ERROR = "payment_authorization_error",

  // 500 Internal Server Error
  UNEXPECTED_STATE = "unexpected_state",
  DATABASE_ERROR = "database_error",
}
```

### Example Error Responses

```typescript
// 400 Bad Request - Validation error
{
  "type": "invalid_data",
  "message": "Invalid product data",
  "errors": [
    {
      "field": "title",
      "message": "String must contain at least 1 character(s)",
      "code": "too_small"
    }
  ]
}

// 401 Unauthorized
{
  "type": "unauthorized",
  "message": "Authentication required"
}

// 403 Forbidden
{
  "type": "not_allowed",
  "message": "Insufficient permissions to perform this action"
}

// 404 Not Found
{
  "type": "not_found",
  "message": "Product with id prod_123 was not found"
}

// 409 Conflict
{
  "type": "duplicate_error",
  "message": "Product with handle 'medusa-tshirt' already exists"
}

// 500 Internal Server Error
{
  "type": "unexpected_state",
  "message": "An unexpected error occurred"
}
```

---

## Rate Limiting

### Rate Limit Headers

```
X-RateLimit-Limit: 100          # Max requests per window
X-RateLimit-Remaining: 95       # Remaining requests
X-RateLimit-Reset: 1700395200   # Unix timestamp when limit resets
Retry-After: 60                 # Seconds to wait (429 response)
```

### Rate Limit Response

```typescript
// 429 Too Many Requests
{
  "type": "rate_limit_exceeded",
  "message": "Too many requests. Please try again later.",
  "retry_after": 60
}
```

### Rate Limits

| Endpoint Type | Requests | Window |
|---------------|----------|--------|
| General API | 100 | 15 minutes |
| Authentication | 5 | 15 minutes |
| Search | 50 | 1 minute |

---

## Versioning Strategy

### URL Versioning (Future)

```bash
# Current (no version in URL)
GET /admin/products

# Future versioning
GET /v2/admin/products
```

### Header Versioning

```bash
curl -X GET http://localhost:9000/admin/products \
  -H "Accept: application/vnd.medusa.v2+json"
```

### Deprecation Headers

```
Deprecation: Sun, 01 Jan 2025 00:00:00 GMT
Sunset: Sun, 01 Jul 2025 00:00:00 GMT
Link: <https://docs.medusajs.com/api/v2>; rel="successor-version"
```

---

## API Examples

### Products API

#### List Products

```bash
# Request
GET /admin/products?limit=20&offset=0&status=published

# Response
{
  "products": [
    {
      "id": "prod_01JCQR9Z4GXK8KVN4GZXQY7W8E",
      "title": "Medusa T-Shirt",
      "handle": "medusa-tshirt",
      "subtitle": "Comfortable cotton tee",
      "description": "High quality t-shirt with Medusa logo",
      "status": "published",
      "thumbnail": "https://cdn.example.com/tshirt.jpg",
      "created_at": "2024-11-19T10:00:00.000Z",
      "updated_at": "2024-11-19T10:00:00.000Z",
      "variants": [
        {
          "id": "variant_01JCQR9Z4HMK8KVN4GZXQY7W9F",
          "title": "Small",
          "sku": "TSHIRT-S",
          "inventory_quantity": 100,
          "prices": [
            {
              "amount": 2999,
              "currency_code": "usd"
            }
          ]
        }
      ],
      "options": [
        {
          "id": "opt_01JCQR9Z4IMK8KVN4GZXQY7W0G",
          "title": "Size",
          "values": ["Small", "Medium", "Large"]
        }
      ],
      "images": [
        {
          "id": "img_01JCQR9Z4JNK8KVN4GZXQY7W1H",
          "url": "https://cdn.example.com/tshirt-1.jpg"
        }
      ]
    }
  ],
  "count": 150,
  "offset": 0,
  "limit": 20
}
```

#### Get Single Product

```bash
# Request
GET /admin/products/prod_01JCQR9Z4GXK8KVN4GZXQY7W8E

# Response
{
  "product": {
    "id": "prod_01JCQR9Z4GXK8KVN4GZXQY7W8E",
    "title": "Medusa T-Shirt",
    "handle": "medusa-tshirt",
    "status": "published",
    "variants": [...],
    "options": [...],
    "images": [...]
  }
}
```

#### Create Product

```bash
# Request
POST /admin/products
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "title": "New Product",
  "handle": "new-product",
  "description": "Product description",
  "status": "draft",
  "variants": [
    {
      "title": "Default Variant",
      "prices": [
        {
          "amount": 1999,
          "currency_code": "usd"
        }
      ]
    }
  ]
}

# Response (201 Created)
{
  "product": {
    "id": "prod_NEW123",
    "title": "New Product",
    "handle": "new-product",
    "status": "draft",
    "created_at": "2024-11-19T10:30:00.000Z",
    "updated_at": "2024-11-19T10:30:00.000Z",
    "variants": [...]
  }
}
```

#### Update Product

```bash
# Request
POST /admin/products/prod_123
Content-Type: application/json

{
  "title": "Updated Product Title",
  "description": "Updated description"
}

# Response (200 OK)
{
  "product": {
    "id": "prod_123",
    "title": "Updated Product Title",
    "description": "Updated description",
    "updated_at": "2024-11-19T11:00:00.000Z",
    ...
  }
}
```

#### Delete Product

```bash
# Request
DELETE /admin/products/prod_123

# Response (200 OK)
{
  "id": "prod_123",
  "object": "product",
  "deleted": true
}
```

### Orders API

#### List Orders

```bash
# Request
GET /admin/orders?status=pending&limit=10

# Response
{
  "orders": [
    {
      "id": "order_01JCQR9Z4KOK8KVN4GZXQY7W2I",
      "display_id": 1001,
      "status": "pending",
      "email": "customer@example.com",
      "currency_code": "usd",
      "total": 5999,
      "subtotal": 4999,
      "tax_total": 500,
      "shipping_total": 500,
      "items": [
        {
          "id": "item_01JCQR9Z4LPK8KVN4GZXQY7W3J",
          "title": "Medusa T-Shirt",
          "quantity": 2,
          "unit_price": 2999,
          "total": 5998
        }
      ],
      "shipping_address": {
        "first_name": "John",
        "last_name": "Doe",
        "address_1": "123 Main St",
        "city": "New York",
        "country_code": "us",
        "postal_code": "10001"
      },
      "customer": {
        "id": "cus_01JCQR9Z4MQK8KVN4GZXQY7W4K",
        "email": "customer@example.com",
        "first_name": "John",
        "last_name": "Doe"
      },
      "created_at": "2024-11-19T09:00:00.000Z"
    }
  ],
  "count": 45,
  "offset": 0,
  "limit": 10
}
```

### Cart API (Store)

#### Create Cart

```bash
# Request
POST /store/carts
Content-Type: application/json

{
  "region_id": "reg_01JCQR9Z4NRK8KVN4GZXQY7W5L",
  "items": [
    {
      "variant_id": "variant_123",
      "quantity": 2
    }
  ]
}

# Response (201 Created)
{
  "cart": {
    "id": "cart_01JCQR9Z4OSK8KVN4GZXQY7W6M",
    "region_id": "reg_01JCQR9Z4NRK8KVN4GZXQY7W5L",
    "currency_code": "usd",
    "items": [
      {
        "id": "item_01JCQR9Z4PTK8KVN4GZXQY7W7N",
        "variant_id": "variant_123",
        "quantity": 2,
        "unit_price": 2999,
        "total": 5998
      }
    ],
    "subtotal": 5998,
    "total": 5998,
    "created_at": "2024-11-19T10:00:00.000Z"
  }
}
```

#### Complete Cart (Create Order)

```bash
# Request
POST /store/carts/cart_123/complete
Content-Type: application/json

{}

# Response (200 OK)
{
  "type": "order",
  "order": {
    "id": "order_456",
    "display_id": 1002,
    "status": "pending",
    "total": 5998,
    ...
  }
}
```

### Auth API

```bash
# Request
POST /auth/customer/emailpass
Content-Type: application/json

{
  "email": "customer@example.com",
  "password": "password123"
}

# Response (200 OK)
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Testing APIs

### cURL Examples

```bash
# GET request
curl -X GET "http://localhost:9000/admin/products" \
  -H "Authorization: Bearer TOKEN"

# POST request
curl -X POST "http://localhost:9000/admin/products" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{
    "title": "Test Product",
    "handle": "test-product"
  }'

# Pretty print response
curl -X GET "http://localhost:9000/admin/products" \
  -H "Authorization: Bearer TOKEN" | jq

# Save response to file
curl -X GET "http://localhost:9000/admin/products" \
  -H "Authorization: Bearer TOKEN" \
  -o products.json
```

### Postman Collection

```json
{
  "info": {
    "name": "Medusa API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{authToken}}",
        "type": "string"
      }
    ]
  },
  "item": [
    {
      "name": "Products",
      "item": [
        {
          "name": "List Products",
          "request": {
            "method": "GET",
            "url": "{{baseUrl}}/admin/products"
          }
        },
        {
          "name": "Create Product",
          "request": {
            "method": "POST",
            "url": "{{baseUrl}}/admin/products",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"title\": \"Test Product\",\n  \"handle\": \"test-product\"\n}"
            }
          }
        }
      ]
    }
  ]
}
```

### Using Insomnia

```yaml
# insomnia-collection.yaml
_type: export
__export_format: 4
resources:
  - _id: env_base
    _type: environment
    name: Base Environment
    data:
      baseUrl: http://localhost:9000
      authToken: your-token-here

  - _id: req_list_products
    _type: request
    name: List Products
    method: GET
    url: "{{ _.baseUrl }}/admin/products"
    headers:
      - name: Authorization
        value: "Bearer {{ _.authToken }}"
```

---

## SDK Usage

### JavaScript SDK

```bash
npm install @medusajs/js-sdk
```

```typescript
import Medusa from "@medusajs/js-sdk"

// Initialize SDK
const medusa = new Medusa({
  baseUrl: "http://localhost:9000",
  apiKey: "your-api-key", // or use auth token
})

// List products
const products = await medusa.admin.product.list({
  limit: 20,
  offset: 0,
  status: "published",
})

// Get product
const product = await medusa.admin.product.retrieve("prod_123")

// Create product
const newProduct = await medusa.admin.product.create({
  title: "New Product",
  handle: "new-product",
  variants: [
    {
      title: "Default",
      prices: [{ amount: 1999, currency_code: "usd" }],
    },
  ],
})

// Update product
const updatedProduct = await medusa.admin.product.update("prod_123", {
  title: "Updated Title",
})

// Delete product
await medusa.admin.product.delete("prod_123")
```

### Store API with SDK

```typescript
// Create cart
const cart = await medusa.store.cart.create({
  region_id: "reg_123",
})

// Add item to cart
await medusa.store.cart.addLineItem(cart.id, {
  variant_id: "variant_123",
  quantity: 2,
})

// Update line item
await medusa.store.cart.updateLineItem(cart.id, "item_123", {
  quantity: 3,
})

// Complete cart
const order = await medusa.store.cart.complete(cart.id)
```

### Error Handling with SDK

```typescript
try {
  const product = await medusa.admin.product.retrieve("invalid_id")
} catch (error) {
  if (error.response?.status === 404) {
    console.error("Product not found")
  } else if (error.response?.status === 401) {
    console.error("Unauthorized")
  } else {
    console.error("Error:", error.message)
  }
}
```

---

## Best Practices

### API Design

✅ **DO:**
- Use plural nouns for resource names (`/products`, not `/product`)
- Use lowercase in URLs
- Use hyphens for multi-word resources (`/product-types`)
- Return appropriate status codes
- Include pagination metadata
- Validate all inputs
- Use meaningful error messages

❌ **DON'T:**
- Use verbs in URLs (`/getProducts` ❌)
- Return sensitive data unnecessarily
- Return entire database records
- Ignore query parameter validation
- Skip rate limiting
- Return stack traces to clients

### Client Best Practices

✅ **DO:**
- Implement retry logic with exponential backoff
- Handle rate limiting (429 responses)
- Validate responses
- Use HTTPS in production
- Store tokens securely
- Implement proper error handling

❌ **DON'T:**
- Hardcode API keys
- Ignore SSL certificate validation
- Retry failed requests immediately
- Store tokens in localStorage (use httpOnly cookies)

---

## Resources

- [Medusa API Reference](https://docs.medusajs.com/api/admin)
- [REST API Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [JWT.io](https://jwt.io/) - JWT Debugger

---

**Next Steps:**
- [Security Guide](./SECURITY_GUIDE.md) - Secure your APIs
- [Testing Guide](./TESTING_GUIDE.md) - Test your endpoints
- [Database Schema](./DATABASE_SCHEMA.md) - Understand data models
