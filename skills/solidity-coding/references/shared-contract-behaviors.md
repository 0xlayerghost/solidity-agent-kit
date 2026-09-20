# Shared Contract Behaviors

Read this reference when two or more contracts need the same owner-only administrative behavior.

## Canonical Asset Recovery

The project treats `owner()` as the highest authority. The owner may withdraw any ERC20 token or native currency held by a contract. Do not add protected-token lists, surplus accounting, or asset-specific restrictions unless the developer explicitly changes this policy.

Use this ABI everywhere:

```solidity
function emergencyWithdraw(address _token, uint256 _amount) external;
```

Canonical definitions:

```solidity
error InvalidAmount();
error WithdrawalFailed();

event EmergencyWithdrawal(
    address indexed token,
    address indexed recipient,
    uint256 amount
);
```

Behavioral invariants:

- `_amount == 0` reverts with `InvalidAmount()`.
- `_token == address(0)` represents native currency.
- ERC20 transfers use `SafeERC20.safeTransfer`.
- Native currency transfers use `call`; a failed call reverts with `WithdrawalFailed()`.
- Resolve `owner()` when the function executes and send the asset to that address.
- Apply `onlyOwner` and `nonReentrant`.
- Emit `EmergencyWithdrawal` only after a successful transfer.
- Do not introduce aliases such as `rescueToken`, `recoverAsset`, or `withdrawStuckToken`.

## Shared Base Contract

Create one `src/abstract/AssetRescue.sol`. Before generating it, use the project's selected ownership model:

- For `Ownable`, import and inherit `Ownable`.
- For `Ownable2Step`, import and inherit `Ownable2Step`.

Generate the selected import and base class directly. Do not leave placeholders or require the developer to edit the generated contract afterward.

The following shows the `Ownable` variant; generate the same contract with `Ownable2Step` when that is the recorded project choice:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {ReentrancyGuard} from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

abstract contract AssetRescue is Ownable, ReentrancyGuard {
    using SafeERC20 for IERC20;

    error InvalidAmount();
    error WithdrawalFailed();

    event EmergencyWithdrawal(
        address indexed token,
        address indexed recipient,
        uint256 amount
    );

    /// @notice Owner withdraws assets held by this contract.
    /// @param _token Asset address; the zero address represents native currency.
    /// @param _amount Amount in the asset's smallest unit; must be non-zero.
    function emergencyWithdraw(
        address _token,
        uint256 _amount
    ) external onlyOwner nonReentrant {
        if (_amount == 0) revert InvalidAmount();

        address recipient = owner();
        if (_token == address(0)) {
            (bool success,) = payable(recipient).call{value: _amount}("");
            if (!success) revert WithdrawalFailed();
        } else {
            IERC20(_token).safeTransfer(recipient, _amount);
        }

        emit EmergencyWithdrawal(_token, recipient, _amount);
    }
}
```

Contracts using asset recovery inherit `AssetRescue` and MUST NOT separately inherit the selected ownership contract or `ReentrancyGuard`; both are already inherited through the shared base. Other owner-only functions in the business contract can use the inherited `onlyOwner` modifier, and other protected functions can use the inherited `nonReentrant` modifier.

If the project already has an ownership base contract, integrate the recovery function into that base or make `AssetRescue` inherit it. Do not create parallel ownership storage or a second authority model.
