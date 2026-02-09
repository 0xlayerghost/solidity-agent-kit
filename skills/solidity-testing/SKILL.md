---
name: solidity-testing
description: Solidity Testing Standards - Test organization, coverage requirements, testing tools for Foundry
author: 0xlayerghost
version: 1.0.0
triggers:
  - "write test"
  - "test"
  - "forge test"
  - "coverage"
  - "fuzz"
globs:
  - "test/**/*.sol"
  - "test/**/*.t.sol"
---

# Testing Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Organization Principles

- **Single Responsibility**: One test contract per module, file naming `<ContractName>.t.sol`
- **Composable**: Tests run independently, support `--match-test` filtering
- **Naming Convention**: Test functions named `test_<feature>_<scenario>`, e.g. `test_transfer_revertsWhenInsufficientBalance`

## Coverage Requirements

Every core function must cover the following scenarios:

- **Happy Path**: Standard input, expected output
- **Permission Checks**: Unauthorized calls should revert
- **Boundary Conditions**: Zero values, max values, overflow boundaries
- **Failure Scenarios**: All require/revert conditions
- **State Changes**: Verify storage variables, balances, and events are correctly updated

## Testing Tools

- Use Foundry cheatcodes: `vm.prank`, `vm.warp`, `vm.expectRevert`, `vm.expectEmit`, etc.
- Use Fuzz testing for critical math and fund flows: `function testFuzz_xxx(uint256 amount) public`
- Run coverage checks regularly: `forge coverage`
- Run gas reports regularly: `forge test --gas-report`

## Common Commands

```bash
# Run all tests
forge test

# Run specific test
forge test --match-test test_transfer

# Run specific contract tests
forge test --match-contract MyTokenTest

# Verbose output (with trace)
forge test -vvvv

# Gas report
forge test --gas-report

# Coverage
forge coverage
```
