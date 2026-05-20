# Common React APIs

- [Common React APIs](#common-react-apis)
  - [1. `lazy()`: The Bundle Splitter](#1-lazy-the-bundle-splitter)
    - [The Core Concept (Why, What, How)](#the-core-concept-why-what-how)
    - [Usage Rules](#usage-rules)
    - [Code Example & Gotchas](#code-example-gotchas)
    - [Mastery & Interview Execution](#mastery-interview-execution)
  - [2. `memo()`: The Render Shield](#2-memo-the-render-shield)
    - [The Core Concept (Why, What, How)](#the-core-concept-why-what-how)
    - [Usage Rules](#usage-rules)
    - [Code Example & Gotchas](#code-example-gotchas)
    - [Mastery & Interview Execution](#mastery-interview-execution)
  - [3. `createContext()`: The Data Wormhole](#3-createcontext-the-data-wormhole)
    - [The Core Concept (Why, What, How)](#the-core-concept-why-what-how)
    - [Usage Rules](#usage-rules)
    - [Code Example & Gotchas](#code-example-gotchas)
  - [4. `act()` (with React Testing Library)](#4-act-with-react-testing-library)
    - [The Core Concept (Why, What, How)](#the-core-concept-why-what-how)
    - [Usage Rules](#usage-rules)
    - [Code Example & Gotchas](#code-example-gotchas)
  - [5. `startTransition()`](#5-starttransition)
    - [The Core Concept (Why, What, How)](#the-core-concept-why-what-how)
    - [Usage Rules](#usage-rules)
    - [Code Example & Gotchas](#code-example-gotchas)
    - [Mastery & Interview Execution](#mastery-interview-execution)

To master React at a senior level, you must understand the core APIs that govern bundle size, render optimization, state propagation, testing boundaries, and concurrent scheduling.

Here is the definitive, architectural deep-dive into these five critical React APIs.

## 1. `lazy()`: The Bundle Splitter

**Introduced:** React 16.6 (2018)

### The Core Concept (Why, What, How)

In a standard React application, Webpack or Vite bundles every single component into one massive JavaScript file. If you have a heavy 3D Chart component on a route the user never visits, they still pay the network cost to download it.

`React.lazy()` allows you to defer loading a component's code until it is actually rendered on the screen. It relies on dynamic `import()`, which tells your bundler to split that code into a separate "chunk."

```text
  WITHOUT LAZY (The Monolith)           WITH LAZY (Chunking)
  +-------------------------+           +------------------+
  | bundle.js (5MB)         |           | main.js (2MB)    |
  | - App                   |           | - App            |
  | - Dashboard             |           | - Dashboard      |
  | - HeavyChart            |           +------------------+
  | - AdminPanel            |                   | (Downloads only when needed)
  +-------------------------+                   v
                                        +------------------+  +------------------+
                                        | chunk1.js (2MB)  |  | chunk2.js (1MB)  |
                                        | - HeavyChart     |  | - AdminPanel     |
                                        +------------------+  +------------------+

```

### Usage Rules

| ✅ When to Use | ❌ When to Avoid |
| --- | --- |
| **Route-level splitting:** Every distinct page in a router should be lazy-loaded. | **Above-the-fold content:** Never lazy-load the hero image or the main navbar. |
| **Heavy UI hidden behind clicks:** Modals, complex charts, or rich text editors. | **Tiny components:** The network overhead of fetching a 2kb chunk isn't worth it. |

### Code Example & Gotchas

To use `lazy()`, you MUST wrap it in a `<Suspense>` boundary. If React encounters a lazy component that hasn't downloaded yet, it "suspends" rendering and shows the fallback.

```javascript
import { lazy, Suspense, useState } from 'react';

// 1. Define the lazy import outside the component
const HeavyChart = lazy(() => import('./HeavyChart'));

export default function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Load Chart</button>
      
      {/* 2. The Suspense boundary catches the loading state */}
      {showChart && (
        <Suspense fallback={<div className="spinner">Downloading...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}

```

**Gotcha #1: Named Exports.** `lazy()` historically only supports default exports. If you have `export const Chart`, it fails.
*Fix:* `const Chart = lazy(() => import('./Charts').then(module => ({ default: module.Chart })));`

### Mastery & Interview Execution

**Machine Coding Question:** "What happens if a user's network drops exactly when they click to load a lazy component? Write a resilient wrapper."

**The Solution:** You must wrap the dynamic import in a retry mechanism.

```javascript
// Retries the import 3 times before failing
function retryImport(fn, retriesLeft = 3, interval = 1000) {
  return new Promise((resolve, reject) => {
    fn()
      .then(resolve)
      .catch((error) => {
        if (retriesLeft === 0) {
          reject(error);
          return;
        }
        setTimeout(() => {
          retryImport(fn, retriesLeft - 1, interval).then(resolve, reject);
        }, interval);
      });
  });
}

const ResilientChart = lazy(() => retryImport(() => import('./HeavyChart')));

```

## 2. `memo()`: The Render Shield

**Introduced:** React 16.6 (2018)

### The Core Concept (Why, What, How)

By default, when a React component re-renders, **all of its children re-render**, regardless of whether their props changed.

`React.memo()` is a Higher-Order Component (HOC). It wraps a functional component and tells React: *"Before you re-render this child, check its props. If the props are mathematically identical to the last render, skip this component and reuse the old UI."*

```text
  +-------------------+
  | Parent Component  |  (State changes)
  +-------------------+
            |
            v
  +-------------------+
  | React.memo()      | ---> Performs Object.is(oldProps, newProps)
  +-------------------+
    /               \
  SAME             CHANGED
  /                   \
[ Skip Render ]    [ Re-Render ]

```

### Usage Rules

| ✅ When to Use | ❌ When to Avoid |
| --- | --- |
| **Heavy pure components:** Complex data grids, SVG maps, or long lists. | **Lightweight wrappers:** A simple `<div>` wrapper takes longer to diff than to re-render. |
| **Frequent parent updates:** If the parent updates 60fps (e.g., mouse tracking). | **`children` prop:** If you pass `children`, it's a new array/object every time, breaking memo. |

### Code Example & Gotchas

```javascript
import { memo, useState, useCallback } from 'react';

// 1. Wrap the component
const ExpensiveGraph = memo(function ExpensiveGraph({ data, onHover }) {
  console.log("Graph rendered");
  return <canvas>{/* Heavy drawing logic */}</canvas>;
});

export default function App() {
  const [count, setCount] = useState(0);
  const [graphData] = useState([1, 2, 3]);

  // 2. PROTECT THE PROPS: If we don't use useCallback, onHover gets a 
  // new memory address on every click, completely breaking React.memo!
  const handleHover = useCallback(() => {
    console.log("Hovered");
  }, []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Clicks: {count}</button>
      <ExpensiveGraph data={graphData} onHover={handleHover} />
    </div>
  );
}

```

### Mastery & Interview Execution

**Conceptual Question:** "If I pass an empty array `[]` directly into a memoized component like `<List items="{[]}"/>`, will it re-render when the parent renders?"

**Your Answer:** "Yes, it will always re-render. `React.memo` performs a shallow comparison. In JavaScript, `[] === []` is false because they occupy different memory addresses. The parent creates a brand new array reference on every render cycle. To fix this, the array must be extracted outside the component or wrapped in `useMemo`."

## 3. `createContext()`: The Data Wormhole

**Introduced:** React 16.3 (2018)

### The Core Concept (Why, What, How)

To avoid Prop Drilling (passing data through dozens of intermediate components), Context creates an ambient data layer. `createContext()` initializes this layer, returning a `Provider` (to broadcast data) and a `Consumer` (which modern React handles via the `useContext` hook).

```text
       +-------------------------------+
       | Provider value={{ id: 1 }}    |
       +-------------------------------+
                  |      \
        (props)   |       \ (teleports)
                  v        \
       +----------------+   \
       |  Intermediate  |    \
       |  (Ignores it)  |     \
       +----------------+      \
                  |             v
                  v      +----------------+
       +----------------+| Child (Hook)   |
       |  Intermediate  || reads { id: 1 }|
       +----------------++----------------+

```

### Usage Rules

| ✅ When to Use | ❌ When to Avoid |
| --- | --- |
| **Global configuration:** Themes, user auth sessions, language/i18n. | **High-frequency state:** Scroll positions, rapid typing. |
| **Compound components:** Linking `<Accordion>` to `<Accordion.Item>`. | **Passing simple props:** If it's only going down two levels, stick to props. |

### Code Example & Gotchas

**Gotcha #1: The Default Value Trap.**
The argument passed to `createContext("Light")` is *only* used if a component attempts to read the context and there is absolutely **no Provider anywhere above it in the tree**.

**Gotcha #2: The Object Identity Avalanche.**

```javascript
import { createContext, useContext, useState, useMemo } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState('Pushkar');

  // FATAL BUG: value={{ user, logout }} 
  // Creates a new object literal every render. Forces all consumers to re-render!

  // FIX: Memoize the context value
  const value = useMemo(() => ({ user, setUser }), [user]);

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

```

## 4. `act()` (with React Testing Library)

**Introduced:** React 16.8 (Testing Utilities)

### The Core Concept (Why, What, How)

In a real browser, React batches state updates and DOM mutations efficiently. In a Node.js testing environment (like Jest/Vitest), events happen instantly, often out of sync with React's internal queue.

`act()` is a boundary. It tells the testing environment: *"Execute all React rendering, handle all state updates, and flush all `useEffect` hooks BEFORE moving on to the next line of code."*

```text
  WITHOUT act()                           WITH act()
  
  fireEvent.click(button)                 act(() => {
       |                                    fireEvent.click(button)
  expect(UI to change) -> FAILS!                 |
       |                                    React updates state
  React updates state (Too late)            React runs useEffects
                                          })
                                                 |
                                          expect(UI to change) -> PASS!

```

### Usage Rules

| ✅ When to Use | ❌ When to Avoid |
| --- | --- |
| **Testing custom hooks:** Testing `renderHook` state changes. | **Standard RTL events:** RTL's `userEvent.click()` automatically wraps itself in `act()`. |
| **Mocking timers/promises:** Resolving fake timers in Jest. | **Standard rendering:** RTL's `render()` is already wrapped in `act()`. |

### Code Example & Gotchas

**Gotcha: The Warning.**
*Warning: An update to Component inside a test was not wrapped in act(...).*
This happens because an asynchronous operation (like a Promise) resolved *after* your test assertion finished, triggering a late state update.

```javascript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('loads and displays user', async () => {
  render(<UserProfile id={1} />);

  // RTL automatically wraps userEvent in act()
  await userEvent.click(screen.getByText('Load'));

  // If you are waiting for an async API call inside the component to finish,
  // do NOT write a manual act(). Use waitFor, which handles the act boundary 
  // internally during its retry loop.
  await waitFor(() => {
    expect(screen.getByText('Pushkar is loaded')).toBeInTheDocument();
  });
});

```

## 5. `startTransition()`

**Introduced:** React 18 (Concurrent Features)

### The Core Concept (Why, What, How)

Before React 18, every state update was urgent. If you filtered 10,000 items while the user typed, the browser froze. `startTransition` hooks into React's Concurrent Engine. It lets you mark a state update as "low priority," allowing React to pause rendering that update if a high-priority event (like typing) occurs.

```text
  [ URGENT QUEUE ]        [ TRANSITION (LOW PRIORITY) QUEUE ]
  - Type "A" in input     - Filter 10,000 items by "A"
  - Click button          - Render new Data Grid
       |                           |
    (Executes instantly)      (Executes when Main Thread is idle)
                              (Interruptible if new Urgent tasks arrive!)

```

### Usage Rules

| ✅ When to Use | ❌ When to Avoid |
| --- | --- |
| **Heavy UI filtering:** Updating maps, charts, or massive lists based on input. | **Input values:** Never transition the state controlling an `<input value={x}>`. It will drop keystrokes. |
| **Tab switching:** Navigating to a heavy view without freezing the current view. | **API calls:** Do not wrap `fetch` requests here; it's for interrupting CPU rendering time. |

### Code Example & Gotchas

**Gotcha: The Synchronous Requirement.**
The callback passed to `startTransition` MUST be synchronous so React can instantly record which state setters were called.

```javascript
import { useState, startTransition } from 'react';

export function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleChange = (e) => {
    // 1. Urgent: Update the input instantly
    setQuery(e.target.value);

    // ❌ BAD: React cannot track async callbacks
    // startTransition(() => setTimeout(() => setResults(data), 1000));

    // ✅ GOOD: State setter is executed synchronously in the wrapper
    startTransition(() => {
      // 2. Non-Urgent: Render the heavy list in the background
      setResults(heavyFilter(e.target.value)); 
    });
  };

  return <input value={query} onChange={handleChange} />;
}

```

### Mastery & Interview Execution

**Conceptual Question:** "What is the difference between `startTransition` and the `useTransition` hook?"

**Your Answer:** "They do the exact same thing under the hood. The difference is access to the pending state and where they can be called. `useTransition` provides an `isPending` boolean to show a loading spinner, but because it's a hook, it can only be called inside a component. `startTransition` is a standalone function imported directly from `react`. You use it when you need to trigger a transition from *outside* a React component, such as inside a Redux thunk, a vanilla JS event listener, or an external data router."</Accordion.Item>
