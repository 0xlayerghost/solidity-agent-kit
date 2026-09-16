# Complex Function Refactoring

Use this guide when creating or refactoring Solidity functions that coordinate several distinct business stages. The goal is to make the execution flow readable without changing observable behavior or fragmenting straightforward logic.

## When to Apply

Apply this pattern when one or more of the following are true:

- A function performs several independent stages such as validation, accounting, external execution, settlement, refunds, and event emission.
- Security-relevant state transitions are difficult to review because calculations and external calls are interleaved.
- Local variables approach Solidity's stack limit.
- The same calculation or interaction appears in multiple paths.
- The user asks to divide a long function into internal functions or make the business flow easier to understand.

Do not apply it to a short function whose complete behavior is already clear in one place. Additional call boundaries have cognitive and gas costs.

## Preferred Structure

Keep `external` and `public` entry points focused on their public contract:

1. Validate caller-provided inputs.
2. Apply access control, pause, deadline, and reentrancy modifiers.
3. Build any execution parameters needed by the internal flow.
4. Call an internal orchestration function.

The orchestration function should read from top to bottom as a summary of the business process. Move detailed work into narrowly scoped internal functions, for example:

- `_validate...` for preconditions and protocol constraints.
- `_quote...` or `_calculate...` for read-only calculations.
- `_allocate...` or `_split...` for deterministic accounting.
- `_execute...` for external protocol interactions.
- `_settle...`, `_distribute...`, or `_refund...` for final fund movement.
- `_emit...` only when event construction is itself complex; ordinary events should remain near the state change they describe.

Names must describe the business responsibility rather than the implementation mechanism. Avoid vague names such as `_handle`, `_process`, `_part1`, or `_helper`.

## Preserve Behavioral Invariants

Refactoring must not silently change:

- Function selectors, visibility, modifiers, return values, or accepted inputs.
- The order of checks, state writes, external calls, refunds, and events when that order is observable or security-relevant.
- Atomic revert behavior. Do not introduce `try/catch`, partial settlement, or failure swallowing unless explicitly required.
- The meaning of `msg.sender`, `msg.value`, `address(this)`, or delegate-call context.
- Checks-Effects-Interactions ordering and reentrancy boundaries.
- Rounding direction, remainder ownership, token decimal assumptions, or minimum-output calculations.
- Balance baselines used to distinguish assets created by the current operation from assets already held by the contract.
- Approval lifecycle, including clearing temporary token allowances after use when required by the existing design.

Capture these invariants before editing and compare them against the final control flow.

## Managing Intermediate State

Use a memory struct when a flow has many related intermediate values or encounters `stack too deep`. Group values by one execution context rather than creating a general-purpose storage abstraction.

Prefer returning a result struct when several outputs always travel together. Prefer individual return values when there are only one or two independent outputs.

Do not move transient execution data into contract storage solely to avoid stack limits. That changes gas costs, creates stale-state risk, and can alter reentrancy behavior.

## Comments and Layout

Use section headings when a contract contains multiple substantial responsibility groups. Keep headings descriptive, such as configuration, user entry points, settlement, pricing, and query helpers.

Add NatSpec to externally callable functions and to internal functions whose inputs, outputs, failure behavior, or fund ownership are not obvious. Comments should explain:

- Why an ordering constraint exists.
- Which balance or accounting boundary protects existing funds.
- Where rounding remainders go.
- Whether a failure reverts the whole operation or is intentionally isolated.
- Why an external call or approval is safe in its position.

Do not narrate individual assignments or restate the function name. Keep simple private helpers self-explanatory through naming.

## Editing Discipline

Modify existing files with focused in-place patches. Do not delete and recreate an existing file for an ordinary refactor, because doing so obscures the review diff and can discard concurrent changes. Full replacement is appropriate only when explicitly requested or when the file is generated and reproducible.

Keep the refactor separate from unrelated renaming, formatting churn, dependency upgrades, or behavior changes.

## Verification

After refactoring:

1. Run the formatter and confirm the diff contains only intended structural changes.
2. Compile without enabling `via-ir` solely to hide a newly introduced stack problem unless the project already uses it.
3. Run focused tests for the changed flow, then the full test suite.
4. Compare balances, state changes, events, reverts, and external-call ordering with the original behavior.
5. Add tests only where the refactor exposes an untested invariant or introduces a new internal boundary with meaningful risk.
