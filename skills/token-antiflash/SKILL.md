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

## Version Compatibility

| OpenZeppelin Version | Hook to Override | Notes |
|---------------------|-----------------|-------|
| OZ 5.x (Solidity ≥0.8.20) | `_update(address from, address to, uint256 amount)` | State already updated when hook runs |
| OZ 4.x (Solidity <0.8.20) | `_beforeTokenTransfer(address from, address to, uint256 amount)` | State NOT yet updated; reads are pre-transfer |

```solidity
// OZ 5.x — preferred for new projects
function _update(address from, address to, uint256 amount) internal override {
    super._update(from, to, amount);
    // ... defense logic here
}

// OZ 4.x — for existing projects
function _beforeTokenTransfer(address from, address to, uint256 amount) internal override {
    super._beforeTokenTransfer(from, to, amount);
    // ... defense logic here (balances not yet changed)
}
```

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

**Implementation**:

```solidity
struct CostBasis {
    uint256 totalCost;    // total acquisition cost in quote token units
    uint256 totalAmount;  // total token amount held
}

mapping(address => CostBasis) private _costBasis;

function _propagateCostBasis(address from, address to, uint256 amount) private {
    CostBasis storage senderCost = _costBasis[from];
    if (senderCost.totalAmount == 0) return;

    // proportional cost transfer
    uint256 transferredCost = (senderCost.totalCost * amount) / senderCost.totalAmount;

    senderCost.totalCost -= transferredCost;
    senderCost.totalAmount -= amount;

    _costBasis[to].totalCost += transferredCost;
    _costBasis[to].totalAmount += amount;
}

// call this when a buy is detected (from == AMM pair)
function _recordBuy(address buyer, uint256 amount, uint256 costInQuote) private {
    _costBasis[buyer].totalCost += costInQuote;
    _costBasis[buyer].totalAmount += amount;
}
```

## Strategy 2: Contract Address Check (EIP-7702 Aware)

**Design Idea**: Block interactions from smart contracts (which flash loan contracts are) while properly handling EIP-7702 delegated EOAs.

**Key Design Points**:
- EIP-7702 delegation designator has a specific code length (23 bytes: `0xef0100` + 20-byte address)
- Allow addresses with this exact code length (they are delegated EOAs, not contracts)
- Block addresses with other non-zero code lengths (regular smart contracts)
- Consider also checking `msg.sender == tx.origin` as a secondary guard
- Whitelist necessary contracts: LP pair, router, staking contracts

**Why It Works Against Flash Loans**: Flash loan execution happens inside a contract call chain. Blocking contract callers forces interaction through EOAs, breaking atomic execution.

**Implementation**:

```solidity
// EIP-7702 delegation designator format:
//   byte[0..2] = 0xef0100  (magic prefix, 3 bytes)
//   byte[3..22] = target contract address (20 bytes)
//   total: 23 bytes
// Plain EOA has extcodesize = 0.
// Regular contract has extcodesize > 0 (and != 23 for the magic prefix case).
bytes3 private constant EIP7702_MAGIC = 0xef0100;

/// @dev Classifies an address into three types:
///   - size == 0         → plain EOA → return false (allow)
///   - size == 23 AND prefix matches 0xef0100 → EIP-7702 delegated EOA → return false (allow)
///   - anything else     → real deployed contract → return true (block)
///
/// EIP-7702 delegated EOAs are allowed because:
///   1. They are user-controlled accounts (smart wallets), not attack contracts
///   2. They cannot execute atomic flash loan buy→sell (Strategy 3 still blocks same-block ops)
///   3. Blocking them would break smart wallet UX for legitimate holders
///
/// extcodecopy gas note: ~700 gas cold read. Only runs when size == 23, which is rare.
function _isRealContract(address account) internal view returns (bool) {
    uint256 size;
    assembly { size := extcodesize(account) }

    // fast path: most addresses are plain EOA or regular contract
    if (size == 0) return false;   // plain EOA → allow
    if (size != 23) return true;   // regular contract (size > 23 or 1..22) → block

    // slow path: exactly 23 bytes → read prefix to distinguish 7702 from an exotic 23-byte contract
    bytes3 prefix;
    assembly {
        // extcodecopy(addr, destMemPtr, srcOffset, length)
        // write 3 bytes starting at offset 0 of the account's code into `prefix`
        extcodecopy(account, prefix, 0, 3)
    }
    if (prefix == EIP7702_MAGIC) return false; // confirmed EIP-7702 delegated EOA → allow
    return true;                               // exotic 23-byte contract → block
}

/// @dev Call at the top of _update (or _beforeTokenTransfer) to block contract interactions.
///      Whitelisted addresses (LP pair, router, staking) bypass this check entirely.
function _checkNotContract(address addr) internal view {
    if (_isWhitelisted[addr]) return;
    require(!_isRealContract(addr), "AntiFlash: contract caller blocked");
}
```

## Strategy 3: Same-Block Cooldown

**Design Idea**: Prevent buying and selling (or receiving and selling) within the same block. This is the most direct defense against flash loans since they must complete within one transaction (one block).

**Key Design Points**:
- Track the block number of last incoming transfer per address
- On sell (transfer to LP pair): require that the tracked block number is strictly less than current `block.number`
- **Critical**: Also handle the transfer bypass — prevent `A buys → A transfers to B → B sells` in the same block
- On wallet-to-wallet transfer: propagate the block restriction using `max(lastBlock[to], lastBlock[from])` to prevent state laundering
- **Do NOT use `block.timestamp` for same-block tracking** — `block.number` is the correct metric

**Why It Works Against Flash Loans**: Flash loans execute and repay within a single transaction (single block). Same-block cooldown completely prevents the buy→sell cycle.

### `_update` Integration Hub

All strategy state propagation must be wired together in a single `_update` override. **This is the most commonly missed vulnerability** — partial wiring leaves transfer bypass paths open.

```solidity
function _update(address from, address to, uint256 amount) internal override {
    // ── Guard: call super first so balances are updated before our checks ──
    super._update(from, to, amount);

    bool fromPair = _isAMMPair[from]; // buy: tokens flow from pair → user
    bool toPair   = _isAMMPair[to];   // sell: tokens flow from user → pair
    bool isTransfer = !fromPair && !toPair && from != address(0) && to != address(0);

    // ── Strategy 2: Block real contracts ──
    // Only check non-whitelisted, non-pair addresses
    if (from != address(0) && !fromPair) {
        _checkNotContract(from);
    }

    // ── Strategy 3: Same-block cooldown ──
    if (toPair && from != address(0)) {
        // sell path: seller must not have received tokens this block
        require(
            block.number > _lastInteractionBlock[from],
            "AntiFlash: same-block sell"
        );
    }
    if (fromPair) {
        // buy path: record the block so this address cannot sell same-block
        _lastInteractionBlock[to] = block.number;
    }
    if (isTransfer) {
        // ★ transfer path: propagate the stricter (larger) block number
        // using max() ensures restriction only extends, never shortens
        // prevents: buy → transfer to fresh address → sell same-block
        if (_lastInteractionBlock[from] > _lastInteractionBlock[to]) {
            _lastInteractionBlock[to] = _lastInteractionBlock[from];
        }
    }

    // ── Strategy 1: Cost basis propagation ──
    if (useCostTracking && isTransfer) {
        _propagateCostBasis(from, to, amount);
    }
    if (useCostTracking && fromPair) {
        // record buy cost using current price from oracle or passed-in value
        // (exact cost recording depends on your price feed integration)
        _recordBuy(to, amount, _getCurrentCostBasis(amount));
    }

    // ── Strategy 7: Progressive tax timestamp propagation ──
    if (useProgressiveTax && isTransfer) {
        // propagate the older (more restrictive) timestamp so receiver
        // cannot launder holding time by receiving a fresh transfer
        if (_firstBuyTimestamp[from] > 0) {
            if (_firstBuyTimestamp[to] == 0 || _firstBuyTimestamp[from] > _firstBuyTimestamp[to]) {
                _firstBuyTimestamp[to] = _firstBuyTimestamp[from];
            }
        }
    }
    if (useProgressiveTax && fromPair && _firstBuyTimestamp[to] == 0) {
        _firstBuyTimestamp[to] = block.timestamp;
    }

    // ── Strategy 4: Per-address daily volume ──
    if (useDailyLimit && !fromPair && from != address(0)) {
        uint256 today = block.timestamp / 1 days;
        _dailyVolume[from][today] += amount;
        require(_dailyVolume[from][today] <= _dailyLimit, "AntiFlash: daily limit");
    }
}
```

> **Key rule for all state propagation**: Use `max()` semantics — restrictions only extend, never shorten. Direct assignment (`state[to] = state[from]`) is exploitable: a sender with shorter cooldown can overwrite a receiver's longer cooldown.

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

**Implementation**:

```solidity
struct TaxTier {
    uint32 maxAge;  // max holding time in seconds for this tier to apply
    uint16 taxBps;  // tax rate in basis points (e.g. 2000 = 20%)
}

TaxTier[] public sellTaxSchedule;

/// @dev Returns the sell tax rate (bps) for `seller` based on their holding time.
///      Tiers must be ordered ascending by maxAge. Returns BASE_TAX_BPS if all tiers exceeded.
function _getSellTaxRate(address seller) internal view returns (uint256) {
    uint256 holdTime = block.timestamp - _firstBuyTimestamp[seller];
    uint256 len = sellTaxSchedule.length;
    for (uint256 i; i < len; ) {
        if (holdTime < sellTaxSchedule[i].maxAge) return sellTaxSchedule[i].taxBps;
        unchecked { ++i; }
    }
    return BASE_TAX_BPS; // long-term holder baseline rate
}

/// @dev Example schedule — owner should configure via setter with an upper-bound guard
function _initDefaultTaxSchedule() internal {
    sellTaxSchedule.push(TaxTier(1 hours,  4000)); // < 1h  → 40%
    sellTaxSchedule.push(TaxTier(1 days,   2500)); // < 1d  → 25%
    sellTaxSchedule.push(TaxTier(7 days,   1500)); // < 7d  → 15%
    sellTaxSchedule.push(TaxTier(30 days,   500)); // < 30d → 5%
    // > 30d → BASE_TAX_BPS
}

/// @dev Owner setter — enforce upper bound to prevent rug via tax
function setSellTaxSchedule(TaxTier[] calldata tiers) external onlyOwner {
    for (uint256 i; i < tiers.length; ) {
        require(tiers[i].taxBps <= 5000, "AntiFlash: tax exceeds 50%");
        unchecked { ++i; }
    }
    delete sellTaxSchedule;
    for (uint256 i; i < tiers.length; ) {
        sellTaxSchedule.push(tiers[i]);
        unchecked { ++i; }
    }
}
```

> **Gas note**: Linear scan is fine for ≤8 tiers. If you need more tiers, replace with a binary search or hardcoded `if-else` chain.

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

## Gas Optimization: Struct Packing

When multiple strategies are active, each separate `mapping(address => uint256)` costs one `SLOAD`/`SSTORE` per field per transfer. Pack all per-address state into one struct to reduce to a single slot read/write.

```solidity
// ❌ Scattered storage — up to 4 separate SLOAD/SSTORE per transfer
mapping(address => uint256) private _lastInteractionBlock; // slot A
mapping(address => uint256) private _firstBuyTimestamp;    // slot B
mapping(address => uint256) private _dailyVolume;          // slot C
mapping(address => uint256) private _cooldownUntil;        // slot D

// ✅ Packed storage — 1 SLOAD + 1 SSTORE covers all four fields
// uint64 block number: safe until block ~1.8×10^19 (far future)
// uint64 timestamp:    safe until year ~584 billion
// uint96 daily volume: up to ~79 billion tokens with 18 decimals
struct UserState {
    uint64  lastBlock;       // Strategy 3: last interaction block
    uint64  firstBuyTime;    // Strategy 7: first buy timestamp
    uint96  dailyVolume;     // Strategy 4: today's traded volume
    uint32  cooldownUntil;   // optional: unix timestamp cooldown end
    // total: 8+8+12+4 = 32 bytes → exactly 1 storage slot
}

mapping(address => UserState) private _userState;

// Example read — one SLOAD fetches all fields
function _getSellTaxRatePacked(address seller) internal view returns (uint256) {
    uint256 holdTime = block.timestamp - uint256(_userState[seller].firstBuyTime);
    // ... same tier lookup as Strategy 7
}

// Example write — one SSTORE updates all touched fields
function _updateOnBuy(address buyer) internal {
    UserState storage s = _userState[buyer];
    s.lastBlock    = uint64(block.number);
    s.firstBuyTime = s.firstBuyTime == 0 ? uint64(block.timestamp) : s.firstBuyTime;
    // dailyVolume and cooldownUntil updated separately as needed
}
```

> **When to use**: Enable struct packing when 3+ strategies are active and transfer gas is a concern. Measure before and after — the saving is typically 10,000–40,000 gas per transfer depending on cold/warm slot status.

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
