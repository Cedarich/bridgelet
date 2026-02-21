# Bridgelet Testing Documentation

This document provides comprehensive testing guidelines for the Bridgelet SDK integration, covering testing strategies, test categories, and best practices for contributors.

## Table of Contents

1. [Testing Philosophy](#testing-philosophy)
2. [Test Categories](#test-categories)
3. [Unit Tests](#unit-tests)
4. [Integration Tests](#integration-tests)
5. [End-to-End (E2E) Tests](#end-to-end-e2e-tests)
6. [Manual Testing](#manual-testing)
7. [Testing Against Live Testnet](#testing-against-live-testnet)
8. [Testing with Mock Data](#testing-with-mock-data)
9. [Test Data Requirements](#test-data-requirements)
10. [Release Testing Checklist](#release-testing-checklist)
11. [Troubleshooting Guide](#troubleshooting-guide)
12. [Contributing Tests](#contributing-tests)

## Testing Philosophy

The Bridgelet testing strategy is built on multiple layers of validation to ensure reliability, security, and correctness of the ephemeral account system. Our approach emphasizes:

- **Security First:** All financial interactions are thoroughly tested to prevent vulnerabilities
- **Deterministic Testing:** Tests should be reproducible and not dependent on external state
- **Clear Boundaries:** Each test category has a specific scope and responsibility
- **Coverage Goals:** Aim for >80% code coverage on critical paths (SDK, smart contracts)
- **Representative Data:** Test data should reflect real-world usage patterns

## Test Categories

Bridgelet employs a multi-tiered testing strategy:

```
┌─────────────────────────────────────────────────────────────┐
│                    End-to-End Tests (E2E)                   │
│              Full user flows across all systems              │
└─────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────┐
│             Integration Tests (SDK + Blockchain)             │
│         Tests SDK with real/mock Stellar interactions        │
└─────────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────────┐
│                    Unit Tests (Isolated)                     │
│         Individual functions, methods, and components        │
└─────────────────────────────────────────────────────────────┘
```

### Unit Tests (SDK Repository)

**Scope:** Individual functions and methods in isolation

**Location:** `bridgelet-sdk/src/**/*.spec.ts`

**Coverage Areas:**

- Account creation logic
- Payment processing
- Claim validation
- Cryptographic operations
- Data validation and sanitization
- Error handling paths

**Tools & Frameworks:**

- Jest for test runner
- TypeScript for type safety
- Mocking libraries (ts-mockito, jest.mock)

**Example Test Structure:**

```typescript
describe("AccountService", () => {
  describe("createEphemeralAccount", () => {
    it("should generate a valid Stellar keypair", () => {
      // Test keypair generation
    });

    it("should reject invalid configuration", () => {
      // Test input validation
    });

    it("should handle cryptographic errors gracefully", () => {
      // Test error handling
    });
  });
});
```

### Integration Tests (SDK + Blockchain)

**Scope:** SDK interactions with Stellar blockchain (testnet)

**Location:** `bridgelet-sdk/tests/integration/**`

**Coverage Areas:**

- Account creation on testnet
- Smart contract interactions
- Payment submission and validation
- Fund sweeping operations
- Testnet state transitions
- Transaction confirmation

**Setup Requirements:**

- Testnet node access
- Test account funding
- Smart contract deployment
- Environment variables for testnet RPC

**Example Test Structure:**

```typescript
describe("BlockchainIntegration", () => {
  describe("accountCreationFlow", () => {
    it("should create an ephemeral account on testnet", async () => {
      // Integration test with actual blockchain
    });

    it("should handle failed transactions gracefully", async () => {
      // Test transaction error handling
    });
  });
});
```

### End-to-End Tests (E2E)

**Scope:** Complete user workflows from claim to fund sweep

**Location:** `bridgelet-sdk/tests/e2e/**`

**Coverage Areas:**

- Full payment initialization workflow
- Account claiming process
- Fund distribution and sweep
- Expiration and recovery flows
- Multi-recipient scenarios
- Edge cases and error recovery

**Environment:**

- Testnet for blockchain operations
- Test database with clean state
- Mock payment processor (optional)

**Example Scenarios:**

```typescript
describe("End-to-End Flows", () => {
  describe("Payment and Claim", () => {
    it("should process complete payment-to-claim workflow", async () => {
      // 1. Initialize payment
      // 2. Create ephemeral accounts
      // 3. Simulate claim
      // 4. Verify fund sweep
    });

    it("should handle account expiration correctly", async () => {
      // 1. Create account
      // 2. Simulate expiration
      // 3. Verify fund recovery
    });
  });
});
```

### Manual Testing

**Scope:** User acceptance testing and exploratory testing

**When to Use:**

- Before release candidate creation
- Testing new features requiring user interaction
- Exploratory testing for edge cases
- UI/UX validation
- Manual security review

**Manual Test Scenarios:**

- Account creation and ownership verification
- Claim flow with various wallet types
- Payment settlement timing
- Fund recovery after expiration
- Error message clarity

## Unit Tests

### Writing Unit Tests

1. **Use descriptive names:**

   ```typescript
   it("should return 404 when account does not exist", () => {
     // Implementation
   });
   ```

2. **Follow AAA Pattern (Arrange, Act, Assert):**

   ```typescript
   it("should calculate sweep amount correctly", () => {
     // Arrange
     const balance = 1000;
     const fee = 0.01;

     // Act
     const sweepAmount = calculateSweepAmount(balance, fee);

     // Assert
     expect(sweepAmount).toBe(999.99);
   });
   ```

3. **Mock external dependencies:**
   ```typescript
   it("should log account creation", () => {
     // Mock the logger
     const loggerSpy = jest.spyOn(logger, "info");

     createAccount();

     expect(loggerSpy).toHaveBeenCalledWith("Account created");
   });
   ```

### Running Unit Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run specific test file
npm test -- AccountService.spec.ts

# Run with coverage report
npm test -- --coverage
```

## Integration Tests

### Setting Up Integration Tests

1. **Configure testnet access:**

   ```bash
   # .env.test
   STELLAR_NETWORK=testnet
   STELLAR_RPC_URL=https://soroban-testnet.stellar.org
   TEST_ACCOUNT_SECRET=SBBB...
   ```

2. **Create test fixtures:**

   ```typescript
   // tests/fixtures/tokenTestData.ts
   export const testToken = {
     code: "USDC",
     issuer: "GBNCH7YQ...",
     testAmount: 500,
   };
   ```

3. **Initialize test database:**

   ```typescript
   beforeAll(async () => {
     await initTestDatabase();
     await seedTestData();
   });

   afterAll(async () => {
     await cleanupTestDatabase();
   });
   ```

### Integration Test Patterns

**Testing transactions:**

```typescript
describe("TransactionSubmission", () => {
  it("should submit and confirm transaction on testnet", async () => {
    const txHash = await submitPaymentTransaction({
      recipient: testAccount.publicKey,
      amount: 100,
    });

    const submitted = await waitForConfirmation(txHash, 30000);
    expect(submitted).toBe(true);
  });
});
```

## End-to-End (E2E) Tests

### E2E Test Environment

E2E tests require a complete environment:

```
┌──────────────────────────────────┐
│  Test Client / Web Driver        │ (Puppeteer/Playwright)
└──────────────┬───────────────────┘
               │
┌──────────────▼───────────────────┐
│      Backend SDK (Test Mode)     │ (NestJS on :3001)
└──────────────┬───────────────────┘
               │
┌──────────────▼───────────────────┐
│    Stellar Testnet Blockchain    │
└──────────────────────────────────┘
```

### Running E2E Tests

```bash
# Start the backend in test mode
npm run start:test

# In another terminal, run E2E tests
npm run test:e2e

# Run specific E2E test
npm run test:e2e -- claim-flow.spec.ts
```

### E2E Test Example

```typescript
describe("Claim Flow E2E", () => {
  let browser: Browser;
  let page: Page;

  beforeAll(async () => {
    browser = await puppeteer.launch();
    page = await browser.newPage();
  });

  it("should complete full claim flow", async () => {
    // Navigate to claim page
    await page.goto("http://localhost:3000/claim/abc123");

    // Enter destination wallet
    await page.type('input[name="wallet"]', "GBNCH7YQ...");

    // Click claim button
    await page.click('button[type="submit"]');

    // Wait for success message
    await page.waitForSelector(".success-message");
    const message = await page.$eval(
      ".success-message",
      (el) => el.textContent,
    );
    expect(message).toContain("Funds claimed successfully");
  });

  afterAll(async () => {
    await browser.close();
  });
});
```

## Testing Against Live Testnet

### Prerequisites

1. **Testnet Account Setup:**

   ```bash
   # Fund a testnet account using friendbot
   curl "https://friendbot.stellar.org?addr=YOUR_PUBLIC_KEY"
   ```

2. **Environment Configuration:**

   ```bash
   # .env.testnet
   STELLAR_NETWORK=testnet
   STELLAR_RPC_URL=https://soroban-testnet.stellar.org
   STELLAR_ACCOUNT_SECRET=SBBB...
   TESTNET_FUNDING_AMOUNT=1000
   ```

3. **Smart Contract Deployment:**
   ```bash
   # Deploy contracts to testnet (from bridgelet-core)
   soroban contract deploy --network testnet --source-account <account> ...
   ```

### Testnet Testing Strategy

**Phase 1: Smoke Tests**

- Quick validation that basic operations work
- Account creation
- Simple fund transfers

**Phase 2: Functional Tests**

- Complete workflow tests
- Multiple payment scenarios
- Expiration handling

**Phase 3: Load Tests** (optional)

- Multiple concurrent transactions
- Performance baseline establishment

### Example Testnet Test

```typescript
describe("Testnet Integration", () => {
  const testnetConfig = {
    networkType: "testnet",
    rpcUrl: process.env.STELLAR_RPC_URL,
    accountSecret: process.env.STELLAR_ACCOUNT_SECRET,
  };

  it("should create ephemeral account and verify on testnet", async () => {
    const account = await createEphemeralAccount(testnetConfig);

    // Verify account exists on testnet
    const fetchedAccount = await fetchAccountFromTestnet(account.publicKey);
    expect(fetchedAccount).toBeDefined();
    expect(fetchedAccount.publicKey).toBe(account.publicKey);
  });
});
```

## Testing with Mock Data

### Benefits of Mock Testing

- ❌ No external dependencies
- ✅ Faster test execution
- ✅ Deterministic behavior
- ✅ Easier to reproduce failures
- ✅ Can test error scenarios easily

### Creating Mock Stellar Responses

```typescript
// tests/mocks/stellarMocks.ts
export const mockAccount = {
  account_id: "GBNCH7YQ...",
  balances: [
    {
      balance: "1000.0000000",
      asset_type: "native",
    },
  ],
  sequence: "12345678",
};

export const mockTransaction = {
  hash: "abc123def456",
  ledger: 1234567,
  created_at: "2024-01-01T00:00:00Z",
  successful: true,
};
```

### Using Mock Data in Tests

```typescript
import * as StellarAPI from "@stellar/js-sdk";
jest.mock("@stellar/js-sdk");

describe("AccountService with Mocks", () => {
  beforeEach(() => {
    jest.mocked(StellarAPI.getAccount).mockResolvedValue(mockAccount);
  });

  it("should retrieve account balance from mock", async () => {
    const account = await accountService.getAccount("GBNCH7YQ...");
    expect(account.balance).toBe("1000.0000000");
  });
});
```

### Mock Scenarios

| Scenario             | Mock Setup              | Use Case              |
| -------------------- | ----------------------- | --------------------- |
| **Success Path**     | Normal response         | Happy path validation |
| **Network Error**    | Throw connection error  | Error handling        |
| **Timeout**          | Delay response >timeout | Timeout behavior      |
| **Invalid Response** | Malformed data          | Data validation       |
| **Rate Limit**       | 429 response            | Rate limit handling   |

## Test Data Requirements

### Test Accounts

**Stellar Testnet Accounts** (funded via friendbot):

```typescript
const testAccounts = {
  // Sender account for payments
  payer: {
    publicKey: "GBNCH7YQ...",
    secret: "SBBB...",
    funding: "friendbot",
  },
  // Recipient accounts
  recipient1: {
    publicKey: "GB...",
    wallet: "Lobstr",
  },
  recipient2: {
    publicKey: "GB...",
    wallet: "Stellar Lab",
  },
};
```

### Test Tokens

```typescript
const testTokens = {
  USDC: {
    code: "USDC",
    issuer: "GBUQWP3BOUZX34ULNQG23RQ6F4YUSXHTQSXUSMIQSTBE2GTUXICSTX2",
    decimals: 6,
    testAmount: 500,
  },
  BRL: {
    code: "BRL",
    issuer: "GBBD47UZQ5LMKRMZMXUIWOY5VVU4SCA3YTBAXQKMXCBUOA57PGGPCQR",
    decimals: 2,
    testAmount: 1000,
  },
};
```

### Test Scenarios

```typescript
const testScenarios = {
  smallPayment: { amount: 10, token: "USDC" },
  largePayment: { amount: 10000, token: "USDC" },
  multiToken: { amounts: [100, 50], tokens: ["USDC", "BRL"] },
  expiredAccount: { age: 30, days: "expired" },
  errorScenario: { invalidAddress: true },
};
```

## Release Testing Checklist

Use this checklist before each release:

### Pre-Release Testing (1 week before)

- [ ] All unit tests passing (>80% coverage)
- [ ] All integration tests passing on testnet
- [ ] E2E tests passing in staging environment
- [ ] Code review completed
- [ ] Security audit passed

### Release Candidate (RC1)

- [ ] **Smoke Tests on Testnet**
  - [ ] Create ephemeral account
  - [ ] Submit payment
  - [ ] Claim funds
  - [ ] Verify sweep completion

- [ ] **Functional Tests**
  - [ ] Multiple recipients payment
  - [ ] Fund expiration and recovery
  - [ ] Error handling for invalid addresses
  - [ ] Transaction confirmation flow

- [ ] **Regression Tests**
  - [ ] Previous version workflow still works
  - [ ] No breaking changes to API
  - [ ] Database migrations pass

- [ ] **Performance Tests**
  - [ ] Account creation <2 seconds
  - [ ] Transaction submission <5 seconds
  - [ ] Fund sweep completes within SLA

- [ ] **Security Tests**
  - [ ] Input validation passed
  - [ ] No SQL injection vulnerabilities
  - [ ] Cryptographic operations verified
  - [ ] Authorization checks working

### Final Release Approval

- [ ] All RC testing passed
- [ ] Release notes prepared
- [ ] Documentation updated
- [ ] Deployment runbook reviewed
- [ ] Rollback plan in place

## Troubleshooting Guide

### Common Test Issues and Solutions

#### 1. Testnet Unavailable

**Symptom:** Tests fail with "Cannot connect to testnet RPC"

**Solutions:**

```bash
# Check testnet status
curl https://soroban-testnet.stellar.org/

# Use backup RPC endpoint
STELLAR_RPC_URL=https://soroban-testnet.stellar.org:443

# Run mock tests instead
npm test -- --testPathPattern=mock
```

#### 2. Account Not Funded

**Symptom:** "Insufficient balance for operation" error

**Solutions:**

```bash
# Fund account using friendbot
curl "https://friendbot.stellar.org?addr=YOUR_PUBLIC_KEY"

# Check account balance
stellar account info <account_id>

# Use test fixture with pre-funded account
import { testAccounts } from './fixtures/accounts';
```

#### 3. Transaction Timeout

**Symptom:** Tests hang waiting for transaction confirmation

**Solutions:**

```typescript
// Increase timeout for integration tests
jest.setTimeout(30000); // 30 seconds

// Add debug logging
console.log(`Transaction ${txHash} pending...`);

// Implement retry logic
const confirmed = await retryAsync(() => waitForConfirmation(txHash), {
  maxRetries: 5,
  delayMs: 1000,
});
```

#### 4. Flaky E2E Tests

**Symptom:** Tests pass sometimes but fail randomly

**Solutions:**

```typescript
// Add explicit waits instead of fixed delays
await page.waitForSelector(".confirmation-message", { timeout: 10000 });

// Retry flaky tests
describe("Unreliable Test", () => {
  jest.retryTimes(3);

  it("should eventually pass", async () => {
    // Test implementation
  });
});

// Use stable selectors
// ❌ Bad: class names can change
await page.click(".btn-primary");
// ✅ Good: data attributes
await page.click('[data-testid="claim-button"]');
```

#### 5. Mock Data Out of Sync

**Symptom:** Mock responses don't match real API responses

**Solutions:**

```bash
# Regenerate mocks from actual testnet
npm run mocks:generate:from-testnet

# Update mock snapshots
npm test -- -u

# Version mock data
// tests/mocks/v1.2.0/responses.ts
```

#### 6. Database State Issues

**Symptom:** Tests fail because database wasn't cleaned up

**Solutions:**

```typescript
afterEach(async () => {
  // Clean specific tables
  await db.query("TRUNCATE accounts CASCADE");
});

// Or use database transactions
it("should test isolation", async () => {
  await withTransaction(async (tx) => {
    // All changes roll back after test
  });
});
```

### Debug Techniques

**Enable debug logging:**

```bash
DEBUG=bridgelet:* npm test
```

**Create test report:**

```bash
npm test -- --json --outputFile=test-report.json
```

**Run specific test:**

```bash
npm test -- -t "should handle expired accounts"
```

**Check testnet state:**

```bash
# View account on testnet
https://laboratory.stellar.org/#account-creator?network=testnet

# Check transaction history
curl https://soroban-testnet.stellar.org/accounts/<account-id>
```

## Contributing Tests

### Guidelines for Test Contributions

1. **Write tests for new features:**
   - One unit test per function
   - One integration test for blockchain interactions
   - Full E2E test for user flows

2. **Test error cases:**

   ```typescript
   describe("error handling", () => {
     it("should handle invalid input", () => {});
     it("should handle network errors", () => {});
     it("should handle timeout", () => {});
   });
   ```

3. **Use meaningful names:**

   ```typescript
   // ❌ Bad
   it("works", () => {});

   // ✅ Good
   it("should reject account with duplicate address", () => {});
   ```

4. **Keep tests independent:**

   ```typescript
   // ❌ Bad - test depends on another
   it("test A", () => {});
   it("test B", () => {
     /* needs result from A */
   });

   // ✅ Good - each test is independent
   it("test A", () => {});
   it("test B", () => {
     /* sets up its own data */
   });
   ```

5. **Document complex tests:**
   ```typescript
   it("should handle concurrent claims from same account", () => {
     // BUG: https://github.com/bridgelet/issues/123
     // Race condition when multiple users claim simultaneously
     // This test verifies the fix is working
   });
   ```

### Test Submission Checklist

Before submitting a PR with tests:

- [ ] All new tests pass locally
- [ ] Tests follow project conventions
- [ ] Coverage improved (no decrease)
- [ ] No hardcoded test data (use fixtures)
- [ ] Tests don't require manual setup
- [ ] Error messages are helpful
- [ ] Documentation updated

## Additional Resources

- [Bridgelet SDK Repository](https://github.com/bridgelet-org/bridgelet-sdk) - Test examples
- [Stellar Testing Documentation](https://developers.stellar.org/learn/fundamentals/stellar-ubiquitous-platform)
- [Jest Documentation](https://jestjs.io/)
- [Soroban Testing Guide](https://soroban.stellar.org/docs/learn/testing)

## Getting Help

- **Test Issues:** Create an issue with `[test]` label
- **Questions:** Post in Discussions
- **Security:** See [SECURITY.md](./SECURITY.md) for responsible disclosure

---

**Last Updated:** February 2026
**Maintained By:** Bridgelet Core Team
