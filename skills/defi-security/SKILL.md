---
name: defi-security
description: DeFi Security Principles - Protection mechanisms, launch checklist, emergency response (DeFi projects only)
author: 0xlayerghost
version: 1.0.0
triggers:
  - "DeFi"
  - "liquidity"
  - "LP"
  - "swap"
  - "staking"
  - "dividend"
  - "anti-whale"
  - "reentrancy"
  - "flash loan"
globs:
  - "src/**/*.sol"
---

# DeFi Security Principles

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

> Only applicable to DeFi projects. Non-DeFi projects can ignore this standard.

## Protection Mechanisms

- **Anti-whale**: Daily amount caps + time window limits
- **Anti-arbitrage**: EOA-only checks + referral binding + liquidity distribution + fixed yield + lock period
- **Anti-reentrancy**: All contracts must use `ReentrancyGuard`
- **Anti-overflow**: Solidity 0.8+ has built-in compiler checks
- **Anti-flash loan**: Check `block.number` changes on critical operations or use TWA pricing

## Launch Checklist

- [ ] All `onlyOwner` functions transferred to multisig wallet
- [ ] Timelock configured
- [ ] Emergency pause switch tested (`Pausable`)
- [ ] Daily limit parameters clearly documented
- [ ] Third-party security audit passed (mandatory for mainnet)
- [ ] Testnet running for at least 7 days
- [ ] Critical parameters set reasonably (slippage, fees, lock period)

## Emergency Response

- Define emergency pause procedure: who triggers it, how to trigger, how to recover after pause
- Clearly designate technical lead and contact information
- Prepare community announcement channels
- Prepare fund recovery plan
