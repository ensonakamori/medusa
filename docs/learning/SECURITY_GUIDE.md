# Security Guide

Comprehensive security best practices for building secure Medusa applications.

## Table of Contents

- [Security Overview](#security-overview)
- [Authentication Architecture](#authentication-architecture)
- [Authorization Patterns](#authorization-patterns)
- [Input Validation](#input-validation)
- [SQL Injection Prevention](#sql-injection-prevention)
- [XSS Prevention](#xss-prevention)
- [CSRF Protection](#csrf-protection)
- [Rate Limiting](#rate-limiting)
- [Password Security](#password-security)
- [Payment Data Security](#payment-data-security)
- [Secrets Management](#secrets-management)
- [Security Headers](#security-headers)
- [HTTPS Enforcement](#https-enforcement)
- [Audit Logging](#audit-logging)
- [Dependency Security](#dependency-security)

---

## Security Overview

### Security Layers

```mermaid
graph TB
    A[Internet] --> B[HTTPS/TLS]
    B --> C[Rate Limiting]
    C --> D[CSRF Protection]
    D --> E[Authentication]
    E --> F[Authorization]
    F --> G[Input Validation]
    G --> H[Business Logic]
    H --> I[SQL Injection Prevention]
    I --> J[Database]
```

### Security Principles

✅ **Defense in Depth**: Multiple layers of security
✅ **Least Privilege**: Grant minimum necessary permissions
✅ **Zero Trust**: Verify everything, trust nothing
✅ **Secure by Default**: Safe configurations out of the box
✅ **Fail Securely**: Errors should not leak sensitive data

---

## Authentication Architecture

### JWT-Based Authentication

Medusa uses JWT (JSON Web Tokens) for API authentication:

```typescript
// Authentication flow
import { authenticate } from "@medusajs/framework/http"

// Protect admin routes
export const GET = authenticate("admin")(
  async (req, res) => {
    // req.auth_context contains authenticated user
    const userId = req.auth_context.actor_id

    // Access user data
    const user = await userService.retrieve(userId)

    res.json({ user })
  }
)

// Protect store routes (customer authentication)
export const GET = authenticate("store")(
  async (req, res) => {
    // Customer authentication
    const customerId = req.auth_context.actor_id

    const customer = await customerService.retrieve(customerId)

    res.json({ customer })
  }
)
```

### Auth Module Service

**Location**: `/home/user/medusa/packages/modules/auth/`

```typescript
import { IAuthModuleService } from "@medusajs/framework/types"

// Authenticate user
const result = await authModuleService.authenticate(
  "emailpass", // provider
  {
    body: {
      email: "user@example.com",
      password: "secure_password",
    },
  }
)

if (result.success) {
  const { authIdentity } = result

  // Create session/JWT
  const token = generateToken({
    actor_id: authIdentity.id,
    app_metadata: {
      user_id: authIdentity.provider_identities[0].entity_id,
    },
  })

  res.json({ token })
} else {
  // Authentication failed
  res.status(401).json({ error: result.error })
}
```

### Session Management

```typescript
// Session-based auth (cookies)
import session from "express-session"
import RedisStore from "connect-redis"

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: true, // HTTPS only
      httpOnly: true, // Not accessible via JavaScript
      sameSite: "strict", // CSRF protection
      maxAge: 24 * 60 * 60 * 1000, // 24 hours
    },
  })
)
```

### Multi-Provider Authentication

```typescript
// Configure auth providers
// medusa-config.js
module.exports = {
  modules: [
    {
      resolve: "@medusajs/auth",
      options: {
        providers: [
          {
            resolve: "@medusajs/auth-emailpass",
            id: "emailpass",
          },
          {
            resolve: "@medusajs/auth-google",
            id: "google",
            options: {
              clientId: process.env.GOOGLE_CLIENT_ID,
              clientSecret: process.env.GOOGLE_CLIENT_SECRET,
              callbackUrl: process.env.GOOGLE_CALLBACK_URL,
            },
          },
          {
            resolve: "@medusajs/auth-github",
            id: "github",
            options: {
              clientId: process.env.GITHUB_CLIENT_ID,
              clientSecret: process.env.GITHUB_CLIENT_SECRET,
            },
          },
        ],
      },
    },
  ],
}
```

⚠️ **Security Warning**: Never commit OAuth secrets to version control!

---

## Authorization Patterns

### Scope-Based Authorization

```typescript
// API Key scopes
import { MedusaRequest } from "@medusajs/framework/http"

// Check if user has required scope
function requireScope(scope: string) {
  return (req: MedusaRequest, res, next) => {
    const scopes = req.auth_context.scopes || []

    if (!scopes.includes(scope)) {
      return res.status(403).json({
        error: "Forbidden",
        message: `Required scope: ${scope}`,
      })
    }

    next()
  }
}

// Use in routes
export const POST = [
  authenticate("admin"),
  requireScope("products:write"),
  async (req, res) => {
    // Only users with products:write scope can create products
    const product = await productService.create(req.body)
    res.json({ product })
  },
]
```

### Role-Based Access Control (RBAC)

```typescript
// Define roles and permissions
const roles = {
  admin: {
    permissions: ["*"], // All permissions
  },
  manager: {
    permissions: [
      "products:read",
      "products:write",
      "orders:read",
      "orders:write",
    ],
  },
  viewer: {
    permissions: ["products:read", "orders:read"],
  },
}

// Check permission
function hasPermission(user, permission) {
  const role = roles[user.role]
  if (!role) return false

  return (
    role.permissions.includes("*") ||
    role.permissions.includes(permission)
  )
}

// Middleware
function requirePermission(permission) {
  return async (req, res, next) => {
    const user = await userService.retrieve(req.auth_context.actor_id)

    if (!hasPermission(user, permission)) {
      return res.status(403).json({
        error: "Forbidden",
        message: `Required permission: ${permission}`,
      })
    }

    next()
  }
}
```

### Resource-Based Authorization

```typescript
// Ensure user owns the resource
async function ensureOwnership(req, res, next) {
  const customerId = req.auth_context.actor_id
  const orderId = req.params.id

  const order = await orderService.retrieve(orderId)

  if (order.customer_id !== customerId) {
    return res.status(403).json({
      error: "Forbidden",
      message: "You can only access your own orders",
    })
  }

  next()
}

// Use in route
export const GET = [
  authenticate("store"),
  ensureOwnership,
  async (req, res) => {
    const order = await orderService.retrieve(req.params.id)
    res.json({ order })
  },
]
```

### API Key Authentication

```typescript
// API key validation middleware
async function validateApiKey(req, res, next) {
  const apiKey = req.headers["x-api-key"]

  if (!apiKey) {
    return res.status(401).json({ error: "API key required" })
  }

  try {
    const keyData = await apiKeyService.retrieve(apiKey, {
      select: ["id", "token", "type", "created_by"],
    })

    // Check if key is revoked
    if (keyData.revoked_at) {
      return res.status(401).json({ error: "API key revoked" })
    }

    // Attach to request
    req.api_key = keyData
    next()
  } catch (error) {
    return res.status(401).json({ error: "Invalid API key" })
  }
}
```

---

## Input Validation

### Zod Schema Validation

Medusa uses Zod for input validation:

```typescript
import { z } from "zod"

// Define validation schema
export const CreateProductSchema = z.object({
  title: z.string().min(1).max(255),
  handle: z
    .string()
    .min(1)
    .regex(/^[a-z0-9-]+$/, "Handle must be lowercase with hyphens"),
  description: z.string().optional(),
  status: z.enum(["draft", "published", "rejected"]).default("draft"),
  variants: z
    .array(
      z.object({
        title: z.string().min(1),
        sku: z.string().optional(),
        prices: z.array(
          z.object({
            amount: z.number().positive(),
            currency_code: z.string().length(3),
          })
        ),
      })
    )
    .min(1),
  metadata: z.record(z.unknown()).optional(),
})

// Use in route
export const POST = async (req, res) => {
  try {
    // Validate input
    const validatedData = CreateProductSchema.parse(req.body)

    // Create product with validated data
    const product = await productService.create(validatedData)

    res.json({ product })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: "Validation failed",
        details: error.errors,
      })
    }
    throw error
  }
}
```

### Real Example: Admin Product Validators

**Location**: `/home/user/medusa/packages/medusa/src/api/admin/products/validators.ts`

```typescript
import { z } from "zod"
import { ProductStatus } from "@medusajs/framework/utils"
import { createSelectParams, createOperatorMap } from "../../utils/validators"

const statusEnum = z.nativeEnum(ProductStatus)

export const AdminGetProductsParamsFields = z.object({
  q: z.string().optional(),
  id: z.union([z.string(), z.array(z.string())]).optional(),
  status: z.union([statusEnum, z.array(statusEnum)]).optional(),
  title: z.union([z.string(), z.array(z.string())]).optional(),
  handle: z.union([z.string(), z.array(z.string())]).optional(),
  is_giftcard: z.boolean().optional(),
  created_at: createOperatorMap().optional(),
  updated_at: createOperatorMap().optional(),
  deleted_at: createOperatorMap().optional(),
})
```

### Sanitizing User Input

```typescript
import validator from "validator"

// Sanitize strings
function sanitizeString(input: string): string {
  return validator.escape(validator.trim(input))
}

// Validate email
function validateEmail(email: string): boolean {
  return validator.isEmail(email)
}

// Validate URL
function validateUrl(url: string): boolean {
  return validator.isURL(url, {
    protocols: ["http", "https"],
    require_protocol: true,
  })
}

// Use in service
async function createCustomer(data) {
  const customerData = {
    email: validateEmail(data.email) ? data.email : null,
    first_name: sanitizeString(data.first_name),
    last_name: sanitizeString(data.last_name),
    phone: validator.isMobilePhone(data.phone) ? data.phone : null,
  }

  if (!customerData.email) {
    throw new Error("Invalid email address")
  }

  return await customerRepository.create(customerData)
}
```

### Path Traversal Prevention

```typescript
import path from "path"

// ❌ Vulnerable to path traversal
app.get("/files/:filename", (req, res) => {
  const filename = req.params.filename
  res.sendFile(`/uploads/${filename}`) // Can access ../../etc/passwd
})

// ✅ Secure version
app.get("/files/:filename", (req, res) => {
  const filename = path.basename(req.params.filename) // Remove directory parts
  const filepath = path.join(__dirname, "uploads", filename)

  // Ensure file is within uploads directory
  if (!filepath.startsWith(path.join(__dirname, "uploads"))) {
    return res.status(403).json({ error: "Forbidden" })
  }

  res.sendFile(filepath)
})
```

---

## SQL Injection Prevention

### MikroORM Protection

MikroORM provides automatic SQL injection protection through parameterized queries:

```typescript
// ✅ Safe: MikroORM automatically parameterizes
const products = await productRepository.find({
  title: userInput, // Automatically escaped
})

// ✅ Safe: Using query builder
const products = await em
  .createQueryBuilder(Product)
  .where({ title: userInput })
  .getResult()

// ⚠️ Dangerous: Raw SQL (use with caution)
// Only use when necessary, and always parameterize
const products = await em.getConnection().execute(
  "SELECT * FROM product WHERE title = ?",
  [userInput] // Parameterized - safe
)

// ❌ NEVER DO THIS: String concatenation
const products = await em.getConnection().execute(
  `SELECT * FROM product WHERE title = '${userInput}'` // VULNERABLE!
)
```

### Query Parameter Validation

```typescript
// Validate and sanitize query parameters
function validateQueryFilters(filters) {
  const allowedFields = ["status", "title", "handle", "created_at"]

  const validatedFilters = {}

  for (const [key, value] of Object.entries(filters)) {
    // Only allow whitelisted fields
    if (!allowedFields.includes(key)) {
      continue
    }

    // Validate field types
    if (key === "status") {
      if (!["draft", "published", "rejected"].includes(value)) {
        throw new Error("Invalid status value")
      }
    }

    validatedFilters[key] = value
  }

  return validatedFilters
}

// Use in route
export const GET = async (req, res) => {
  const filters = validateQueryFilters(req.query)
  const products = await productService.list(filters)
  res.json({ products })
}
```

---

## XSS Prevention

### React Auto-Escaping

React automatically escapes values in JSX, providing XSS protection:

```typescript
// ✅ Safe: React auto-escapes
function ProductCard({ product }) {
  return (
    <div>
      <h2>{product.title}</h2> {/* Auto-escaped */}
      <p>{product.description}</p> {/* Auto-escaped */}
    </div>
  )
}

// ⚠️ Dangerous: dangerouslySetInnerHTML
function ProductDescription({ html }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: html }} // Can execute scripts!
    />
  )
}

// ✅ Safe: Sanitize HTML first
import DOMPurify from "dompurify"

function ProductDescription({ html }) {
  const sanitized = DOMPurify.sanitize(html)
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />
}
```

### Content Security Policy (CSP)

```typescript
// Set CSP headers
import helmet from "helmet"

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"], // Avoid unsafe-inline in production
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.medusajs.com"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  })
)
```

### Output Encoding

```typescript
// Encode for different contexts
import { escape, escapeAttr } from "html-escaper"

// HTML context
const htmlSafe = escape(userInput)

// Attribute context
const attrSafe = escapeAttr(userInput)

// JavaScript context
const jsSafe = JSON.stringify(userInput)

// URL context
const urlSafe = encodeURIComponent(userInput)
```

---

## CSRF Protection

### CSRF Tokens

```typescript
import csrf from "csurf"

// Enable CSRF protection
const csrfProtection = csrf({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
  },
})

// Apply to state-changing routes
app.post("/api/products", csrfProtection, async (req, res) => {
  // CSRF token validated automatically
  const product = await productService.create(req.body)
  res.json({ product })
})

// Send token to client
app.get("/api/csrf-token", csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() })
})
```

### SameSite Cookie Attribute

```typescript
// Set SameSite attribute on cookies
res.cookie("session", sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: "strict", // Prevents CSRF
  maxAge: 86400000,
})
```

### Double Submit Cookie Pattern

```typescript
// Alternative CSRF protection
import crypto from "crypto"

function generateCsrfToken() {
  return crypto.randomBytes(32).toString("hex")
}

app.use((req, res, next) => {
  if (!req.cookies.csrfToken) {
    const token = generateCsrfToken()
    res.cookie("csrfToken", token, {
      httpOnly: false, // Accessible to JavaScript
      secure: true,
      sameSite: "strict",
    })
  }
  next()
})

// Validate CSRF token
function validateCsrfToken(req, res, next) {
  const cookieToken = req.cookies.csrfToken
  const headerToken = req.headers["x-csrf-token"]

  if (!cookieToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: "Invalid CSRF token" })
  }

  next()
}
```

---

## Rate Limiting

### Express Rate Limit

```typescript
import rateLimit from "express-rate-limit"

// General API rate limiting
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: "Too many requests, please try again later",
  standardHeaders: true,
  legacyHeaders: false,
})

app.use("/api", apiLimiter)

// Strict rate limiting for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 login attempts per window
  message: "Too many login attempts, please try again later",
  skipSuccessfulRequests: true, // Don't count successful logins
})

app.post("/auth/login", authLimiter, async (req, res) => {
  // Login logic
})
```

### Redis-Based Rate Limiting

```typescript
import { RateLimiterRedis } from "rate-limiter-flexible"

const rateLimiter = new RateLimiterRedis({
  storeClient: redisClient,
  keyPrefix: "rl",
  points: 10, // Number of requests
  duration: 1, // Per second
  blockDuration: 60, // Block for 60 seconds if exceeded
})

async function rateLimitMiddleware(req, res, next) {
  try {
    await rateLimiter.consume(req.ip)
    next()
  } catch (error) {
    res.status(429).json({
      error: "Too Many Requests",
      retryAfter: error.msBeforeNext / 1000,
    })
  }
}

app.use(rateLimitMiddleware)
```

### DDoS Protection

```typescript
// Slow down repeated requests
import slowDown from "express-slow-down"

const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000, // 15 minutes
  delayAfter: 50, // Allow 50 requests at full speed
  delayMs: 500, // Add 500ms delay per request after delayAfter
  maxDelayMs: 20000, // Maximum delay of 20 seconds
})

app.use("/api", speedLimiter)
```

---

## Password Security

### Password Hashing

```typescript
import bcrypt from "bcrypt"

// Hash password before storing
async function hashPassword(password: string): Promise<string> {
  const saltRounds = 10
  return await bcrypt.hash(password, saltRounds)
}

// Verify password
async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return await bcrypt.compare(password, hash)
}

// Use in auth provider
class EmailPassProvider {
  async authenticate({ email, password }) {
    const identity = await this.authService.retrieveByEmail(email)

    if (!identity) {
      throw new Error("Invalid email or password")
    }

    const isValid = await verifyPassword(
      password,
      identity.provider_metadata.password_hash
    )

    if (!isValid) {
      throw new Error("Invalid email or password")
    }

    return { success: true, authIdentity: identity }
  }
}
```

### Password Strength Validation

```typescript
import validator from "validator"

function validatePasswordStrength(password: string): {
  valid: boolean
  errors: string[]
} {
  const errors = []

  // Minimum length
  if (password.length < 8) {
    errors.push("Password must be at least 8 characters")
  }

  // Maximum length (prevent DoS via bcrypt)
  if (password.length > 72) {
    errors.push("Password must be less than 72 characters")
  }

  // Require uppercase
  if (!/[A-Z]/.test(password)) {
    errors.push("Password must contain at least one uppercase letter")
  }

  // Require lowercase
  if (!/[a-z]/.test(password)) {
    errors.push("Password must contain at least one lowercase letter")
  }

  // Require number
  if (!/\d/.test(password)) {
    errors.push("Password must contain at least one number")
  }

  // Require special character
  if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
    errors.push("Password must contain at least one special character")
  }

  // Check against common passwords
  const commonPasswords = [
    "password",
    "123456",
    "qwerty",
    "admin",
    "letmein",
  ]
  if (commonPasswords.includes(password.toLowerCase())) {
    errors.push("Password is too common")
  }

  return {
    valid: errors.length === 0,
    errors,
  }
}
```

### Password Reset Security

```typescript
import crypto from "crypto"

// Generate secure reset token
function generateResetToken(): string {
  return crypto.randomBytes(32).toString("hex")
}

// Hash token before storing
function hashToken(token: string): string {
  return crypto.createHash("sha256").update(token).digest("hex")
}

// Password reset flow
async function requestPasswordReset(email: string) {
  const user = await userService.retrieveByEmail(email)

  if (!user) {
    // Don't reveal if email exists
    return { message: "If email exists, reset link was sent" }
  }

  const resetToken = generateResetToken()
  const hashedToken = hashToken(resetToken)

  // Store hashed token with expiration
  await userService.update(user.id, {
    reset_token: hashedToken,
    reset_token_expires: new Date(Date.now() + 3600000), // 1 hour
  })

  // Send email with unhashed token
  await emailService.send({
    to: email,
    subject: "Password Reset",
    template: "password-reset",
    data: {
      resetLink: `${process.env.FRONTEND_URL}/reset-password?token=${resetToken}`,
    },
  })

  return { message: "If email exists, reset link was sent" }
}

// Reset password
async function resetPassword(token: string, newPassword: string) {
  const hashedToken = hashToken(token)

  const user = await userService.retrieve({
    reset_token: hashedToken,
    reset_token_expires: { $gt: new Date() },
  })

  if (!user) {
    throw new Error("Invalid or expired reset token")
  }

  // Validate new password
  const validation = validatePasswordStrength(newPassword)
  if (!validation.valid) {
    throw new Error(validation.errors.join(", "))
  }

  // Update password and clear reset token
  await userService.update(user.id, {
    password_hash: await hashPassword(newPassword),
    reset_token: null,
    reset_token_expires: null,
  })
}
```

---

## Payment Data Security

### PCI Compliance

⚠️ **Never store sensitive payment data!**

```typescript
// ❌ NEVER store these:
// - Full credit card numbers
// - CVV/CVC codes
// - PIN numbers

// ✅ Safe: Use payment provider tokens
async function processPayment(paymentData) {
  // Send card data directly to payment provider
  const token = await stripeProvider.createToken({
    number: paymentData.cardNumber, // Sent directly to Stripe
    exp_month: paymentData.expMonth,
    exp_year: paymentData.expYear,
    cvc: paymentData.cvc,
  })

  // Store only the token, not card details
  const payment = await paymentService.create({
    amount: paymentData.amount,
    currency: paymentData.currency,
    provider_token: token.id, // Safe to store
    last4: token.card.last4, // Safe: last 4 digits only
    brand: token.card.brand, // Safe: card brand
  })

  return payment
}
```

### Stripe Payment Intent

```typescript
import Stripe from "stripe"

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: "2023-10-16",
})

async function createPaymentIntent(amount, currency, customerId) {
  // Card data never touches your server
  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency,
    customer: customerId,
    // Payment method collected on frontend via Stripe Elements
    automatic_payment_methods: {
      enabled: true,
    },
  })

  // Return client secret for frontend
  return {
    clientSecret: paymentIntent.client_secret,
  }
}
```

### Webhook Signature Verification

```typescript
// Verify webhook signatures from payment providers
app.post(
  "/webhooks/stripe",
  express.raw({ type: "application/json" }),
  async (req, res) => {
    const sig = req.headers["stripe-signature"]
    const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET

    let event

    try {
      // Verify signature
      event = stripe.webhooks.constructEvent(req.body, sig, webhookSecret)
    } catch (error) {
      console.error("Webhook signature verification failed:", error)
      return res.status(400).send(`Webhook Error: ${error.message}`)
    }

    // Process verified event
    switch (event.type) {
      case "payment_intent.succeeded":
        await handlePaymentSuccess(event.data.object)
        break
      case "payment_intent.payment_failed":
        await handlePaymentFailure(event.data.object)
        break
    }

    res.json({ received: true })
  }
)
```

---

## Secrets Management

### Environment Variables

```bash
# .env (NEVER commit to git!)
DATABASE_URL=postgresql://user:pass@localhost:5432/medusa
STRIPE_SECRET_KEY=sk_live_...
JWT_SECRET=your-super-secret-jwt-key
SESSION_SECRET=your-super-secret-session-key
SENDGRID_API_KEY=SG.xxx...
```

### .gitignore

```
# Environment files
.env
.env.local
.env.production
.env.*.local

# Secrets
secrets/
*.key
*.pem
```

### Secret Rotation

```typescript
// Support multiple JWT secrets for rotation
const secrets = [
  process.env.JWT_SECRET_CURRENT,
  process.env.JWT_SECRET_PREVIOUS, // For grace period
]

function verifyToken(token: string) {
  for (const secret of secrets) {
    try {
      return jwt.verify(token, secret)
    } catch (error) {
      continue
    }
  }
  throw new Error("Invalid token")
}
```

### Using Secret Managers

```typescript
// AWS Secrets Manager
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager"

async function getSecret(secretName: string) {
  const client = new SecretsManagerClient({ region: "us-east-1" })

  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretName })
  )

  return JSON.parse(response.SecretString)
}

// Load secrets at startup
const secrets = await getSecret("medusa/production")
process.env.DATABASE_URL = secrets.DATABASE_URL
process.env.STRIPE_SECRET_KEY = secrets.STRIPE_SECRET_KEY
```

---

## Security Headers

### Helmet.js

```typescript
import helmet from "helmet"

app.use(
  helmet({
    // HSTS: Force HTTPS
    hsts: {
      maxAge: 31536000, // 1 year
      includeSubDomains: true,
      preload: true,
    },

    // Content Security Policy
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
      },
    },

    // X-Frame-Options: Prevent clickjacking
    frameguard: {
      action: "deny",
    },

    // X-Content-Type-Options: Prevent MIME sniffing
    noSniff: true,

    // Referrer-Policy
    referrerPolicy: {
      policy: "same-origin",
    },
  })
)
```

### CORS Configuration

```typescript
import cors from "cors"

app.use(
  cors({
    origin: process.env.FRONTEND_URL,
    credentials: true, // Allow cookies
    methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
    allowedHeaders: ["Content-Type", "Authorization"],
  })
)
```

---

## HTTPS Enforcement

### Force HTTPS

```typescript
// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (req.headers["x-forwarded-proto"] !== "https" && process.env.NODE_ENV === "production") {
    return res.redirect(301, `https://${req.hostname}${req.url}`)
  }
  next()
})
```

### Let's Encrypt SSL

```bash
# Install certbot
sudo apt-get install certbot

# Generate certificate
sudo certbot certonly --standalone -d your-domain.com

# Auto-renewal
sudo certbot renew --dry-run
```

---

## Audit Logging

### Track Security Events

```typescript
// Audit log service
class AuditLogService {
  async log(event: {
    action: string
    actor_id: string
    resource_type: string
    resource_id: string
    changes?: object
    ip_address?: string
    user_agent?: string
  }) {
    await auditLogRepository.create({
      ...event,
      timestamp: new Date(),
    })
  }
}

// Use in services
async function deleteProduct(productId: string, userId: string, req) {
  await productRepository.delete(productId)

  // Log deletion
  await auditLogService.log({
    action: "product.deleted",
    actor_id: userId,
    resource_type: "product",
    resource_id: productId,
    ip_address: req.ip,
    user_agent: req.headers["user-agent"],
  })
}
```

---

## Dependency Security

### Regular Updates

```bash
# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Force fix (may break things)
npm audit fix --force
```

### Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "security-team"
```

---

## Security Checklist

✅ **Authentication:**
- [ ] Use strong password hashing (bcrypt)
- [ ] Implement password strength requirements
- [ ] Use secure JWT secrets
- [ ] Implement token expiration
- [ ] Support multi-factor authentication (MFA)

✅ **Authorization:**
- [ ] Implement least privilege access
- [ ] Use role-based or scope-based authorization
- [ ] Verify resource ownership
- [ ] Validate API keys

✅ **Data Protection:**
- [ ] Never store plain-text passwords
- [ ] Never store full credit card numbers
- [ ] Encrypt sensitive data at rest
- [ ] Use HTTPS for data in transit

✅ **Input Validation:**
- [ ] Validate all user input with Zod
- [ ] Sanitize HTML output
- [ ] Use parameterized queries
- [ ] Implement rate limiting

✅ **Security Headers:**
- [ ] Use Helmet.js for security headers
- [ ] Configure CORS properly
- [ ] Enable HSTS
- [ ] Set CSP headers

✅ **Monitoring:**
- [ ] Implement audit logging
- [ ] Monitor for suspicious activity
- [ ] Set up error tracking (Sentry)
- [ ] Regular security audits

---

**Next Steps:**
- [API Documentation](./API_DOCUMENTATION.md) - Secure API patterns
- [Testing Guide](./TESTING_GUIDE.md) - Security testing
- [Production Deployment](./DEVELOPMENT_WORKFLOW.md) - Production security
