---
name: token-antiflash
description: "[AUTO-INVOKE] MUST be invoked when designing or reviewing ERC20 token contracts that need flash loan protection. Covers token-level defense design patterns: cost tracking, same-block cooldown, progressive sell tax, minimum balance retention, EIP-7702 aware address checks, front-run protection, and referral binding. Trigger: any ERC20 token with anti-flash-loan, anti-bot, or tokenomics security design requirements."
---

# Token-Level Flash Loan Prevention Design Patterns

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

> **Scope**: Applicable to ERC20 token contracts that need protection against flash loan attacks at the token contract level. Complements the `defi-security` skill (protocol-level) with token-internal defense mechanisms. All parameters mentioned below are design references — actual values should be determined based on project requirements, tokenomics, and market conditions.

## Workflow Rule

When this skill is triggered, **DO NOT directly implement all strategies**. Follow this workflow:

1. **Assess**: Identify the project's threat model — what type of token (meme, community, DeFi ecosystem), what value at stake, what attack vectors are realistic
2. **Present**: Show the developer the Strategy Decision Matrix and Combination Guide. Clearly explain the trade-offs of each strategy (gas cost, UX friction, implementation complexity)
3. **Let the developer choose**: Ask the developer which protection level (Basic / Standard / Advanced / Maximum) or which specific strategies they want. Do NOT assume a protection level
4. **Confirm parameters**: For each chosen strategy, confirm key design parameters with the developer (e.g., tax tiers, volume limit percentages, cooldown granularity) before writing code
5. **Implement**: Only after developer confirmation, implement the selected strategies with the agreed parameters

**Exception**: If the developer explicitly says "implement all" or "maximum protection", skip steps 2-4 and implement the Maximum combination. If the developer specifies exact strategies by number, skip to step 4 for those strategies.

## Core Design Principle

Flash loan attacks rely on **atomic execution** within a single transaction: borrow → manipulate → profit → repay. Token-level defense breaks this atomicity by introducing **cost**, **time**, and **identity** barriers inside the token contract itself.

## Strategy Decision Matrix

When designing a token with flash loan protection, evaluate which strategies to apply based on the threat model:

| Threat Vector | Recommended Strategies | Priority |
|--------------|----------------------|----------|
| Single-block buy→sell arbitrage | Same-block cooldown + Contract address check | High |
| Zero-cost token exploitation (airdrop/exploit) | Cost tracking + Progressive sell tax | High |
| Large-volume manipulation | Per-address volume limits + Base trade fee | Medium |
| Multi-address Sybil bypass | Referral binding + State propagation on transfer | Medium |
| Bot front-running / sandwich | Front-run sell protection + Dynamic fee | Situational |

## Strategy 1: Cost Tracking

**Design Idea**: Track the acquisition cost basis of tokens per address. Limit realized profit to a configurable multiplier of the cost. Tokens acquired at zero cost (airdrop, exploit, flash loan) have zero cost basis, meaning profit is capped at zero or the tokens are burned on sell.

**Key Design Points**:
- Maintain per-address cost basis state, updated on every `_transfer` / `_update`
- On transfer: **cost follows the token** — propagate cost from sender to receiver proportionally
- Zero-cost tokens should be flagged and handled separately (burn or restrict sell)
- Profit cap multiplier should be a configurable owner parameter with a reasonable upper bound
- Consider gas cost of maintaining the extra mapping — measure impact on transfer gas

**Why It Works Against Flash Loans**: Flash-borrowed tokens inherently have zero cost basis. With profit capped relative to cost, there is no economic incentive to execute the attack.

## Strategy 2: Contract Address Check (EIP-7702 Aware)

**Design Idea**: Block interactions from smart contracts (which flash loan contracts are) while properly handling EIP-7702 delegated EOAs.

**Key Design Points**:
- EIP-7702 delegation designator has a specific code length (23 bytes: `0xef0100` + 20-byte address)
- Allow addresses with this exact code length (they are delegated EOAs, not contracts)
- Block addresses with other non-zero code lengths (regular smart contracts)
- Consider also checking `msg.sender == tx.origin` as a secondary guard
- Whitelist necessary contracts: LP pair, router, staking contracts

**Why It Works Against Flash Loans**: Flash loan execution happens inside a contract call chain. Blocking contract callers forces interaction through EOAs, breaking atomic execution.

## Strategy 3: Same-Block Cooldown

**Design Idea**: Prevent buying and selling (or receiving and selling) within the same block. This is the most direct defense against flash loans since they must complete within one transaction (one block).

**Key Design Points**:
- Track the block number of last incoming transfer per address
- On sell (transfer to LP pair): require that the tracked block number is strictly less than current `block.number`
- **Critical**: Also handle the transfer bypass — prevent `A buys → A transfers to B → B sells` in the same block
- On wallet-to-wallet transfer: propagate the block restriction using `max(lastBlock[to], lastBlock[from])` to prevent state laundering
- **Do NOT use `block.timestamp` for same-block tracking** — `block.number` is the correct metric

**Why It Works Against Flash Loans**: Flash loans execute and repay within a single transaction (single block). Same-block cooldown completely prevents the buy→sell cycle.

## Strategy 4: Per-Address Volume Limits

**Design Idea**: Limit individual trading volume at both per-transaction and per-period levels to reduce the economic scale of any attack.

**Key Design Points**:
- Per-transaction limit: configurable as a percentage of total supply
- Per-period (e.g., daily) cumulative limit per address
- Period tracking via `block.timestamp / period_length` as the mapping key
- Whitelist LP pairs, staking contracts, and operational addresses from limits
- Parameters should be adjustable by owner with defined upper bounds

**Why It Works Against Flash Loans**: Limits the amount an attacker can move in a single transaction. Even if other defenses are bypassed, the damage is capped.

## Strategy 5: Minimum Balance Retention

**Design Idea**: Require that every transfer leaves a small minimum balance in the sender's account rather than allowing full withdrawal.

**Key Design Points**:
- Define a configurable minimum retained balance (small dust amount)
- Enforce in `_update` / `_transfer`: `balanceOf(from) - amount >= minBalance`
- Exception for specific operations: authorized burn, contract migration
- Side benefit: maintains holder count metrics (addresses never go to zero balance)
- Keep the minimum small enough to not affect normal user experience

## Strategy 6: Front-Run Sell Protection

**Design Idea**: When detecting a large sell order, adjust parameters or trigger protective mechanisms to cushion the price impact.

**Key Design Points**:
- Define a threshold (e.g., percentage of LP reserves) that triggers protection
- Options: temporarily increase sell fee, use collected tax for buy-back, adjust LP parameters
- Best implemented at the **business/router layer** rather than inside the token contract
- Consider using Uniswap V4 hooks for native LP integration
- Monitor sell-to-buy ratio per time window; escalate fees when ratio is heavily sell-side
- This strategy is more complex and may not be suitable for all projects

## Strategy 7: Progressive Sell Tax (Tax Ladder)

**Design Idea**: Apply a higher sell tax for recently acquired tokens, with the tax rate decreasing as holding time increases. Encourages holding and penalizes quick flips.

**Key Design Points**:
- Track first-buy timestamp per address
- Define a configurable tax schedule: highest rate for shortest hold, decreasing to a baseline
- Tax is deducted from the sell amount and sent to a designated fee receiver
- **Critical**: On wallet-to-wallet transfer, propagate the timestamp to prevent timestamp laundering (receiver should not get a "fresh" timestamp if the sender has a restriction)
- Tax parameters should be adjustable by owner
- Consider using time tiers (hours/days) rather than continuous formulas for gas efficiency

**Why It Works Against Flash Loans**: Flash-borrowed tokens are held for effectively zero time → highest tax rate applies → eliminates profit margin.

## Strategy 8: Ecosystem Profit Tax

**Design Idea**: Additional sell tax directed toward ecosystem value accrual — such as NFT holder dividends, child-token burns, or other ecosystem contracts.

**Key Design Points**:
- Extra tax on sell (on top of base fee), split between designated ecosystem contracts
- Tax amount can be based on realized profit (requires cost tracking from Strategy 1) or flat rate
- Distribute dividends via **pull pattern (claim)** not push pattern — avoids gas griefing
- Can be combined with Strategy 7 to create a multi-layered tax system
- Ensure tax receiver contracts are verified and immutable (or behind timelock)

## Strategy 9: Base Trade Fee

**Design Idea**: Standard baseline fee on all buy/sell trades, combined with natural AMM slippage, creating a minimum cost floor for any round-trip trade.

**Key Design Points**:
- Apply a configurable fee on both buy and sell directions
- Fee split among: development fund / LP injection / burn (configurable)
- Combined with AMM slippage, establishes a minimum round-trip cost
- **This alone is NOT sufficient flash loan protection** — it's a baseline layer that works in combination with other strategies
- Fee should be reasonable enough not to deter normal trading

## Strategy 10: Referral Binding System

**Design Idea**: Require a referral relationship before an address can trade, creating an identity barrier that anonymous flash loan contracts cannot bypass.

**Key Design Points**:
- New addresses must be "activated" by an existing referrer before first trade
- Referral relationship set once and immutable per address
- Multi-level referral commission from trade fees (configurable depth and rates)
- Flash loan contracts have no referral → cannot trade
- Provides a degree of Sybil resistance (harder to create many activated addresses quickly)
- Consider UX impact — may add friction for legitimate new users

## Combination Guide

Choose strategies based on project needs. More strategies = stronger protection but higher gas cost and implementation complexity:

| Protection Level | Strategies | Suitable For |
|-----------------|------------|-------------|
| Basic | 3 + 4 + 9 | Standard token with basic protection needs |
| Standard | 2 + 3 + 4 + 5 + 7 + 9 | Community token with moderate value at stake |
| Advanced | 1 + 2 + 3 + 4 + 5 + 7 + 8 + 9 | High-value DeFi token with ecosystem |
| Maximum | All strategies | Token with NFT ecosystem, child tokens, and high TVL |

## Implementation Checklist

Before deploying a token with anti-flash-loan protection:

- [ ] All protection state (cost basis, block tracking, timestamps) propagated correctly on ALL transfer paths including `_transfer` / `_update`
- [ ] **Wallet-to-wallet transfers propagate restriction state** — not just pair interactions (this is the most commonly missed vulnerability)
- [ ] Whitelist correctly configured for: LP pair, router, staking contract, deployer multisig
- [ ] All configurable parameters have owner setters with defined upper/lower bounds
- [ ] EIP-7702 delegation designator (23 bytes) handled correctly if using contract address checks
- [ ] Same-block cooldown covers both direct sells AND transfer-then-sell patterns
- [ ] Progressive tax timestamp cannot be reset via transfer to a fresh address
- [ ] All fee/tax collection uses pull pattern (claim) not push pattern
- [ ] Gas impact of extra tracking mappings measured — ensure `_transfer` stays under acceptable gas limits
- [ ] Fuzz testing covers: zero-cost token sell, same-block multi-hop, address cycling, multi-address Sybil bypass attempts
- [ ] TWAP integration considered at the business layer if oracle-dependent pricing is involved (hard to implement in token contract directly — prefer router/business contract)
