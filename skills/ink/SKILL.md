---
name: ink
description: Build interactive terminal UIs and CLI apps with Ink (React for CLIs). Use when users ask for menus, forms, progress UI, keyboard input handling, multi-screen terminal flows, or testing Ink components.
---

# Ink

Use this skill to build robust Ink-based CLIs quickly.

## Workflow
1. Confirm use case: simple interactive prompt, multi-screen CLI, or dashboard/tool.
2. Scaffold entrypoint + core component structure.
3. Implement keyboard input with `useInput` and controlled app exit with `useApp`.
4. Add loading, errors, and clear key hints for UX.
5. Add tests for navigation and input behavior.

## Defaults
- Keep layout predictable with `Box` + `Text`.
- Prefer explicit keybindings (`q`, arrows, enter, ctrl+c behavior).
- Use graceful exits and cleanup for async/file-watch flows.

## References
- Full patterns/examples/troubleshooting: `references/ink-playbook.md`
