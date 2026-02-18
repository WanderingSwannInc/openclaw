# Ink - React for Interactive CLIs

This skill guides creation of interactive command-line interfaces using Ink, which brings React's component model to the terminal. Build everything from simple interactive prompts to complex multi-screen CLI applications.

## When to Use This Skill

Use this skill when the user wants to:
- Build interactive terminal UIs with React patterns
- Create CLI tools with forms, menus, progress indicators, or dynamic displays
- Convert existing CLIs to interactive experiences
- Build developer tools, installers, or terminal dashboards
- Use React knowledge to build terminal applications

## Core Concepts

### Ink Fundamentals

**Ink renders React components to the terminal instead of the DOM:**
- Uses Flexbox-like layout with `<Box>` components
- Text rendering with `<Text>` components
- Hooks for input, app lifecycle, and terminal state
- Component composition and state management work like React

**Key differences from React web:**
- Layout is done with Box (flexbox-inspired), not div/CSS
- No mouse events (keyboard-driven)
- Terminal has width/height constraints
- Colors use chalk/hex, not CSS
- Input handling is different (stdin, not DOM events)

### Project Structure

**Simple Ink component (pure UI):**
```
my-component/
├── package.json
├── src/
│   └── App.tsx        # Main Ink component
└── test/
    └── App.test.tsx
```

**Full CLI application:**
```
my-cli/
├── package.json
├── src/
│   ├── cli.tsx              # Entry point with arg parsing
│   ├── components/
│   │   ├── App.tsx          # Main app component
│   │   ├── MenuScreen.tsx   # Individual screens
│   │   ├── FormScreen.tsx
│   │   └── shared/
│   │       ├── Spinner.tsx
│   │       └── ErrorBox.tsx
│   ├── hooks/
│   │   ├── useApiData.ts
│   │   └── useFileSystem.ts
│   └── utils/
│       └── formatting.ts
├── bin/
│   └── my-cli.js            # Executable
└── test/
```

## Implementation Guide

### 1. Setting Up an Ink Project

**Package.json essentials:**
```json
{
  "name": "my-cli",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "my-cli": "./bin/cli.js"
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node bin/cli.js"
  },
  "dependencies": {
    "ink": "^4.0.0",
    "react": "^18.2.0",
    "meow": "^13.0.0",
    "chalk": "^5.3.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "typescript": "^5.0.0",
    "ink-testing-library": "^3.0.0"
  }
}
```

**TypeScript configuration:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "node",
    "jsx": "react",
    "jsxImportSource": "react",
    "outDir": "./bin",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 2. Basic Ink Components

**Core building blocks:**

```tsx
import React from 'react';
import { Box, Text } from 'ink';

// Box - Layout container (like flexbox)
<Box flexDirection="column" padding={1}>
  <Text>Content here</Text>
</Box>

// Text - Render text with styling
<Text color="green" bold>Success!</Text>
<Text dimColor>Hint: press q to quit</Text>

// Common patterns
<Box borderStyle="round" borderColor="cyan" padding={1}>
  <Text>Boxed content</Text>
</Box>

<Box flexDirection="row" justifyContent="space-between">
  <Text>Left</Text>
  <Text>Right</Text>
</Box>
```

### 3. Handling User Input

**useInput hook for keyboard events:**

```tsx
import { useInput } from 'ink';

function Menu({ items, onSelect }) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  useInput((input, key) => {
    if (key.upArrow) {
      setSelectedIndex(prev => Math.max(0, prev - 1));
    }
    if (key.downArrow) {
      setSelectedIndex(prev => Math.min(items.length - 1, prev + 1));
    }
    if (key.return) {
      onSelect(items[selectedIndex]);
    }
    if (input === 'q') {
      process.exit(0);
    }
  });

  return (
    <Box flexDirection="column">
      {items.map((item, i) => (
        <Text key={i} color={i === selectedIndex ? 'cyan' : 'white'}>
          {i === selectedIndex ? '❯ ' : '  '}
          {item}
        </Text>
      ))}
    </Box>
  );
}
```

### 4. App Lifecycle & Control

**useApp hook for controlling the app:**

```tsx
import { useApp } from 'ink';

function MyComponent() {
  const { exit } = useApp();

  const handleComplete = () => {
    // Exit with success
    exit();
  };

  const handleError = () => {
    // Exit with error code
    exit(new Error('Something went wrong'));
  };

  return <Box>{/* ... */}</Box>;
}
```

### 5. Common UI Patterns

**Loading spinner:**
```tsx
import Spinner from 'ink-spinner';

<Box>
  <Text color="green">
    <Spinner type="dots" />
  </Text>
  <Text> Loading...</Text>
</Box>
```

**Progress bar:**
```tsx
function ProgressBar({ current, total }) {
  const percentage = Math.round((current / total) * 100);
  const barLength = 40;
  const filled = Math.round((barLength * current) / total);
  const bar = '█'.repeat(filled) + '░'.repeat(barLength - filled);

  return (
    <Box flexDirection="column">
      <Text>{bar}</Text>
      <Text dimColor>
        {current}/{total} ({percentage}%)
      </Text>
    </Box>
  );
}
```

**Input form:**
```tsx
import TextInput from 'ink-text-input';

function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [step, setStep] = useState(0);

  if (step === 0) {
    return (
      <Box flexDirection="column">
        <Text>What's your name?</Text>
        <TextInput
          value={name}
          onChange={setName}
          onSubmit={() => setStep(1)}
        />
      </Box>
    );
  }

  if (step === 1) {
    return (
      <Box flexDirection="column">
        <Text>What's your email?</Text>
        <TextInput
          value={email}
          onChange={setEmail}
          onSubmit={() => setStep(2)}
        />
      </Box>
    );
  }

  return <Text>Thanks {name}!</Text>;
}
```

### 6. Building Full CLI Applications

**Entry point with argument parsing (using meow):**

```tsx
#!/usr/bin/env node
import React from 'react';
import { render } from 'ink';
import meow from 'meow';
import App from './components/App.js';

const cli = meow(`
  Usage
    $ my-cli [options]

  Options
    --name, -n  Your name
    --verbose   Enable verbose output
    --help      Show this help

  Examples
    $ my-cli --name John
    $ my-cli --verbose
`, {
  importMeta: import.meta,
  flags: {
    name: {
      type: 'string',
      alias: 'n'
    },
    verbose: {
      type: 'boolean',
      default: false
    }
  }
});

render(<App {...cli.flags} />);
```

**Multi-screen navigation:**

```tsx
type Screen = 'menu' | 'settings' | 'processing' | 'results';

function App() {
  const [screen, setScreen] = useState<Screen>('menu');
  const [data, setData] = useState(null);

  switch (screen) {
    case 'menu':
      return <MenuScreen onNavigate={setScreen} />;
    case 'settings':
      return <SettingsScreen onBack={() => setScreen('menu')} />;
    case 'processing':
      return <ProcessingScreen onComplete={(result) => {
        setData(result);
        setScreen('results');
      }} />;
    case 'results':
      return <ResultsScreen data={data} onExit={() => process.exit(0)} />;
  }
}
```

### 7. Advanced Patterns

**Custom hooks for async operations:**

```tsx
function useAsyncTask<T>(task: () => Promise<T>) {
  const [state, setState] = useState<{
    loading: boolean;
    error: Error | null;
    data: T | null;
  }>({
    loading: true,
    error: null,
    data: null
  });

  useEffect(() => {
    let cancelled = false;

    task()
      .then(data => {
        if (!cancelled) {
          setState({ loading: false, error: null, data });
        }
      })
      .catch(error => {
        if (!cancelled) {
          setState({ loading: false, error, data: null });
        }
      });

    return () => {
      cancelled = true;
    };
  }, []);

  return state;
}
```

**File watching with live updates:**

```tsx
import { useEffect, useState } from 'react';
import { watch } from 'fs';

function FileWatcher({ path }) {
  const [lastModified, setLastModified] = useState<Date | null>(null);
  const [changes, setChanges] = useState(0);

  useEffect(() => {
    const watcher = watch(path, (eventType) => {
      setLastModified(new Date());
      setChanges(prev => prev + 1);
    });

    return () => watcher.close();
  }, [path]);

  return (
    <Box flexDirection="column">
      <Text>Watching: {path}</Text>
      <Text dimColor>Changes: {changes}</Text>
      {lastModified && (
        <Text dimColor>Last modified: {lastModified.toLocaleTimeString()}</Text>
      )}
    </Box>
  );
}
```

**Table rendering:**

```tsx
function DataTable({ headers, rows }) {
  const columnWidths = headers.map((header, i) =>
    Math.max(
      header.length,
      ...rows.map(row => String(row[i]).length)
    )
  );

  const renderRow = (cells: string[]) => (
    <Box>
      {cells.map((cell, i) => (
        <Box key={i} width={columnWidths[i] + 2}>
          <Text>{cell}</Text>
        </Box>
      ))}
    </Box>
  );

  return (
    <Box flexDirection="column">
      {renderRow(headers)}
      <Box>
        <Text>{'─'.repeat(columnWidths.reduce((a, b) => a + b + 2, 0))}</Text>
      </Box>
      {rows.map((row, i) => (
        <Box key={i}>{renderRow(row)}</Box>
      ))}
    </Box>
  );
}
```

### 8. Testing Ink Components

**Using ink-testing-library:**

```tsx
import { render } from 'ink-testing-library';
import { expect, test } from 'vitest';
import Menu from './Menu.js';

test('menu renders items', () => {
  const items = ['Option 1', 'Option 2', 'Option 3'];
  const { lastFrame } = render(<Menu items={items} onSelect={() => {}} />);

  expect(lastFrame()).toContain('Option 1');
  expect(lastFrame()).toContain('Option 2');
  expect(lastFrame()).toContain('Option 3');
});

test('menu responds to input', () => {
  const items = ['Option 1', 'Option 2'];
  const onSelect = vi.fn();
  const { stdin, lastFrame } = render(<Menu items={items} onSelect={onSelect} />);

  // Simulate down arrow and enter
  stdin.write('\u001B[B'); // Down arrow
  stdin.write('\r');       // Enter

  expect(onSelect).toHaveBeenCalledWith('Option 2');
});
```

**Testing async behavior:**

```tsx
test('loading spinner appears during async operation', async () => {
  const { lastFrame, waitUntilExit } = render(<AsyncComponent />);

  expect(lastFrame()).toContain('Loading');

  await waitUntilExit();

  expect(lastFrame()).toContain('Complete');
});
```

## Best Practices

### Layout & Design

1. **Use Box for all layout** - Don't rely on newlines, use flexbox principles
2. **Mind the terminal width** - Test on different terminal sizes, use `useStdout()` to get dimensions
3. **Respect flexbox constraints** - Terminal rendering is strict about flexbox rules
4. **Use padding/margin sparingly** - Terminal space is limited

### Performance

1. **Avoid re-renders** - Use useMemo/useCallback for expensive computations
2. **Debounce rapid updates** - Especially for file watching or streams
3. **Clean up effects** - Always return cleanup functions from useEffect
4. **Exit cleanly** - Use `useApp().exit()` instead of `process.exit()` when possible

### User Experience

1. **Show feedback immediately** - Loading states, spinners, progress bars
2. **Make keyboard controls obvious** - Show available commands in dimmed text
3. **Handle errors gracefully** - Show clear error messages, don't crash
4. **Provide help text** - Include `--help` flag with examples
5. **Support common keys** - q to quit, Ctrl+C to cancel, arrows for navigation

### Error Handling

```tsx
function SafeComponent() {
  const [error, setError] = useState<Error | null>(null);

  if (error) {
    return (
      <Box borderStyle="round" borderColor="red" padding={1}>
        <Box flexDirection="column">
          <Text color="red" bold>Error</Text>
          <Text>{error.message}</Text>
          <Text dimColor>Press q to quit</Text>
        </Box>
      </Box>
    );
  }

  // Normal render...
}
```

## Common Gotchas

1. **Static rendering in tests** - Ink renders don't update in tests unless you use `rerender()` or `waitUntilExit()`
2. **Exit codes** - Use `exit(error)` or `exit()`, not `process.exit()`
3. **Input focus** - Only one component can use `useInput` at a time with `isActive`
4. **ANSI escape codes** - Terminal output can include escape codes, strip them for testing
5. **Flexbox differences** - Not all CSS flexbox features work (no gap, limited alignment)

## Architecture Recommendations

### Simple Interactive Tool (< 100 lines)
- Single file with one component
- useInput for interactivity
- Minimal argument parsing (or none)

### Medium CLI (100-500 lines)
- Separate components for UI sections
- Argument parsing with meow/yargs
- Custom hooks for business logic
- Basic error handling

### Complex Application (500+ lines)
- Multiple screens with navigation
- Shared component library
- Custom hooks for data/filesystem/API
- Comprehensive error handling and validation
- Testing suite
- Configuration file support

## Dependencies Reference

**Essential:**
- `ink` - Core framework
- `react` - Peer dependency
- `@types/react` - TypeScript types

**Argument Parsing:**
- `meow` - Simple, modern
- `yargs` - Feature-rich
- `commander` - Popular alternative

**UI Components:**
- `ink-spinner` - Loading spinners
- `ink-text-input` - Text input
- `ink-select-input` - Select menus
- `ink-link` - Clickable links

**Utilities:**
- `chalk` - Terminal colors
- `figures` - Unicode symbols
- `cli-boxes` - Box drawing characters
- `wrap-ansi` - Text wrapping

**Testing:**
- `ink-testing-library` - Component testing
- `vitest` or `jest` - Test runner

## Example: Complete CLI Application

Here's how to structure a real-world CLI tool:

**src/cli.tsx** - Entry point
**src/components/App.tsx** - Main app with navigation
**src/components/MenuScreen.tsx** - Initial menu
**src/components/ProcessScreen.tsx** - Do the work
**src/hooks/useConfig.ts** - Load/save config
**src/utils/validation.ts** - Input validation

The app flow:
1. Parse arguments with meow
2. Load config if exists
3. Render App component
4. Navigate between screens
5. Handle async operations
6. Exit with proper codes

## Quick Start Template

```tsx
#!/usr/bin/env node
import React, { useState } from 'react';
import { render, Box, Text, useInput, useApp } from 'ink';
import meow from 'meow';

const cli = meow(`
  Usage
    $ my-tool [options]

  Options
    --help  Show this help
`);

function App() {
  const { exit } = useApp();
  const [count, setCount] = useState(0);

  useInput((input, key) => {
    if (input === 'q') exit();
    if (key.upArrow) setCount(c => c + 1);
    if (key.downArrow) setCount(c => Math.max(0, c - 1));
  });

  return (
    <Box flexDirection="column" padding={1}>
      <Text>Count: {count}</Text>
      <Text dimColor>↑/↓ to change, q to quit</Text>
    </Box>
  );
}

render(<App />);
```

## Conclusion

Ink brings React's power to the terminal. Focus on:
- Component composition and React patterns
- Keyboard-driven UX
- Clear visual hierarchy with Box/Text
- Proper async handling and error states
- Testing interactive flows

Build CLIs that are a joy to use by leveraging React knowledge in the terminal environment.
