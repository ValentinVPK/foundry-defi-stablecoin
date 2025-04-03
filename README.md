# Decentralized Stablecoin (DSC)

A decentralized stablecoin system that maintains a 1:1 peg with USD through overcollateralization and algorithmic stability mechanisms.

## Overview

This project implements a decentralized stablecoin system similar to DAI, but with a simplified design focused on ETH and BTC collateral. The system ensures stability through overcollateralization and algorithmic minting mechanisms.

### Key Features

1. **Relative Stability (USD Peg)**

   - Maintains 1:1 peg with USD
   - Uses Chainlink Price Feeds for real-time price data
   - Implements stale price checks for security

2. **Stability Mechanism**

   - Algorithmic minting based on collateral value
   - Overcollateralization requirement
   - Liquidation mechanism for maintaining collateral ratio

3. **Collateral Types**

   - Wrapped Ether (WETH)
   - Wrapped Bitcoin (WBTC)

4. **Security Features**
   - Oracle staleness checks
   - Health factor monitoring
   - Liquidation thresholds
   - Reentrancy protection

## Technical Implementation

### Core Components

1. **DSCEngine**

   - Handles collateral deposits and withdrawals
   - Manages DSC minting and burning
   - Implements liquidation mechanism
   - Uses Chainlink price feeds for collateral valuation

2. **DecentralizedStableCoin (DSC)**

   - ERC20 implementation of the stablecoin
   - Burnable functionality for debt management
   - Ownable for administrative functions

3. **OracleLib**
   - Manages Chainlink price feed interactions
   - Implements stale price checks
   - Ensures data freshness (3-hour timeout)

### Testing Strategy

1. **Unit Tests**

   - Core functionality testing
   - Edge case handling
   - Error condition verification

2. **Fuzz Testing**

   - Property-based testing
   - Invariant testing
   - Random input validation
   - Handler-based test scenarios

3. **Integration Tests**
   - End-to-end system testing
   - Cross-contract interactions
   - Price feed integration

### Key Technical Concepts

1. **Price Feed Integration**

   - Chainlink V3 Aggregator interface
   - Stale price detection
   - Price precision handling

2. **Security Measures**

   - ReentrancyGuard implementation
   - Oracle staleness checks
   - Health factor monitoring
   - Liquidation thresholds

3. **Testing Techniques**
   - Fuzz testing with Forge
   - Invariant testing
   - Handler-based test scenarios
   - Mock contracts for testing

## Getting Started

### Prerequisites

- Solidity ^0.8.19
- Foundry
- Chainlink Price Feeds

### Installation

1. Clone the repository
2. Install dependencies:

```bash
forge install
```

### Testing

Run the test suite:

```bash
forge test
```

For verbose output:

```bash
forge test -vv
```

For fuzz testing:

```bash
forge test --match-test testFuzz
```

## Architecture

The system is built with a modular architecture:

1. **Core Contracts**

   - DSCEngine: Main protocol logic
   - DecentralizedStableCoin: Token implementation
   - OracleLib: Price feed management

2. **Testing Infrastructure**

   - Unit tests
   - Fuzz tests
   - Integration tests
   - Mock contracts

3. **Deployment Scripts**
   - Network configuration
   - Contract deployment
   - Initial setup

## Security Considerations

1. **Price Feed Security**

   - Stale price detection
   - Multiple price feed support
   - Price precision handling

2. **Protocol Security**

   - Overcollateralization requirements
   - Liquidation mechanisms
   - Health factor monitoring

3. **Contract Security**
   - Reentrancy protection
   - Access control
   - Input validation

## License

MIT
