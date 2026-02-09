---
name: claude-code-usage
description: Claude Code Best Practices - Context management, task strategies, prompt techniques for Solidity development
author: 0xlayerghost
version: 1.0.0
triggers:
  - "claude"
  - "AI"
  - "prompt"
  - "/plan"
  - "/clear"
---

# Claude Code Best Practices

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Context Management

- **One window, one task**: Don't switch between different features in the same conversation
- **Don't compress**: When context is nearly full, use `/clear` to start fresh, re-describe key requirements
- **Clean up promptly**: `/clear` immediately after completing complex tasks to avoid old context interfering
- **Cross-window reference**: Copy and paste key information to new windows when needed

## Task Execution Strategy

| Task Type | Approach |
|-----------|----------|
| Small bug fix (few lines) | Describe the requirement directly, let Claude modify |
| Large feature/refactoring | `/plan` -> Confirm approach -> `/clear` -> Paste plan and execute |
| Code analysis/learning | Let Claude analyze directly, no plan needed |
| Multi-file changes | Must go through `/plan` workflow |

## Prompt Techniques

- **Give specific paths**: "Modify the `_transfer` function in `src/MyToken.sol`" (good)
- **Avoid vague references**: "Modify that transfer function" (bad)
- **Give examples**: Provide specific input/output examples when describing features
- **Set boundaries**: "Only modify this function, don't touch other code"
- **Limit output**: Say "keep it concise" or "show only key parts" if output is too long

## Git Operation Rules

- Run `git diff` before committing to review changes
- Only commit, do not push unless explicitly requested

## Common Commands

| Command | Usage |
|---------|-------|
| `/clear` | Clear context, start new task |
| `/plan` | Enter planning mode, analyze without modifying code |
| `/help` | View all available commands |
| `/compact` | Compress context (not recommended, prefer `/clear`) |

## Project-level Configuration

Create `.claude/instructions.md` in the project root with project rules. Claude will automatically read it at the start of each conversation.
