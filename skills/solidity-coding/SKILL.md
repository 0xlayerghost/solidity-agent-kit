---
name: solidity-coding
description: Solidity Coding Standards - Coding principles, naming conventions, code organization for Foundry projects
author: 0xlayerghost
version: 1.0.0
triggers:
  - "write contract"
  - "new contract"
  - "solidity"
  - "coding standards"
  - "naming convention"
globs:
  - "src/**/*.sol"
  - "contracts/**/*.sol"
---

# Solidity Coding Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Coding Principles

- **Version**: Use `pragma solidity ^0.8.20;`, keep consistent across the project
- **Dependencies**: OpenZeppelin Contracts 4.9.x, managed via `remappings.txt`
- **Event Indexing**: Only add `indexed` to `address` types by default, comment for exceptions
- **Documentation**: All `public`/`external` functions must have NatSpec comments
- **Error Handling**: Prefer Custom Errors over `require`, use short English messages
- **Keywords**: `immutable`/`constant`/`unchecked`/`assembly` must have comments explaining the reason

## Naming Conventions

- **Contracts/Libraries**: PascalCase (`MyToken`, `StakingPool`)
- **Interfaces**: `I` + PascalCase (`IMyToken`)
- **State Variables/Functions**: lowerCamelCase (`totalSupply`, `claimDividend`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_DAILY_RECYCLE_PERCENT`)
- **Forbidden**: Pinyin, single-letter variables, excessive abbreviations

## Code Organization

- **Constants**: Cross-contract reusable constants go in `src/common/Const.sol`
- **Interface Separation**: Interfaces go in `src/interfaces/`, separated from implementation
- **Tool Selection**: Use `cast` for simple queries, `forge script` for complex flows

## Project Directory Structure

```
src/          - Contract source code (includes interfaces/, common/ subdirectories)
test/         - Test files (*.t.sol)
script/       - Deployment scripts
config/       - Configuration files (*.json)
deployments/  - Deployment records (latest.env)
docs/         - Documentation
lib/          - Dependencies
```

## Configuration Management

- **config/\*.json**: Network config, contract addresses, business parameters, deployment info
- **deployments/latest.env**: Latest deployment addresses, must update after each deployment
- **Change Tracking**: Important config changes must be documented in PR
