---
name: claude-code-usage
description: "[AUTO-INVOKE] MUST be invoked at the START of each new coding session. Covers context management, task strategies, and Foundry-specific workflows. Trigger: beginning of any new conversation or coding session in a Solidity/Foundry project."
---

# Claude Code Best Practices

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Context Management Rules

| Rule | Why |
|------|-----|
| One window = one task | Mixing tasks pollutes context and degrades output quality |
| Use `/clear` over `/compact` | Clean start is more reliable than compressed context |
| `/clear` after complex tasks | Prevents old context from interfering with new work |
| Copy key info to new windows | Don't rely on context persistence — paste critical details |

## Task Execution Strategy

| Task Type | Recommended Approach |
|-----------|---------------------|
| Small bug fix (few lines) | Describe directly, let Claude modify in-place |
| Large feature / refactor | `/plan` → review approach → `/clear` → paste plan → execute step by step |
| Multi-file changes | Must use `/plan` workflow — never modify multiple files without a plan |
| Code analysis / learning | Ask Claude to analyze directly — no plan needed |
| Debugging | Provide error message + file path + relevant code — ask for root cause |

## Prompt Techniques

### Do This

- Give specific paths: *"Modify the `_transfer` function in `src/MyToken.sol`"*
- Give examples: *"Input: 100 tokens, Expected: 95 tokens after 5% fee"*
- Set boundaries: *"Only modify this function, don't touch other code"*
- Reference tests: *"The fix should make `test_transfer_feeDeduction` pass"*

### Avoid This

- Vague references: *"Modify that transfer function"* — which one? Where?
- Open-ended requests without constraints: *"Make it better"*
- Multiple unrelated tasks in one message

## Git Operation Rules

- Always run `git diff` before committing to review changes
- Only commit — do not push unless explicitly requested
- Never push directly to main/master branch
- Stage specific files — never use `git add .` in Solidity projects (risk of committing `.env`)

## Foundry-Specific Workflow

| Action | Command |
|--------|---------|
| Before committing | `forge fmt && forge test` |
| After modifying contracts | `forge build` to check compilation |
| Before PR | `forge test --gas-report` to check gas impact |
| Debugging failed test | `forge test --match-test <name> -vvvv` for full trace |

## Quick Command Reference

| Command | Purpose |
|---------|---------|
| `/clear` | Clear context, start fresh |
| `/plan` | Enter planning mode — analyze before modifying |
| `/help` | View all available commands |
| `/compact` | Compress context (prefer `/clear` instead) |

## Project-level Configuration

Create `.claude/instructions.md` in the project root with project-specific rules. Claude automatically reads it at the start of every conversation — no manual loading needed.
