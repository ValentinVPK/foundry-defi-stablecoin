# Understanding Fuzz Testing in Solidity

## What is Fuzz Testing?

Fuzz testing is a software testing technique that provides random inputs to a program to find potential bugs or vulnerabilities. In Solidity, fuzz testing is particularly useful for finding edge cases and unexpected behaviors in smart contracts.

## Types of Fuzz Testing

### 1. Stateless Fuzz Testing

Stateless fuzz testing is the simpler form where each test run is independent and doesn't maintain any state between runs. It's like testing a function in isolation.

#### Example:

```solidity
function testFuzz_Add(uint256 a, uint256 b) public {
    // Bound the inputs to prevent overflow
    a = bound(a, 0, type(uint128).max);
    b = bound(b, 0, type(uint128).max);

    uint256 result = a + b;
    assert(result >= a && result >= b);
}
```

Key characteristics:

- Each test run is independent
- No state is maintained between runs
- Simpler to write and understand
- Good for testing pure functions
- Faster execution

### 2. Stateful Fuzz Testing (Invariant Testing)

Stateful fuzz testing maintains state between test runs and tests properties that should always hold true regardless of the sequence of operations.

#### Example:

```solidity
contract InvariantTest is StdInvariant {
    DSCEngine engine;
    DecentralizedStableCoin dsc;

    function setUp() public {
        // Setup contracts
        engine = new DSCEngine();
        dsc = new DecentralizedStableCoin();
    }

    function invariant_TotalSupplyLessThanCollateral() public {
        // This invariant should always hold true
        uint256 totalSupply = dsc.totalSupply();
        uint256 totalCollateralValue = engine.getTotalCollateralValue();
        assert(totalCollateralValue >= totalSupply);
    }
}
```

Key characteristics:

- Maintains state between operations
- Tests system-wide properties
- More complex to write
- Better for testing protocol-level invariants
- Slower execution

## Practical Examples from Our Project

### 1. Stateless Fuzz Testing Example

```solidity
function testFuzz_GetUsdValue(uint256 amount) public {
    // Bound the amount to prevent overflow
    amount = bound(amount, 0, type(uint128).max);

    uint256 usdValue = engine.getUsdValue(weth, amount);
    assert(usdValue >= amount); // Basic check that value is at least the amount
}
```

### 2. Stateful Fuzz Testing Example

```solidity
function invariant_ProtocolMustHaveMoreCollateralValueThanTotalSupply() public {
    // This invariant should always hold true regardless of operations
    uint256 totalSupply = dsc.totalSupply();
    uint256 totalWethDeposited = IERC20(weth).balanceOf(address(engine));
    uint256 totalBtcDeposited = IERC20(wbtc).balanceOf(address(engine));

    uint256 wethValue = engine.getUsdValue(weth, totalWethDeposited);
    uint256 btcValue = engine.getUsdValue(wbtc, totalBtcDeposited);

    assert(wethValue + btcValue >= totalSupply);
}
```

## When to Use Each Type

### Use Stateless Fuzz Testing When:

- Testing pure functions
- Testing mathematical operations
- Testing input validation
- Testing individual contract functions
- You need fast test execution

### Use Stateful Fuzz Testing When:

- Testing protocol-level invariants
- Testing complex state transitions
- Testing system-wide properties
- Testing multiple contract interactions
- Testing economic properties

## Best Practices

1. **Input Bounding**

   ```solidity
   // Use bound() to limit input ranges
   amount = bound(amount, 1, MAX_DEPOSIT_SIZE);
   ```

2. **Handler Pattern**

   ```solidity
   contract Handler is Test {
       function depositCollateral(uint256 amount) public {
           amount = bound(amount, 1, MAX_DEPOSIT_SIZE);
           // Perform deposit
       }
   }
   ```

3. **Invariant Naming**

   ```solidity
   function invariant_DescriptiveName() public {
       // Test invariant
   }
   ```

4. **Setup Properly**
   ```solidity
   function setUp() public {
       // Initialize contracts
       // Set up initial state
   }
   ```

## Running Fuzz Tests

```bash
# Run all fuzz tests
forge test --match-test testFuzz

# Run specific invariant test
forge test --match-test invariant_

# Run with more verbosity
forge test -vvv

# Run with specific fuzz runs
forge test --fuzz-runs 1000
```

## Common Pitfalls

1. **Unbounded Inputs**

   - Always bound inputs to prevent overflow
   - Consider realistic ranges for your use case

2. **Missing State Setup**

   - Ensure proper state setup in setUp()
   - Consider edge cases in initial state

3. **Too Complex Invariants**

   - Keep invariants simple and focused
   - Test one property at a time

4. **Ignoring Reverts**
   - Consider whether reverts are expected
   - Use try/catch when appropriate

## The `fail_on_revert` Property

### What is `fail_on_revert`?

The `fail_on_revert` property is a configuration setting in Foundry's `foundry.toml` file that controls how the testing framework handles reverts during fuzz and invariant testing.

```toml
[fuzz]
fail_on_revert = true  # or false
```

### How It Works

- When `fail_on_revert = true` (default):

  - Any revert in a fuzz test causes the test to fail
  - The framework shows the inputs that caused the revert
  - Useful for catching unexpected reverts

- When `fail_on_revert = false`:
  - Reverts are caught and ignored during testing
  - The test continues with different inputs
  - Only explicit assertion failures will fail the test

### When to Use Each Setting

#### Use `fail_on_revert = true` when:

- You want to ensure your functions don't revert with any valid inputs
- You're testing normal operation paths
- You're writing stateless fuzz tests for input validation

#### Use `fail_on_revert = false` when:

- You expect some inputs to cause reverts
- You're testing invariants where some operations might legitimately revert
- You're using handlers that might call functions with invalid combinations of inputs

### Practical Example

Consider a stablecoin system where we're testing deposit and withdrawal operations:

```solidity
// With fail_on_revert = true, this test will fail if any inputs cause a revert
function testFuzz_DepositCollateral(uint256 amount) public {
    amount = bound(amount, 1, MAX_DEPOSIT_SIZE);

    // This will fail the test if it reverts with any amount in the bound range
    engine.depositCollateral(weth, amount);
}

// Handler for invariant testing with fail_on_revert = false
contract Handler is Test {
    function depositCollateral(uint256 amount) public {
        amount = bound(amount, 0, type(uint96).max);

        // With fail_on_revert = false, the test continues even if this reverts
        try engine.depositCollateral(weth, amount) {} catch {}
    }
}
```
