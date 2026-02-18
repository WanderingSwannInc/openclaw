# Ink Skill - React for CLIs

A comprehensive skill for building interactive command-line interfaces using Ink.

## What This Skill Does

This skill helps Claude build terminal UIs using Ink, which brings React's component model to the command line. It covers:

- **Pure Ink UI components** - Menus, forms, progress bars, tables, spinners
- **Full CLI applications** - Argument parsing, multi-screen navigation, file operations
- **Simple to complex** - From single-file scripts to full-featured CLI tools
- **Testing** - How to test Ink components with ink-testing-library
- **Best practices** - Performance, UX, error handling, common gotchas

## Scope

### Both Pure UI and Full CLI Apps
- Ink components that can be embedded
- Complete CLI tools with argument parsing and navigation

### Both Simple and Complex Use Cases
- Simple: Interactive prompts, basic menus (< 100 lines)
- Medium: Multi-component tools with custom hooks (100-500 lines)
- Complex: Multi-screen apps with navigation, config, testing (500+ lines)

### Testing Included
- Guidance on ink-testing-library
- Testing user input and interactions
- Testing async operations and state changes

## Evaluation Test Cases

The skill includes 10 test prompts covering:

1. **Simple menu** - Basic navigation and selection
2. **Progress bar** - Async operations with visual feedback
3. **Multi-step form** - Input validation and flow
4. **CLI with arguments** - Flag parsing and conditional rendering
5. **File watcher** - Live updates and cleanup
6. **Component testing** - Writing tests for Ink components
7. **Dashboard** - Tables and auto-refresh
8. **Error handling** - Loading states and error boundaries
9. **Open-ended guidance** - Helping users plan their CLI
10. **Specific problem** - Ctrl+C handling and cleanup

## Next Steps

To evaluate and improve this skill:

1. **Run evaluations** - Test the skill against the eval prompts
2. **Collect feedback** - See what works well and what needs improvement
3. **Iterate** - Refine the skill based on results
4. **Expand tests** - Add more edge cases and advanced scenarios

## Skill Structure

```
ink-skill/
├── SKILL.md           # Main skill file with all guidance
├── evals/
│   ├── evals.json     # Test prompts and expectations
│   └── files/         # Any files needed for tests
└── README.md          # This file
```

## Key Features of the Skill

- Comprehensive coverage from basics to advanced patterns
- Real code examples throughout
- Architecture recommendations based on complexity
- Testing guidance with concrete examples
- Common gotchas and solutions
- Quick-start template for immediate use
- Dependencies reference with explanations

## Usage

When you want Claude to help with Ink/CLI development, the skill will guide:
- Project setup and configuration
- Component patterns and best practices
- Handling user input and navigation
- Async operations and state management
- Testing strategies
- Debugging common issues
