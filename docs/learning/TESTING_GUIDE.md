# Testing Guide

Complete guide to testing in Medusa - from philosophy to practical implementation.

## Table of Contents

- [Testing Philosophy](#testing-philosophy)
- [Testing Strategy](#testing-strategy)
- [Test Types](#test-types)
- [Unit Testing with Jest](#unit-testing-with-jest)
- [Integration Testing](#integration-testing)
- [API Testing](#api-testing)
- [Frontend Testing](#frontend-testing)
- [Test Organization](#test-organization)
- [Mocking Strategies](#mocking-strategies)
- [Coverage Expectations](#coverage-expectations)
- [TDD Workflow](#tdd-workflow)
- [Running Tests](#running-tests)
- [Real Examples](#real-examples)

---

## Testing Philosophy

Medusa follows a comprehensive testing approach:

- **Test what matters**: Focus on behavior, not implementation details
- **Fast feedback**: Unit tests run in milliseconds, integration tests in seconds
- **Isolation**: Each test should be independent and repeatable
- **Real scenarios**: Integration tests use real database and services
- **Confidence**: Tests should give confidence to refactor and ship

### Testing Pyramid

```
        /\
       /  \
      / E2E \
     /________\
    /          \
   / Integration \
  /________________\
 /                  \
/   Unit Tests       \
/______________________\
```

- **70% Unit Tests**: Fast, isolated, test single functions/methods
- **20% Integration Tests**: Test module interactions with real dependencies
- **10% E2E/API Tests**: Full request-response cycles

---

## Testing Strategy

### What to Test

✅ **DO Test:**
- Business logic and calculations
- Data transformations
- API endpoints (request/response)
- Module service methods
- Workflow steps and compensation logic
- Database queries and relationships
- Validation and error handling
- Event emissions and subscriptions

⚠️ **DON'T Test:**
- Third-party libraries
- Framework internals
- Simple getters/setters
- Auto-generated code

---

## Test Types

### 1. Unit Tests
Fast, isolated tests for individual functions/methods.

**Location**: `src/__tests__/` or `__tests__/` directories
**Runner**: Jest 29.7.0
**File naming**: `*.spec.ts` or `*.test.ts`

### 2. Integration Tests
Test module interactions with real database and dependencies.

**Location**: `packages/modules/*/integration-tests/` or `integration-tests/modules/`
**Runner**: Vitest 3.0.5 with moduleIntegrationTestRunner
**File naming**: `*.spec.ts`

### 3. API Tests
Full HTTP request/response testing.

**Location**: `integration-tests/http/` or `integration-tests/api/`
**Runner**: Jest + Supertest
**File naming**: `*.spec.ts`

---

## Unit Testing with Jest

### Configuration

Root configuration at `/home/user/medusa/jest.config.js`:

```javascript
module.exports = {
  notify: true,
  verbose: true,
  roots: ["<rootDir>"],
  projects: [
    "<rootDir>/packages/*/jest.config.js",
    "<rootDir>/packages/cli/*/jest.config.js",
    "<rootDir>/packages/core/*/jest.config.js",
    "<rootDir>/packages/modules/*/jest.config.js",
    "<rootDir>/packages/modules/providers/*/jest.config.js",
  ],
  testPathIgnorePatterns: [
    `<rootDir>/examples/`,
    `<rootDir>/dist/`,
    `<rootDir>/node_modules/`,
    `__tests__/fixtures`,
  ],
}
```

### Basic Unit Test Structure

```typescript
// Example: Testing a utility function
describe("ProductUtils", () => {
  describe("calculateTotal", () => {
    it("should sum product prices correctly", () => {
      const products = [
        { price: 1000 },
        { price: 2000 },
        { price: 500 }
      ]

      const total = calculateTotal(products)

      expect(total).toBe(3500)
    })

    it("should handle empty array", () => {
      expect(calculateTotal([])).toBe(0)
    })

    it("should throw on invalid input", () => {
      expect(() => calculateTotal(null)).toThrow()
    })
  })
})
```

### Testing Best Practices

```typescript
// ✅ Good: Descriptive test names
it("should return 404 when product does not exist", () => {})

// ❌ Bad: Vague test names
it("works", () => {})
it("test 1", () => {})

// ✅ Good: Test one thing
it("should validate email format", () => {
  expect(validateEmail("test@example.com")).toBe(true)
})

// ❌ Bad: Testing multiple things
it("should validate and transform user", () => {
  expect(validateEmail("test@example.com")).toBe(true)
  expect(transformUser(user)).toHaveProperty("id")
  expect(saveUser(user)).resolves.toBeTruthy()
})
```

---

## Integration Testing

Integration tests verify module behavior with real dependencies (database, event bus, etc.).

### Module Integration Test Runner

Medusa provides `moduleIntegrationTestRunner` from `@medusajs/test-utils`:

```typescript
import { moduleIntegrationTestRunner, SuiteOptions } from "@medusajs/test-utils"
import { IAuthModuleService } from "@medusajs/framework/types"
import { Module, Modules } from "@medusajs/framework/utils"

jest.setTimeout(30000)

moduleIntegrationTestRunner({
  moduleName: Modules.AUTH,
  moduleOptions: {
    providers: [
      {
        resolve: "./providers/default-provider",
        id: "plaintextpass",
      },
    ],
  },
  testSuite: ({ service }: SuiteOptions<IAuthModuleService>) => {
    describe("Auth Module Service", () => {
      beforeEach(async () => {
        await service.createAuthIdentities({
          provider_identities: [
            {
              entity_id: "test@admin.com",
              provider: "plaintextpass",
              provider_metadata: {
                password: "plaintext",
              },
            },
          ],
        })
      })

      it("should authenticate with correct password", async () => {
        const result = await service.authenticate("plaintextpass", {
          body: {
            email: "test@admin.com",
            password: "plaintext",
          },
        })

        expect(result).toEqual(
          expect.objectContaining({
            success: true,
            authIdentity: expect.objectContaining({
              id: expect.any(String),
            }),
          })
        )
      })

      it("should fail with incorrect password", async () => {
        const result = await service.authenticate("plaintextpass", {
          body: {
            email: "test@admin.com",
            password: "wrong",
          },
        })

        expect(result).toEqual(
          expect.objectContaining({
            success: false,
            error: "Invalid email or password",
          })
        )
      })
    })
  },
})
```

### Real Example: Customer Module Integration Test

**Location**: `/home/user/medusa/packages/modules/customer/integration-tests/__tests__/services/customer-module/index.spec.ts`

```typescript
import { ICustomerModuleService } from "@medusajs/framework/types"
import { Module, Modules } from "@medusajs/framework/utils"
import { moduleIntegrationTestRunner } from "@medusajs/test-utils"

moduleIntegrationTestRunner({
  moduleName: Modules.CUSTOMER,
  testSuite: ({ service }: { service: ICustomerModuleService }) => {
    describe("Customer Module Service", () => {
      it("should create a customer", async () => {
        const customer = await service.createCustomers({
          first_name: "John",
          last_name: "Doe",
          email: "john@example.com",
        })

        expect(customer).toEqual(
          expect.objectContaining({
            id: expect.stringMatching(/^cus_/),
            first_name: "John",
            last_name: "Doe",
            email: "john@example.com",
          })
        )
      })

      it("should list customers with pagination", async () => {
        // Create test data
        await service.createCustomers([
          { email: "customer1@test.com", first_name: "Customer", last_name: "One" },
          { email: "customer2@test.com", first_name: "Customer", last_name: "Two" },
          { email: "customer3@test.com", first_name: "Customer", last_name: "Three" },
        ])

        const [customers, count] = await service.listAndCountCustomers(
          {},
          { take: 2, skip: 0 }
        )

        expect(customers).toHaveLength(2)
        expect(count).toBeGreaterThanOrEqual(3)
      })
    })
  },
})
```

---

## API Testing

API tests verify full HTTP request/response cycles using real endpoints.

### medusaIntegrationTestRunner

For HTTP API tests, use `medusaIntegrationTestRunner`:

```typescript
import { medusaIntegrationTestRunner } from "@medusajs/test-utils"
import { createAdminUser, adminHeaders } from "../helpers/create-admin-user"

jest.setTimeout(30000)

medusaIntegrationTestRunner({
  env: {},
  testSuite: ({ dbConnection, getContainer, api }) => {
    let productType

    beforeEach(async () => {
      const container = getContainer()
      await createAdminUser(dbConnection, adminHeaders, container)

      // Create test data via API
      const response = await api.post(
        "/admin/product-types",
        { value: "electronics" },
        adminHeaders
      )
      productType = response.data.product_type
    })

    describe("GET /admin/product-types", () => {
      it("should return list of product types", async () => {
        const res = await api.get("/admin/product-types", adminHeaders)

        expect(res.status).toEqual(200)
        expect(res.data.product_types).toEqual(
          expect.arrayContaining([
            expect.objectContaining({
              id: expect.stringMatching(/ptyp_.{24}/),
              value: "electronics",
              created_at: expect.any(String),
              updated_at: expect.any(String),
            }),
          ])
        )
      })

      it("should filter by search query", async () => {
        await api.post(
          "/admin/product-types",
          { value: "clothing" },
          adminHeaders
        )

        const res = await api.get(
          "/admin/product-types?q=electr",
          adminHeaders
        )

        expect(res.status).toEqual(200)
        expect(res.data.product_types).toHaveLength(1)
        expect(res.data.product_types[0].value).toBe("electronics")
      })
    })

    describe("GET /admin/product-types/:id", () => {
      it("should return a single product type", async () => {
        const res = await api.get(
          `/admin/product-types/${productType.id}`,
          adminHeaders
        )

        expect(res.status).toEqual(200)
        expect(res.data.product_type).toEqual({
          id: productType.id,
          value: "electronics",
          created_at: expect.any(String),
          updated_at: expect.any(String),
          metadata: null,
        })
      })

      it("should return 404 for non-existent type", async () => {
        const res = await api
          .get("/admin/product-types/ptyp_doesnotexist", adminHeaders)
          .catch(e => e.response)

        expect(res.status).toEqual(404)
      })
    })
  },
})
```

### Testing with Supertest

For lower-level HTTP testing:

```typescript
import request from "supertest"

describe("Product API", () => {
  it("should create a product", async () => {
    const response = await request(app)
      .post("/admin/products")
      .set("Authorization", `Bearer ${adminToken}`)
      .send({
        title: "Test Product",
        handle: "test-product",
        status: "published",
      })
      .expect(200)

    expect(response.body.product).toHaveProperty("id")
    expect(response.body.product.title).toBe("Test Product")
  })
})
```

---

## Frontend Testing

Frontend tests use React Testing Library and Jest.

### Component Testing

```typescript
import { render, screen, fireEvent } from "@testing-library/react"
import { ProductCard } from "./product-card"

describe("ProductCard", () => {
  const mockProduct = {
    id: "prod_123",
    title: "Test Product",
    price: 2999,
    thumbnail: "/image.jpg",
  }

  it("should render product information", () => {
    render(<ProductCard product={mockProduct} />)

    expect(screen.getByText("Test Product")).toBeInTheDocument()
    expect(screen.getByText("$29.99")).toBeInTheDocument()
    expect(screen.getByRole("img")).toHaveAttribute("src", "/image.jpg")
  })

  it("should call onClick when clicked", () => {
    const handleClick = jest.fn()
    render(<ProductCard product={mockProduct} onClick={handleClick} />)

    fireEvent.click(screen.getByRole("button"))

    expect(handleClick).toHaveBeenCalledWith(mockProduct)
  })

  it("should show 'Out of Stock' when inventory is 0", () => {
    render(<ProductCard product={{ ...mockProduct, inventory: 0 }} />)

    expect(screen.getByText("Out of Stock")).toBeInTheDocument()
  })
})
```

### Hook Testing

```typescript
import { renderHook, act } from "@testing-library/react"
import { useCart } from "./use-cart"

describe("useCart", () => {
  it("should add item to cart", () => {
    const { result } = renderHook(() => useCart())

    act(() => {
      result.current.addItem({ id: "prod_123", quantity: 2 })
    })

    expect(result.current.items).toHaveLength(1)
    expect(result.current.items[0]).toEqual({
      id: "prod_123",
      quantity: 2,
    })
  })

  it("should calculate total correctly", () => {
    const { result } = renderHook(() => useCart())

    act(() => {
      result.current.addItem({ id: "prod_1", price: 1000, quantity: 2 })
      result.current.addItem({ id: "prod_2", price: 500, quantity: 1 })
    })

    expect(result.current.total).toBe(2500)
  })
})
```

---

## Test Organization

### Directory Structure

```
packages/modules/product/
├── src/
│   ├── models/
│   ├── services/
│   └── __tests__/              # Unit tests
│       ├── product-service.spec.ts
│       └── utils.spec.ts
└── integration-tests/
    ├── __tests__/              # Integration tests
    │   ├── product-module-service.spec.ts
    │   └── product-variants.spec.ts
    └── __fixtures__/
        └── test-data.ts

integration-tests/
├── http/                       # API tests
│   └── __tests__/
│       └── product/
│           ├── admin/
│           │   └── product.spec.ts
│           └── store/
│               └── product.spec.ts
└── modules/                    # Cross-module tests
    └── __tests__/
```

### File Naming Conventions

- `*.spec.ts` - Primary test file format
- `*.test.ts` - Alternative test file format
- `__fixtures__/` - Test data and mocks
- `__testfixtures__/` - Alternative fixtures directory

---

## Mocking Strategies

### Mocking Awilix Container

```typescript
import { asValue } from "awilix"

describe("OrderService", () => {
  let container
  let orderService

  beforeEach(() => {
    container = createContainer()

    // Mock dependencies
    container.register({
      productService: asValue({
        retrieve: jest.fn().mockResolvedValue(mockProduct),
        list: jest.fn().mockResolvedValue([mockProduct]),
      }),
      eventBusService: asValue({
        emit: jest.fn(),
      }),
      logger: asValue({
        info: jest.fn(),
        error: jest.fn(),
      }),
    })

    orderService = new OrderService(container)
  })

  it("should create order and emit event", async () => {
    const order = await orderService.create({ items: [...] })

    expect(container.cradle.eventBusService.emit).toHaveBeenCalledWith(
      "order.created",
      { id: order.id }
    )
  })
})
```

### Mocking MikroORM Repositories

```typescript
describe("ProductRepository", () => {
  let em
  let productRepo

  beforeEach(() => {
    em = {
      find: jest.fn(),
      findOne: jest.fn(),
      persistAndFlush: jest.fn(),
      removeAndFlush: jest.fn(),
    }

    productRepo = em.getRepository(Product)
  })

  it("should find products by status", async () => {
    em.find.mockResolvedValue([mockProduct])

    const products = await productRepo.find({ status: "published" })

    expect(em.find).toHaveBeenCalledWith(
      Product,
      { status: "published" },
      expect.any(Object)
    )
    expect(products).toHaveLength(1)
  })
})
```

### Mocking External Services

```typescript
import axios from "axios"
import MockAdapter from "axios-mock-adapter"

describe("PaymentProvider", () => {
  let mock

  beforeEach(() => {
    mock = new MockAdapter(axios)
  })

  afterEach(() => {
    mock.restore()
  })

  it("should process payment successfully", async () => {
    mock.onPost("/api/payments").reply(200, {
      id: "charge_123",
      status: "succeeded",
    })

    const result = await paymentProvider.processPayment({
      amount: 5000,
      currency: "usd",
    })

    expect(result.status).toBe("succeeded")
  })

  it("should handle payment failure", async () => {
    mock.onPost("/api/payments").reply(400, {
      error: "Card declined",
    })

    await expect(
      paymentProvider.processPayment({ amount: 5000 })
    ).rejects.toThrow("Card declined")
  })
})
```

### Mocking Event Bus

```typescript
describe("Workflow with Events", () => {
  let eventBus

  beforeEach(() => {
    eventBus = {
      emit: jest.fn(),
      subscribe: jest.fn(),
    }
  })

  it("should emit events during workflow", async () => {
    await executeWorkflow({ eventBus })

    expect(eventBus.emit).toHaveBeenCalledTimes(3)
    expect(eventBus.emit).toHaveBeenNthCalledWith(
      1,
      "workflow.started",
      expect.any(Object)
    )
    expect(eventBus.emit).toHaveBeenNthCalledWith(
      2,
      "workflow.step.completed",
      expect.any(Object)
    )
    expect(eventBus.emit).toHaveBeenNthCalledWith(
      3,
      "workflow.completed",
      expect.any(Object)
    )
  })
})
```

---

## Coverage Expectations

### Coverage Goals

- **Overall**: 70%+ code coverage
- **Critical paths**: 90%+ (payment, checkout, auth)
- **Services**: 80%+
- **Utils**: 85%+
- **Models**: Not required (mostly declarations)

### Measuring Coverage

```bash
# Run tests with coverage
yarn test --coverage

# Generate HTML coverage report
yarn test --coverage --coverageReporters=html

# View coverage report
open coverage/index.html
```

### Coverage Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    "src/**/*.{js,ts}",
    "!src/**/*.d.ts",
    "!src/**/__tests__/**",
    "!src/**/index.ts",
  ],
  coverageThresholds: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
}
```

---

## TDD Workflow

### Test-Driven Development Process

1. **Write failing test first**
2. **Write minimal code to pass**
3. **Refactor while keeping tests green**

```typescript
// 1️⃣ RED: Write failing test
describe("DiscountCalculator", () => {
  it("should apply percentage discount", () => {
    const calculator = new DiscountCalculator()
    const result = calculator.applyDiscount(1000, { type: "percentage", value: 10 })
    expect(result).toBe(900)
  })
})

// 2️⃣ GREEN: Write minimal implementation
class DiscountCalculator {
  applyDiscount(price, discount) {
    if (discount.type === "percentage") {
      return price - (price * discount.value / 100)
    }
    return price
  }
}

// 3️⃣ REFACTOR: Improve code quality
class DiscountCalculator {
  applyDiscount(price: number, discount: Discount): number {
    const calculators = {
      percentage: (p, d) => p - (p * d.value / 100),
      fixed: (p, d) => Math.max(0, p - d.value),
    }

    const calculator = calculators[discount.type]
    return calculator ? calculator(price, discount) : price
  }
}
```

---

## Running Tests

### Run All Tests

```bash
# Run all tests
yarn test

# Run tests in watch mode
yarn test --watch

# Run tests with coverage
yarn test --coverage
```

### Run Specific Tests

```bash
# Run tests in a specific file
yarn jest packages/modules/product/src/__tests__/product-service.spec.ts

# Run tests matching a pattern
yarn jest --testPathPattern=product

# Run a specific test suite
yarn jest --testNamePattern="ProductService"

# Run a single test
yarn jest --testNamePattern="should create a product"
```

### Run Integration Tests

```bash
# Run all integration tests (packages)
yarn test:integration:packages

# Run fast integration tests
yarn test:integration:packages:fast

# Run slow integration tests
yarn test:integration:packages:slow

# Run API integration tests
yarn test:integration:api

# Run HTTP integration tests
yarn test:integration:http

# Run module integration tests
yarn test:integration:modules
```

### Run Tests in CI

```bash
# Run tests in CI mode (no cache, force)
yarn test --no-cache --force

# Run specific chunk of tests
yarn test:chunk
```

### Watch Mode Commands

When in watch mode, you can use:

- `p` - Filter by filename pattern
- `t` - Filter by test name
- `a` - Run all tests
- `f` - Run only failed tests
- `q` - Quit watch mode
- `Enter` - Trigger a test run

### Debugging Tests

```bash
# Run tests with Node debugger
node --inspect-brk node_modules/.bin/jest --runInBand

# Run with increased timeout
yarn jest --testTimeout=30000

# Run with verbose output
yarn jest --verbose

# Show console.log statements
yarn jest --verbose --silent=false
```

---

## Real Examples

### Example 1: Product Type Admin API Test

**Location**: `/home/user/medusa/integration-tests/http/__tests__/product-type/admin/product-type.spec.ts`

```typescript
import { medusaIntegrationTestRunner } from "@medusajs/test-utils"
import {
  createAdminUser,
  adminHeaders,
} from "../../../../helpers/create-admin-user"

jest.setTimeout(30000)

medusaIntegrationTestRunner({
  env: {},
  testSuite: ({ dbConnection, getContainer, api }) => {
    let type1, type2

    beforeEach(async () => {
      const container = getContainer()
      await createAdminUser(dbConnection, adminHeaders, container)

      type1 = (
        await api.post(
          "/admin/product-types",
          { value: "test1" },
          adminHeaders
        )
      ).data.product_type

      type2 = (
        await api.post(
          "/admin/product-types",
          { value: "test2" },
          adminHeaders
        )
      ).data.product_type
    })

    describe("/admin/product-types", () => {
      it("returns a list of product types", async () => {
        const res = await api.get("/admin/product-types", adminHeaders)

        expect(res.status).toEqual(200)
        expect(res.data.product_types).toEqual(
          expect.arrayContaining([
            {
              id: expect.stringMatching(/ptyp_.{24}/),
              value: "test1",
              created_at: expect.any(String),
              updated_at: expect.any(String),
              metadata: null,
            },
            {
              id: expect.stringMatching(/ptyp_.{24}/),
              value: "test2",
              created_at: expect.any(String),
              updated_at: expect.any(String),
              metadata: null,
            },
          ])
        )
      })

      it("returns filtered list by search query", async () => {
        const res = await api.get("/admin/product-types?q=test1", adminHeaders)

        expect(res.status).toEqual(200)
        expect(res.data.product_types).toEqual([
          {
            id: expect.stringMatching(/ptyp_.{24}/),
            value: "test1",
            created_at: expect.any(String),
            updated_at: expect.any(String),
            metadata: null,
          },
        ])
      })
    })

    describe("/admin/product-types/:id", () => {
      it("returns a product type", async () => {
        const res = await api.get(
          `/admin/product-types/${type1.id}`,
          adminHeaders
        )

        expect(res.status).toEqual(200)
        expect(res.data.product_type).toEqual({
          id: expect.stringMatching(/ptyp_.{24}/),
          value: "test1",
          created_at: expect.any(String),
          updated_at: expect.any(String),
          metadata: null,
        })
      })
    })
  },
})
```

### Example 2: Auth Module Integration Test

**Location**: `/home/user/medusa/packages/modules/auth/integration-tests/__tests__/auth-module-service/index.spec.ts`

```typescript
import { IAuthModuleService } from "@medusajs/framework/types"
import { Module, Modules } from "@medusajs/framework/utils"
import { moduleIntegrationTestRunner, SuiteOptions } from "@medusajs/test-utils"

let moduleOptions = {
  providers: [
    {
      resolve: "./integration-tests/__fixtures__/providers/default-provider",
      id: "plaintextpass",
    },
  ],
}

jest.setTimeout(30000)

moduleIntegrationTestRunner({
  moduleName: Modules.AUTH,
  moduleOptions,
  testSuite: ({ service }: SuiteOptions<IAuthModuleService>) =>
    describe("Auth Module Service", () => {
      beforeEach(async () => {
        await service.createAuthIdentities({
          provider_identities: [
            {
              entity_id: "test@admin.com",
              provider: "plaintextpass",
              provider_metadata: {
                password: "plaintext",
              },
            },
          ],
        })
      })

      it("should authenticate with correct password", async () => {
        const result = await service.authenticate("plaintextpass", {
          body: {
            email: "test@admin.com",
            password: "plaintext",
          },
        })

        expect(result).toEqual(
          expect.objectContaining({
            success: true,
            authIdentity: expect.objectContaining({
              id: expect.any(String),
              provider_identities: [
                expect.objectContaining({ entity_id: "test@admin.com" }),
              ],
            }),
          })
        )
      })

      it("should fail if password is incorrect", async () => {
        const result = await service.authenticate("plaintextpass", {
          body: {
            email: "test@admin.com",
            password: "incorrect",
          },
        })

        expect(result).toEqual(
          expect.objectContaining({
            success: false,
            error: "Invalid email or password",
          })
        )
      })

      it("should create new entity if nonexistent", async () => {
        const result = await service.authenticate("plaintextpass", {
          body: {
            email: "new@admin.com",
            password: "newpass",
          },
        })

        const dbAuthIdentity = await service.retrieveAuthIdentity(
          result.authIdentity?.id!,
          { relations: ["provider_identities"] }
        )

        expect(dbAuthIdentity).toEqual(
          expect.objectContaining({
            id: expect.any(String),
            provider_identities: [
              expect.objectContaining({ entity_id: "new@admin.com" }),
            ],
          })
        )
      })
    }),
})
```

---

## Best Practices Checklist

✅ **Before Writing Tests:**
- [ ] Understand what you're testing (behavior, not implementation)
- [ ] Choose the right test type (unit, integration, or API)
- [ ] Set up proper test data and fixtures

✅ **Writing Tests:**
- [ ] Use descriptive test names
- [ ] Follow AAA pattern (Arrange, Act, Assert)
- [ ] Test one thing per test
- [ ] Use proper matchers
- [ ] Clean up after each test

✅ **After Writing Tests:**
- [ ] Ensure tests are independent
- [ ] Verify tests fail when they should
- [ ] Check test performance (< 1s for unit, < 10s for integration)
- [ ] Review code coverage
- [ ] Add tests to CI/CD pipeline

---

## Troubleshooting

### Tests Timeout

```bash
# Increase timeout globally
jest.setTimeout(30000)

# Or per test
it("slow test", async () => {
  jest.setTimeout(60000)
  // ...
}, 60000)
```

### Database Connection Issues

```javascript
// Ensure proper cleanup
afterEach(async () => {
  await dbConnection.close()
})
```

### Flaky Tests

```typescript
// Use waitFor for async operations
import { waitFor } from "@testing-library/react"

await waitFor(() => {
  expect(screen.getByText("Loaded")).toBeInTheDocument()
})
```

### Memory Leaks

```bash
# Run with --detectLeaks
yarn jest --detectLeaks

# Run with --logHeapUsage
yarn jest --logHeapUsage
```

---

## Resources

- [Jest Documentation](https://jestjs.io/)
- [Vitest Documentation](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/react)
- [Supertest Documentation](https://github.com/ladjs/supertest)
- [Testing Best Practices](https://testingjavascript.com/)

---

**Next Steps:**
- [Debugging Guide](./DEBUGGING_GUIDE.md) - Debug failing tests
- [Security Guide](./SECURITY_GUIDE.md) - Test security features
- [API Documentation](./API_DOCUMENTATION.md) - API testing reference
