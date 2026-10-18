# React Lifecycle

- [React lifecycle](#react-lifecycle)
  - [1. The Core Concept: The Lifecycle Phases](#1-the-core-concept-the-lifecycle-phases)
    - [The Unidirectional Flow of a Render](#the-unidirectional-flow-of-a-render)
  - [2. Modern React: Handling the Lifecycle with Hooks](#2-modern-react-handling-the-lifecycle-with-hooks)
    - [Pattern A: Run on EVERY Render](#pattern-a-run-on-every-render)
    - [Pattern B: Run ONLY on Initial Mount](#pattern-b-run-only-on-initial-mount)
    - [Pattern C: Run on Mount AND when Specific Dependencies Change](#pattern-c-run-on-mount-and-when-specific-dependencies-change)
    - [Pattern D: Clean Up Before Unmount (The Cleanup Function)](#pattern-d-clean-up-before-unmount-the-cleanup-function)
  - [3. Mapping Old Class Lifecycles to Modern Hooks](#3-mapping-old-class-lifecycles-to-modern-hooks)
    - [A. Initializing State (`constructor` vs `useState`)](#a-initializing-state-constructor-vs-usestate)
    - [B. Birth (`componentDidMount` vs `useEffect`)](#b-birth-componentdidmount-vs-useeffect)
    - [C. Maintenance (`componentDidUpdate` vs `useEffect` with Deps)](#c-maintenance-componentdidupdate-vs-useeffect-with-deps)
    - [D. Death (`componentWillUnmount` vs `useEffect` Cleanup)](#d-death-componentwillunmount-vs-useeffect-cleanup)
    - [E. Optimizing Re-renders (`shouldComponentUpdate` vs `React.memo`)](#e-optimizing-re-renders-shouldcomponentupdate-vs-reactmemo)
    - [F. Rare Case: `getDerivedStateFromProps`](#f-rare-case-getderivedstatefromprops)
  - [React lifecycle gotchas](#react-lifecycle-gotchas)
    - [1. The Stale Closure (The "Frozen in Time" Trap)](#1-the-stale-closure-the-frozen-in-time-trap)
      - [Deep Dive: The Updater Function Pattern](#deep-dive-the-updater-function-pattern)
    - [2. The Double Mount (The "Ghost Trigger")](#2-the-double-mount-the-ghost-trigger)
    - [3. Referential Inequality (The "Infinite Loop" Trap)](#3-referential-inequality-the-infinite-loop-trap)
    - [4. Batched State Updates (The "Lost Math")](#4-batched-state-updates-the-lost-math)
    - [5. The Render-Phase Update (The "Accidental Lifecycle")](#5-the-render-phase-update-the-accidental-lifecycle)
  - [4. The Anatomy of a Render: Step-by-Step Execution Order](#4-the-anatomy-of-a-render-step-by-step-execution-order)
    - [The Master Pipeline: Render vs. Commit vs. Effects](#the-master-pipeline-render-vs-commit-vs-effects)
    - [Phase 1: The Initial Mount Sequence](#phase-1-the-initial-mount-sequence)
    - [Phase 2: The Re-render (Update) Sequence](#phase-2-the-re-render-update-sequence)
      - [How Previous Effect Cleanup Works](#how-previous-effect-cleanup-works)
    - [Phase 3: The Unmount Sequence](#phase-3-the-unmount-sequence)
    - [Parent vs. Child Execution Order](#parent-vs-child-execution-order)
    - [Verification Example: The Full Lifecycle in Action](#verification-example-the-full-lifecycle-in-action)
  - [5. Conditional Rendering Under the Hood: What Actually Happens to a Child](#5-conditional-rendering-under-the-hood-what-actually-happens-to-a-child)
    - [What Happens When `show` Toggles to `false` (Destruction)](#what-happens-when-show-toggles-to-false-destruction)
    - [What Happens When `show` Toggles Back to `true` (Recreation)](#what-happens-when-show-toggles-back-to-true-recreation)
    - [Conditional Rendering vs. CSS Hiding (`display: none`)](#conditional-rendering-vs-css-hiding-display-none)
    - [Common Gotchas with Conditional Rendering](#common-gotchas-with-conditional-rendering)
    - [How to Preserve State Across Conditional Toggles](#how-to-preserve-state-across-conditional-toggles)
  - [6. What Happens to a Parent When a Child State Changes?](#6-what-happens-to-a-parent-when-a-child-state-changes)
  - [7. Effect Cleanup on Dependency Changes: API Aborts & Event Listeners](#7-effect-cleanup-on-dependency-changes-api-aborts--event-listeners)
    - [Case 1: Canceling In-Flight API Requests (Preventing Race Conditions)](#case-1-canceling-in-flight-api-requests-preventing-race-conditions)
    - [Case 2: Event Listener Teardown (Preventing Listener Accumulation)](#case-2-event-listener-teardown-preventing-listener-accumulation)
    - [Summary Checklist: What Happens on Dependency Change](#summary-checklist-what-happens-on-dependency-change)

In React, a component has a lifecycle that you can monitor and manipulate during its three main phases: **Mounting**, **Updating**, and **Unmounting**.

In older versions of React, you had to write Class components and hook into these phases using specific built-in methods. In modern React (18+), you use functional components and **Hooks** (primarily `useEffect`) to manage these phases declaratively.

## 1. The Core Concept: The Lifecycle Phases

Every component travels through three stages, like a timeline:

```text
       +-------------------------------------------------------------+
       |                       COMPONENT TIMELINE                    |
       +-------------------------------------------------------------+
              |                             |                        |
              v                             v                        v
       1. MOUNTING                  2. UPDATING              3. UNMOUNTING
  (Component is born &         (State or props change,   (Component dies &
   inserted into the DOM)       forcing a re-render)      leaves the DOM)

```

### The Unidirectional Flow of a Render

1. **Mounting:** React sets up initial state, executes your component code, figures out the UI layout, and places the physical DOM nodes into the browser.
2. **Updating:** The user types something, fetches data, or receives new props. React re-runs the component function, computes the differences (the diffing algorithm), and updates _only_ the modified parts of the DOM.
3. **Unmounting:** The component is no longer needed (e.g., user navigates away to another page). React cleans it up from the browser memory.

## 2. Modern React: Handling the Lifecycle with Hooks

In functional components, you do not think about discrete "methods." Instead, you think in terms of **Side Effects and Synchronization** using the `useEffect` hook.

The structure of `useEffect` allows it to act as multiple lifecycle milestones combined, configured entirely by its **Dependency Array** (the second argument).

```text
  useEffect( () => {
     // 1. Your side-effect code here (Mount / Update logic)

     return () => {
        // 2. Optional Cleanup function here (Unmount logic)
     };
  }, [ /* 3. Dependency Array determines WHEN this triggers */ ] );

```

Here are the 4 configurations you will use most often:

### Pattern A: Run on EVERY Render

If you leave the dependency array completely empty (no brackets at all), the effect fires on the initial mount and after _every single update_ to the screen.

```javascript
import { useEffect, useState } from "react";

function LogEveryRender() {
  const [text, setText] = useState("");

  useEffect(() => {
    // Triggers on mount AND whenever "text" or any other state triggers a re-render
    console.log("The component rendered or updated!");
  });

  return <input value={text} onChange={(e) => setText(e.target.value)} />;
}
```

### Pattern B: Run ONLY on Initial Mount

Adding an empty array `[]` tells React: "This effect does not depend on any variables from the component scope." It runs exactly once when the component first appears.

```javascript
import { useEffect, useState } from "react";

function UserProfile() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Triggers EXACTLY ONCE when the component is added to the screen
    fetch("/api/user/1")
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, []); // Empty array = mount only

  return <div>{user ? user.name : "Loading user..."}</div>;
}
```

### Pattern C: Run on Mount AND when Specific Dependencies Change

Passing variables inside the array `[dependency1, dependency2]` tells React to skip executing the effect unless one of those values changes between renders.

```javascript
import { useEffect, useState } from "react";

function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    // Triggers on mount AND whenever the "query" prop changes value
    fetch(`/api/search?q=${query}`)
      .then((res) => res.json())
      .then((data) => setResults(data));
  }, [query]); // Re-runs ONLY when query changes

  return (
    <ul>
      {results.map((r) => (
        <li key={r.id}>{r.title}</li>
      ))}
    </ul>
  );
}
```

### Pattern D: Clean Up Before Unmount (The Cleanup Function)

If your effect returns a function, React will execute that function right before the component is destroyed (unmounted), or immediately before re-running the effect on a new update. This prevents memory leaks.

```javascript
import { useEffect, useState } from "react";

function WindowTracker() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);

    // 1. Setup subscription on mount
    window.addEventListener("resize", handleResize);

    // 2. Return a cleanup function
    return () => {
      // Cleans up the event listener when the component disappears
      window.removeEventListener("resize", handleResize);
      console.log("Cleaned up window listener");
    };
  }, []); // Runs setup once on mount; runs cleanup once on unmount

  return <h1>Window Width: {width}px</h1>;
}
```

## 3. Mapping Old Class Lifecycles to Modern Hooks

If you are reading older React documentation or working on a legacy codebase, you will encounter Class lifecycle methods.

Here is the exact conceptual bridge showing how old methods map to modern hooks.

```text
+-------------------------+                           +-------------------------+
|   CLASS LIFECYCLE       |                           |      MODERN HOOKS       |
+-------------------------+                           +-------------------------+
|  constructor()          |  ======================>  |  useState() init value  |
|  componentDidMount()    |  ======================>  |  useEffect(() => {}, [])|
|  componentDidUpdate()   |  ======================>  |  useEffect(() => {}, [x])|
|  componentWillUnmount() |  ======================>  |  useEffect(() => return)|
|  shouldComponentUpdate()|  ======================>  |  React.memo() wrapper   |
+-------------------------+                           +-------------------------+

```

### A. Initializing State (`constructor` vs `useState`)

In Class components, you configured default state variables inside an object constructor. In modern functional components, you call `useState`.

```text
  Class Workflow:      constructor() ---> bind methods ---> set initial object state
  Functional Workflow: component run ---> useState(initialValue) execution

```

- **The Old Class Way:**

```javascript
class OldCounter extends React.Component {
  constructor(props) {
    super(props);
    // State initialization
    this.state = { count: 0 };
    // Event binding required for class methods
    this.increment = this.increment.bind(this);
  }
  increment() {
    this.setState({ count: this.state.count + 1 });
  }
  render() {
    return <button onClick={this.increment}>{this.state.count}</button>;
  }
}
```

- **The Modern Hooks Way:**

```javascript
function ModernCounter() {
  // Simple, inline state definition. No method binding needed.
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### B. Birth (`componentDidMount` vs `useEffect`)

`componentDidMount` fired immediately after a component was inserted into the DOM tree. It was the designated place to trigger API fetches or DOM setup.

```text
  Class Timeline:      render() ---> DOM injection ---> componentDidMount()
  Functional Timeline: function execution ---> DOM injection ---> useEffect([], empty array)

```

- **The Old Class Way:**

```javascript
class OldFetch extends React.Component {
  componentDidMount() {
    console.log("Component is mounted to the actual screen!");
    // Trigger side effect
  }
  render() {
    return <div>Hello World</div>;
  }
}
```

- **The Modern Hooks Way:**

```javascript
function ModernFetch() {
  useEffect(() => {
    console.log("Component is mounted to the actual screen!");
  }, []); // An empty array mimics componentDidMount exactly

  return <div>Hello World</div>;
}
```

### C. Maintenance (`componentDidUpdate` vs `useEffect` with Deps)

`componentDidUpdate` fired immediately after rendering occurred due to a change in updates or props. You had to manually compare old props versus new props inside the method to avoid infinite render loops.

```text
  Class Path:      State Changes ---> render() ---> componentDidUpdate(prevProps, prevState)
  Functional Path: State Changes ---> component execution ---> useEffect([dependencies])

```

- **The Old Class Way:**

```javascript
class OldSync extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    // You had to manually write manual conditions to see if data changed
    if (prevProps.userId !== this.props.userId) {
      this.fetchNewUserData(this.props.userId);
    }
  }
  render() {
    return <div>User UI</div>;
  }
}
```

- **The Modern Hooks Way:**

```javascript
function ModernSync({ userId }) {
  useEffect(() => {
    // React handles the comparison checking behind the scenes
    fetchNewUserData(userId);
  }, [userId]); // Triggers automatically whenever userId changes

  return <div>User UI</div>;
}
```

### D. Death (`componentWillUnmount` vs `useEffect` Cleanup)

`componentWillUnmount` fired right before a component was removed from the DOM. It was strictly used to teardown intervals, cancel net requests, or remove listeners.

```text
  Class Cleanup:       Remove request ---> Remove DOM node ---> Unmounted
  Functional Cleanup:  Run returned clean function ---> Remove DOM node ---> Unmounted

```

- **The Old Class Way:**

```javascript
class OldClock extends React.Component {
  componentDidMount() {
    this.timerId = setInterval(() => this.tick(), 1000);
  }
  componentWillUnmount() {
    // Clean up interval before destruction
    clearInterval(this.timerId);
  }
  render() {
    return <div>Clock UI</div>;
  }
}
```

- **The Modern Hooks Way:**

```javascript
function ModernClock() {
  useEffect(() => {
    const timerId = setInterval(() => tick(), 1000);

    // Return function behaves like componentWillUnmount
    return () => {
      clearInterval(timerId);
    };
  }, []);

  return <div>Clock UI</div>;
}
```

### E. Optimizing Re-renders (`shouldComponentUpdate` vs `React.memo`)

`shouldComponentUpdate` returned a boolean telling React whether it should skip rendering the component entirely when new props came in. In functional components, we optimize this by wrapping our component function inside `React.memo`.

- **The Old Class Way:**

```javascript
class OldPureCard extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Return true if you want a re-render, false if you want to skip it
    return nextProps.title !== this.props.title;
  }
  render() {
    return <h1>{this.props.title}</h1>;
  }
}
```

- **The Modern Hooks Way:**

```javascript
import React from "react";

// Wrap the whole component function in React.memo()
const ModernPureCard = React.memo(function ModernPureCard({ title }) {
  return <h1>{title}</h1>;
});
// React.memo automatically performs a shallow comparison of props
// and skips re-rendering if props are unchanged.
```

### F. Rare Case: `getDerivedStateFromProps`

This was a rare, complex class method used to update internal component state when incoming props changed over time. React core maintainers recommend bypassing this altogether in functional components by setting state directly during the rendering cycle.

- **The Modern Functional Way:**

```javascript
function Form({ items }) {
  const [selection, setSelection] = useState(null);
  const [prevItems, setPrevItems] = useState(items);

  // If items prop changed since the last render pass, sync local state
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null); // Clear selection state
  }

  return <div>Form items go here</div>;
}
```

## React lifecycle gotchas

Here are the most common "gotcha" questions surrounding the React lifecycle and hooks. These are the traps that create frustrating bugs in production and frequently appear in senior-level technical interviews.

### 1. The Stale Closure (The "Frozen in Time" Trap)

**The Question:** Why does my `setInterval` or event listener inside a `useEffect` keep logging the initial state value, even after the state has updated?

**The Gotcha:** JavaScript closures capture variables at the time they are created. If you use an empty dependency array `[]`, the effect only runs on the first render. Any functions created inside it will forever see the state as it was during that very first render.

```text
  +--------------------+       +-----------------------------------+
  | Render 1 (count=0) | ----> | useEffect() runs                  |
  +--------------------+       | Creates interval()                |
                               | interval() remembers count as 0   |
                               +-----------------------------------+
                                                |
  +--------------------+                        | (1 sec later)
  | Render 2 (count=1) |                        v
  +--------------------+                 interval() fires!
                                       Logs: "Count is 0" (BUG!)

```

**The Code:**

```javascript
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      // BUG: count is forever locked at 0 here
      console.log("Count is:", count);
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []); // Empty array prevents the effect from recreating the closure

  return <div>{count}</div>;
}
```

**The Solution:** Use the "updater function" form of `setState`, which guarantees you get the latest state regardless of the closure, or add `count` to the dependency array (though that resets the interval every time count changes).

```javascript
// FIX: Updater function doesn't rely on the 'count' variable in scope
setCount((prevCount) => prevCount + 1);
```

#### Deep Dive: The Updater Function Pattern

Instead of passing a raw value (`setCount(count + 1)`), you pass an updater function: `setCount(prev => prev + 1)`. React passes the **pending/current** state into your callback.

```text
  Direct Value:     setCount(count + 1)         --> Reads 'count' from closure (can be stale!)
  Updater Function: setCount(prev => prev + 1)  --> React feeds latest state from pending queue

       State Queue: [ (prev: 0) => 1 ] ---> [ (prev: 1) => 2 ] ---> Final State: 2
```

- **Benefits:**
  - **Immune to Stale Closures:** Always receives the freshest state, even inside `setInterval`, event listeners, or asynchronous callbacks.
  - **Safe with Batching:** Chained updates in the same tick execute sequentially without overwriting each other.
  - **Stabilizes Effect Dependencies:** Allows omitting the state variable from `useEffect`/`useCallback` dependency arrays, avoiding unnecessary resets or re-subscriptions.
- **When to Use:**
  - When the new state depends directly on the previous state (counters, toggling booleans `prev => !prev`, appending to arrays `prev => [...prev, item]`).
  - Inside effects or callbacks where closing over a changing dependency would cause stale values or unwanted re-subscriptions.
- **Drawbacks:**
  - **Must Be Pure:** React may run updater functions twice in development (under Strict Mode) to detect side effects. Never mutate objects directly or trigger side effects inside the updater.
  - **Syntax Overhead:** Unnecessary ceremony when setting a fixed or independent value.
- **When NOT to Use:**
  - When setting a static or replacement value that does not depend on past state (e.g., `setIsOpen(false)`, `setUser(apiData)`, `setCount(0)`).
  - When computing state depends on multiple other state variables simultaneously (better handled via `useReducer`).


### 2. The Double Mount (The "Ghost Trigger")

**The Question:** I added an API call to my `useEffect` with an empty dependency array `[]`. Why is the API being called twice when the component loads?

**The Gotcha:** In React 18, when using Strict Mode in development, React intentionally mounts your component, unmounts it, and remounts it immediately to simulate a component being hidden and shown. This is a stress test designed to catch missing cleanup functions.

```text
       Development Mode (Strict Mode ON)
       ---------------------------------
  1. Mount    ---> useEffect runs (API Call 1)
  2. Unmount  ---> Cleanup function runs
  3. Remount  ---> useEffect runs (API Call 2)

```

**The Solution:** You do not "fix" the double fire itself (it won't happen in production). Instead, you fix your code to handle being interrupted. For API calls, use an `AbortController` in the cleanup function so the first duplicate request is cancelled.

```javascript
useEffect(() => {
  const controller = new AbortController();

  fetch("/api/data", { signal: controller.signal })
    .then((res) => res.json())
    .then(setData);

  return () => {
    // Cancels the fetch if the component unmounts (or during Strict Mode's ghost unmount)
    controller.abort();
  };
}, []);
```

### 3. Referential Inequality (The "Infinite Loop" Trap)

**The Question:** I passed an array/object to a child component, and now `useEffect` is caught in an infinite loop, crashing the browser. What happened?

**The Gotcha:** In JavaScript, `[1, 2] === [1, 2]` is `false` because they occupy different memory addresses. If you create a new array, object, or function directly inside the render body, it gets a new memory address on _every single render_. If that object is used in a dependency array, it triggers the effect, which updates state, which triggers a render, which creates a new object...

```text
  Render Starts
       |
       v
  const config = { retries: 3 }  (Creates memory address 0x001)
       |
       v
  useEffect checks deps: 0x001 is different from last render!
       |
       v
  Runs effect -> calls setState() -> Triggers Re-render
       |
       v
  Render Starts
       |
       v
  const config = { retries: 3 }  (Creates memory address 0x002) ---> INFINITE LOOP

```

**The Code:**

```javascript
function DataFetcher() {
  const [data, setData] = useState(null);

  // BUG: This object is recreated in memory every single render
  const fetchOptions = { method: "GET" };

  useEffect(() => {
    fetch("/api", fetchOptions).then(setData);
  }, [fetchOptions]); // Triggered continuously!
}
```

**The Solution:** Move static objects _outside_ the component entirely so they are created once, or wrap them in `useMemo` (or `useCallback` for functions) so React caches the memory reference.

```javascript
// FIX 1: Move static objects outside the component (constant memory address)
const fetchOptions = { method: "GET" };

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api", fetchOptions).then(setData);
  }, []); // Safe: fetchOptions memory address never changes across renders
}

// FIX 2: Wrap dynamic objects/functions in useMemo (cached memory address)
import { useMemo } from "react";

function SearchFetcher({ query }) {
  const [data, setData] = useState(null);

  // Re-creates object ONLY when 'query' actually changes value
  const fetchOptions = useMemo(() => ({
    method: "POST",
    body: JSON.stringify({ query }),
  }), [query]);

  useEffect(() => {
    fetch("/api/search", fetchOptions).then(setData);
  }, [fetchOptions]); // Stable: won't trigger on unrelated parent re-renders
}
```

### 4. Batched State Updates (The "Lost Math")

**The Question:** I call `setCount(count + 1)` three times in a row inside a button click handler. Why does the count only increase by 1?

**The Gotcha:** React batches state updates for performance. It does not update the state immediately on the next line of code. Instead, it queues the updates and executes them in one go at the end of the event handler. Because the closure sees `count` as its current value (e.g., 0), all three calls are effectively saying `setCount(0 + 1)`.

```text
  Initial State: count = 0

  Click Handler executes:
  1. setCount(0 + 1) ---> Queued: Set to 1
  2. setCount(0 + 1) ---> Queued: Set to 1
  3. setCount(0 + 1) ---> Queued: Set to 1

  End of function ---> React processes queue ---> Final state is 1

```

**The Code:**

```javascript
const handleTripleClick = () => {
  // If count is 0, all three lines resolve to setCount(1)
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
};
```

**The Solution:** When the next state depends on the previous state, always use the updater function. React guarantees the updater function will receive the most recent state in the queue.

```javascript
const handleTripleClick = () => {
  setCount((prev) => prev + 1); // Receives 0, returns 1
  setCount((prev) => prev + 1); // Receives 1, returns 2
  setCount((prev) => prev + 1); // Receives 2, returns 3
};
```

### 5. The Render-Phase Update (The "Accidental Lifecycle")

**The Question:** I'm getting a console warning: _"Cannot update a component while rendering a different component."_ What causes this?

**The Gotcha:** You called a `setState` function directly in the main body of your component function, rather than inside a `useEffect` or an event handler (like `onClick`). Because calling `setState` tells React to re-render, calling it _during_ the render tells React to start over immediately, causing a rapid loop.

```text
  +-------------------------------+
  | function BadComponent() {     |
  |   const [x, setX] = useState; |
  |                               |
  |   setX(5); <------------------|--- RENDER HALTS, RESTARTS IMMEDIATELY
  |                               |
  |   return <div>{x}</div>;      |
  | }                             |
  +-------------------------------+

```

**The Solution:** State updates must happen as a reaction to something. They either belong in an event handler (reaction to a user) or a `useEffect` (reaction to a lifecycle/prop change). Alternatively, calculate values inline as derived state.

```javascript
// FIX 1: Trigger update inside an event handler (reaction to user action)
function GoodButton() {
  const [count, setCount] = useState(0);

  // Safe: Runs ONLY on user click, NOT while React is rendering the component
  return <button onClick={() => setCount(5)}>Set to 5</button>;
}

// FIX 2: Trigger update inside useEffect (reaction to lifecycle/prop change)
function SyncOnPropChange({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Safe: Runs AFTER the component has rendered and painted to the DOM
    setUser(null); // Reset or re-sync state safely
  }, [userId]);

  return <div>{user ? user.name : "Loading..."}</div>;
}

// FIX 3: Avoid state entirely — compute derived state inline during render
function FilteredList({ items, filter }) {
  // Safe & performant: No setState call needed! Computes fresh value on every render
  const filteredItems = items.filter((item) => item.includes(filter));

  return (
    <ul>
      {filteredItems.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

## 4. The Anatomy of a Render: Step-by-Step Execution Order

Every single render in React — whether it is the initial birth (**Mount**), an interactive state change (**Update**), or component destruction (**Unmount**) — flows through an exact, deterministic pipeline under the hood.

Understanding this sequence clarifies *why* closures freeze, *when* DOM elements exist to be measured, and *in what order* parent and child components run.

### The Master Pipeline: Render vs. Commit vs. Effects

React splits component lifecycle execution into three distinct phases:

1. **The Render Phase (Pure Calculation):** React executes the component function body, runs `useState`/hooks, calculates derived state, evaluates child components, and generates the Virtual DOM tree (JSX). **No real DOM changes happen here.**
2. **The Commit Phase (DOM Mutation):** React applies the calculated differences to the actual browser DOM (inserting, updating, or deleting physical DOM nodes). `useLayoutEffect` runs synchronously here before screen paint.
3. **The Effects Phase (Browser Paint & Side Effects):** The browser paints the pixels to the screen. Immediately after paint, asynchronous `useEffect` callbacks and cleanups run.

```text
  +-------------------------------------------------------------------------+
  |                           THE 3-STAGE PIPELINE                          |
  +-------------------------------------------------------------------------+
       |
       v
  [ 1. RENDER PHASE ]     • Pure calculation, NO side-effects
       |                  • Runs component function & child functions
       |                  • Produces Virtual DOM (JSX)
       v
  [ 2. COMMIT PHASE ]     • React updates Real DOM (createElement / appendChild)
       |                  • useLayoutEffect runs SYNCHRONOUSLY (before paint)
       v
  [ BROWSER PAINT ]       • Browser recalculates geometry & paints pixels to screen
       |
       v
  [ 3. EFFECTS PHASE ]    • useEffect callbacks run ASYNCHRONOUSLY (post-paint)
                          • Network requests, subscriptions, event listeners
```

---

### Phase 1: The Initial Mount Sequence

When a component is mounted for the very first time, React executes these steps in strict numerical order:

```text
  1. Trigger Root Mount
       |
       v
  2. State & Hook Init (useState, useRef created)
       |
       v
  3. Parent Function Body Runs (Calculates JSX)
       |
       v
  4. Child Component Render (Recursively runs child body & children)
       |
       v
  5. Commit Phase (React writes all nodes to the Real DOM)
       |
       v
  6. useLayoutEffect Fires (Synchronous, pre-paint DOM measurement)
       |
       v
  7. Browser Paint (User physically sees the UI)
       |
       v
  8. useEffect Fires (Child effects run first, then Parent effect)
```

1. **Trigger:** `createRoot().render()` or a parent conditionally rendering the component (`{show && <Component />}`).
2. **State & Ref Initialization:**
   - `useState(initialValue)` computes initial values (or runs lazy initializer `useState(() => compute())`).
   - `useRef(initialValue)` creates the persistent `{ current: initialValue }` container.
3. **Execute Component Function (Render Phase):**
   - The function body runs from top to bottom.
   - Computes local variables, functions, and derived state.
   - `useMemo` runs its compute callback for the first time.
   - Evaluates and returns JSX elements.
4. **Child Components Render (Recursive):**
   - React encounters child component tags in JSX and runs each child component function body in top-down order.
5. **Reconciliation (Commit Prep):**
   - React constructs the initial Virtual DOM tree (the "Fiber tree").
6. **Commit to DOM:**
   - React creates physical HTML elements (`document.createElement`) and inserts them into the browser DOM.
7. **`useLayoutEffect` Execution:**
   - Runs *synchronously* immediately after the DOM mutations, before the browser paints. (Used for measuring DOM nodes like `getBoundingClientRect()`).
8. **Browser Paint:**
   - The browser paints the pixels on screen. The user can now see the rendered output.
9. **`useEffect` Execution:**
   - React runs `useEffect(() => {}, [])` *asynchronously* after paint.
   - **Child `useEffect` callbacks run first**, followed by the **Parent `useEffect`**.

---

### Phase 2: The Re-render (Update) Sequence

Triggered whenever state changes (`setState`), new props are passed from above, or a consumed context updates:

```text
  1. State Update Triggered (e.g., setCount(prev => prev + 1))
       |
       v
  2. React Batches Updates (Processes pending state queue)
       |
       v
  3. Parent Component Re-runs (useState returns latest state; ignores init)
       |
       v
  4. Child Components Re-render (All children re-render by default)
       |
       v
  5. Reconciliation / Diffing (Compares old Virtual DOM vs new Virtual DOM)
       |
       v
  6. Commit Phase: Minimal DOM Mutation (Patches only changed DOM nodes)
       |
       v
  7. useLayoutEffect Cleanup & Run (Synchronous, pre-paint)
       |
       v
  8. Browser Paint (Screen reflects updated state)
       |
       v
  9. useEffect Cleanup (Runs previous render's cleanup function)
       |
       v
  10. useEffect Execution (Runs new effect callback with fresh deps)
```

1. **Trigger & Queue Batching:** An event fires (e.g. `onClick`), updater functions are queued, and React batches the updates.
2. **Re-execute Component Body (Render Phase):**
   - React re-invokes the component function.
   - `useState` returns the latest state from the queue; the initial value argument is completely ignored.
   - `useMemo` and `useCallback` check dependency arrays: if dependencies haven't changed, cached references are returned.
   - Generates new JSX.
3. **Child Re-renders:**
   - By default, every child inside the JSX re-renders, **unless** wrapped in `React.memo` with identical props.
4. **Reconciliation (Diffing):**
   - React diffs the new Virtual DOM against the previous Virtual DOM snapshot to find minimal changes.
5. **Commit to DOM:**
   - React updates *only* the specific DOM attributes or text nodes that changed (`node.textContent = ...`).
6. **Browser Paint:**
   - The browser recalculates layout and repaints the screen with the new DOM.
7. **Previous Effect Cleanup (Post-Paint):**
   - If dependencies changed, React calls the **cleanup function** returned by the *previous* render's `useEffect`.
8. **New `useEffect` Runs:**
   - React executes the new `useEffect` callback with the updated values captured in its scope.

#### How Previous Effect Cleanup Works

Many developers assume effect cleanups only run on **Unmount**. In reality, on every re-render where dependencies change, React runs **Cleanup of Previous Render $\rightarrow$ Setup of Next Render**.

React strictly enforces: **"Clean up the old world before setting up the new world."**

```text
  Render 1 (id: 1)  -----> Executes Effect(id: 1)
                           Returns cleanupFn (captures id: 1 in closure)
                           Stored in React Fiber: [ cleanupFn_1 ]
                                 |
                                 |  (Prop/state changes id to 2)
                                 v
  Render 2 (id: 2)  -----> 1. React invokes stored cleanupFn_1()
                               Logs: "Disconnect from 1"  (Cleans old world)
                           2. React invokes new Effect(id: 2)
                               Logs: "Connect to 2"      (Sets up new world)
                               Returns cleanupFn_2 (captures id: 2)
```

##### Behind the Scenes: How React Remembers What to Clean Up
1. **Fiber Storage:** When an effect runs, React saves the returned cleanup callback on that hook's internal Fiber object (`fiber.memoizedState.destroy`).
2. **Dependency Check:** On re-render, React compares current deps vs. previous deps (`Object.is`).
3. **Old Closure Execution:** If dependencies changed, React retrieves `destroy` from the previous render and invokes it. Thanks to JavaScript closures, that cleanup function still accesses the **old** state/prop values from when it was declared.

##### Code Example

```javascript
import { useEffect } from "react";

function ChatRoom({ roomId }) {
  useEffect(() => {
    console.log(`🟢 [Setup] Connected to room: "${roomId}"`);

    return () => {
      // When roomId changes from "general" to "random",
      // this closure still holds roomId = "general"!
      console.log(`🧹 [Cleanup] Disconnected from room: "${roomId}"`);
    };
  }, [roomId]);

  return <h2>Room: {roomId}</h2>;
}
```

##### Exact Console Sequence Across Lifecycle:
```text
1. Initial Mount (roomId = "general"):
🟢 [Setup] Connected to room: "general"

2. Prop Changes (roomId = "random"):
🧹 [Cleanup] Disconnected from room: "general"  <-- Old render's cleanup runs FIRST!
🟢 [Setup] Connected to room: "random"          <-- New render's effect runs SECOND!

3. Component Unmounts:
🧹 [Cleanup] Disconnected from room: "random"   <-- Final unmount cleanup!
```

---

### Phase 3: The Unmount Sequence

When a component is removed from the DOM tree (e.g., `{isOpen && <Modal />}` becomes false, or navigating away):

```text
  1. Unmount Triggered ({isOpen && <Modal />} becomes false)
       |
       v
  2. Child Effect Cleanups Run (Bottom-up: Child cleanups run first)
       |
       v
  3. Parent Effect Cleanups Run (Parent useEffect return functions run)
       |
       v
  4. Real DOM Removal (Nodes removed from document tree)
       |
       v
  5. Memory Teardown (Fiber nodes, state closures, and refs garbage-collected)
```

1. **Trigger:** The component is omitted from the newly returned JSX tree.
2. **Cleanups Fire (Bottom-Up):**
   - All active `useEffect` and `useLayoutEffect` cleanup functions execute.
   - **Children cleanup functions fire before Parent cleanup functions**.
3. **Real DOM Removal:** React detaches the actual DOM nodes from the browser document.
4. **Garbage Collection:** Component Fiber instances, state bindings, and refs are cleared from memory.

---

### Parent vs. Child Execution Order

A major point of confusion in senior interviews is the interleaving order between parent and child components:

```text
  MOUNT ORDER:
  1. Parent Render Body       (Top-Down)
  2. Child Render Body        (Top-Down)
  3. Real DOM Injection
  4. Child useEffect          (Bottom-Up! Child mounts first)
  5. Parent useEffect         (Bottom-Up! Parent mounts after all children)

  UPDATE ORDER:
  1. Parent Render Body       (Top-Down)
  2. Child Render Body        (Top-Down)
  3. Real DOM Mutation
  4. Child Effect Cleanup     (Bottom-Up)
  5. Parent Effect Cleanup    (Bottom-Up)
  6. Child useEffect          (Bottom-Up)
  7. Parent useEffect         (Bottom-Up)

  UNMOUNT ORDER:
  1. Child Effect Cleanup     (Bottom-Up)
  2. Parent Effect Cleanup    (Bottom-Up)
  3. Real DOM Removal
```

> [!IMPORTANT]
> **Key Rule of Thumb:**
> - **Render Phase is TOP-DOWN (Parent -> Child):** A child cannot render until its parent executes and creates the child element in JSX.
> - **Effect Phase is BOTTOM-UP (Child -> Parent):** A parent cannot be considered fully "mounted" or "updated" until all of its children have completed mounting and running their own effects.

---

### Verification Example: The Full Lifecycle in Action

Here is a runnable example instrumented with console logs to trace the entire lifecycle:

```javascript
import { useState, useEffect } from "react";

function Parent() {
  const [showChild, setShowChild] = useState(true);
  const [count, setCount] = useState(0);

  console.log("➡️ [1] Parent Render Body (count =", count, ")");

  useEffect(() => {
    console.log("🟢 [Parent] useEffect Mounted / Updated (count =", count, ")");

    return () => {
      console.log("🧹 [Parent] useEffect Cleanup (count =", count, ")");
    };
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={() => setShowChild((s) => !s)}>Toggle Child</button>
      {showChild && <Child parentCount={count} />}
    </div>
  );
}

function Child({ parentCount }) {
  console.log("  ➡️ [2] Child Render Body (parentCount =", parentCount, ")");

  useEffect(() => {
    console.log("  🟢 [Child] useEffect Mounted / Updated (parentCount =", parentCount, ")");

    return () => {
      console.log("  🧹 [Child] useEffect Cleanup (parentCount =", parentCount, ")");
    };
  }, [parentCount]);

  return <p>Child Count: {parentCount}</p>;
}
```

#### Exact Console Output for Each Phase:

**1. Initial Mount:**
```text
➡️ [1] Parent Render Body (count = 0)
  ➡️ [2] Child Render Body (parentCount = 0)
  🟢 [Child] useEffect Mounted / Updated (parentCount = 0)
🟢 [Parent] useEffect Mounted / Updated (count = 0)
```

**2. State Update (Click "Increment"):**
```text
➡️ [1] Parent Render Body (count = 1)
  ➡️ [2] Child Render Body (parentCount = 1)
  🧹 [Child] useEffect Cleanup (parentCount = 0)
🧹 [Parent] useEffect Cleanup (count = 0)
  🟢 [Child] useEffect Mounted / Updated (parentCount = 1)
🟢 [Parent] useEffect Mounted / Updated (count = 1)
```

**3. Unmount (Click "Toggle Child"):**
```text
➡️ [1] Parent Render Body (count = 1)
  🧹 [Child] useEffect Cleanup (parentCount = 1)
```

## 5. Conditional Rendering Under the Hood: What Actually Happens to a Child

In React, conditionally rendering a component (`{show && <Child />}` or `{show ? <Child /> : null}`) does **not** merely make it invisible. It completely destroys and recreates the component's underlying internal machinery.

Understanding what React does behind the scenes prevents silent bugs like lost form data, dangling event listeners, and unexpected network storms.

---

### What Happens When `show` Toggles to `false` (Destruction)

When the parent re-renders and the conditional expression evaluates to `false` (or `null`/`undefined`), React executes a full teardown:

```text
  Parent Re-renders with show = false
                 |
                 v
  1. Reconciliation: React notices <Child /> slot is now null / false
                 |
                 v
  2. Cleanup Execution: React invokes all return () => { ... } functions
     from useEffect and useLayoutEffect inside <Child /> (Bottom-Up)
                 |
                 v
  3. Real DOM Removal: React detaches child DOM elements from browser tree
     (e.g., container.removeChild(childNode))
                 |
                 v
  4. State & Fiber Node Destruction:
     • useState hook values are ERASED from memory
     • useRef .current references to DOM nodes become null
     • The internal Fiber node representing <Child /> is destroyed
```

1. **Reconciliation Diffing:** React’s reconciler compares the previous Virtual DOM tree against the new one. Where it previously held a Fiber node of type `<Child />`, it now receives `null`.
2. **Effect Cleanup:** React fires all cleanup functions registered by the child’s `useEffect` and `useLayoutEffect`. If the component set up `setInterval`, `addEventListener`, or network subscriptions, they **must** be cleared here to prevent memory leaks.
3. **Physical DOM Deletion:** The child’s actual HTML elements are detached from the document object model. They no longer consume layout, paint, or event listener resources in the browser.
4. **Complete State Wipeout:** All internal states (`useState`), reducers (`useReducer`), and timers are deleted from memory. **React does not preserve or stash this state anywhere.**

---

### What Happens When `show` Toggles Back to `true` (Recreation)

When `show` flips back to `true`, React does **not** "wake up" or unhide the old component. It treats it as an entirely new entity born for the first time:

```text
  Parent Re-renders with show = true
                 |
                 v
  1. New Fiber Node Created: React allocates brand new memory structures
                 |
                 v
  2. State Initialization: useState(initialValue) runs from SCRATCH
     (Previous state is NOT remembered; resets to default)
                 |
                 v
  3. DOM Insertion: New physical HTML elements created & attached to DOM
                 |
                 v
  4. Mount Effects Fire: useEffect(() => {}, []) runs as a brand-new mount
```

- Any text typed into an `<input>` is gone.
- Any scroll positions are reset to top (`0, 0`).
- Any API calls configured to run on mount (`useEffect([], ...)`) will execute again.

---

### Conditional Rendering vs. CSS Hiding (`display: none`)

A frequent architectural decision is deciding between unmounting a child component vs. hiding it visually via CSS:

```text
  PATTERN A: Conditional Rendering ({show && <Child />})
  +---------------------------------------------------------------+
  | DOM: Removed | State: Destroyed | Effects: Cleaned Up         |
  +---------------------------------------------------------------+

  PATTERN B: CSS Hiding (<div style={{ display: show ? 'block' : 'none' }}>)
  +---------------------------------------------------------------+
  | DOM: Present | State: Preserved | Effects: Keep Running!      |
  +---------------------------------------------------------------+
```

| Feature | Conditional Rendering (`{show && <Child />}`) | CSS Hiding (`display: 'none'`) |
| :--- | :--- | :--- |
| **Real DOM Nodes** | Completely removed from browser DOM | Remain in the DOM tree (hidden) |
| **Component State** | **Destroyed** on unmount; resets to initial on remount | **Preserved** intact in memory |
| **`useEffect` Lifecycle** | Unmount cleanup runs; mount effect re-runs on show | **Never unmounts**; effects keep running |
| **Active Intervals / Timers** | Stop immediately (if cleaned up in effect) | **Keep ticking** in background |
| **Memory Footprint** | Low (unused components are garbage collected) | Higher (hidden DOM nodes and closures stay in RAM) |
| **Initial Toggle Speed** | Slower (must recreate DOM & run effects) | Instant (just toggles a CSS property) |

---

### Common Gotchas with Conditional Rendering

#### 1. The Falsy `0` Trap (`items.length && <List />`)
In JavaScript, `0 && <Component />` evaluates to the number `0`, not `false`. React prints raw numbers directly to the screen!

```javascript
// BUG: When items is empty, this renders "0" directly onto your UI!
<div>{items.length && <ItemList items={items} />}</div>

// FIX 1: Turn into a boolean check
<div>{items.length > 0 && <ItemList items={items} />}</div>

// FIX 2: Use ternary operator
<div>{items.length ? <ItemList items={items} /> : null}</div>
```

#### 2. Accidental Reset of User Input / Form State
When a tab or modal conditionally renders child forms, users lose their entered progress if they navigate away or close the modal:

```javascript
// PROBLEM: Closing the tab wipes out all unsaved form inputs!
function SettingsModal({ isOpen }) {
  if (!isOpen) return null;
  return <HeavyForm />; // Unmounted -> all input state lost!
}
```

#### 3. Dangling Subscriptions & Memory Leaks
If `<Child />` fails to return a cleanup function from its effects, unmounting the child leaves zombie callbacks in memory that attempt to call `setState` on an unmounted component.

---

### How to Preserve State Across Conditional Toggles

If you need a child to disappear visually without losing its internal state, use one of these three standard patterns:

```javascript
// PATTERN 1: Lift State Up to Parent
// The state lives in Parent, so it survives Child mounting and unmounting
function Parent() {
  const [formData, setFormData] = useState({ name: "" });
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && <Child formData={formData} onChange={setFormData} />}
    </>
  );
}

// PATTERN 2: CSS Visibility Hiding
// Keeps the component mounted in memory while invisible on screen
function TabView({ isVisible }) {
  return (
    <div style={{ display: isVisible ? "block" : "none" }}>
      <ExpensiveEditor />
    </div>
  );
}

// PATTERN 3: URL Parameters or Global Cache (React Query / Zustand)
// The child pulls its state from the URL/cache on mount, automatically restoring it
function SearchFilter() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get("q") || ""; // Restored immediately on mount
  return <input value={query} onChange={(e) => setSearchParams({ q: e.target.value })} />;
}
```

## 6. What Happens to a Parent When a Child State Changes?

The short answer: **Nothing!**

In React, renders flow strictly **downwards** (unidirectional data flow). A child component's internal state update never causes its parent to re-render.

```text
       +-----------------------+
       |   Parent Component    |  ---> DOES NOT RE-RENDER (Untouched)
       +-----------------------+
                   |
                   v (props flow down)
       +-----------------------+
       |    Child Component    |  <--- setState() fires here
       +-----------------------+       ONLY this child & its descendants re-render!
```

### Why React Works This Way
- **Isolated Component Boundaries:** Each component owns and encapsulates its own state. When a child calls `setState()`, React schedules a re-render starting at that child's Fiber node, skipping all ancestor components above it.
- **Performance Optimization ("Pushing State Down"):** Moving volatile state (like typing into a local `<input>` or toggling a dropdown) down into isolated leaf components prevents the entire parent tree from re-rendering needlessly.

### The ONLY Exceptions (When a Parent DOES Re-render)
A parent component will only re-render in response to a child action if that action alters state belonging to the parent:
1. **Invoking a Parent Callback:** The child calls an event function passed down via props (e.g., `props.onSelect(item)`), which calls `setState` inside the parent.
2. **Updating Shared Context / Store:** The child updates a React Context or global store (Zustand/Redux) that the parent component also subscribes to.

```javascript
import { useState } from "react";

function Parent() {
  console.log("Parent rendered"); // Runs ONLY on initial mount!

  return (
    <div>
      <h1>Heavy Dashboard</h1>
      {/* State updates inside ChildCounter stay strictly inside ChildCounter */}
      <ChildCounter />
    </div>
  );
}

function ChildCounter() {
  const [count, setCount] = useState(0);
  console.log("Child rendered:", count); // Runs on every button click!

  return <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>;
}
```

## 7. Effect Cleanup on Dependency Changes: API Aborts & Event Listeners

Whenever a dependency in `useEffect(..., [dependency])` changes, React executes the **previous effect's cleanup function before running the new effect**.

This cleanup is not just for component unmounting — it is React’s primary defense against **network race conditions** and **memory leaks** during normal component updates.

---

### Case 1: Canceling In-Flight API Requests (Preventing Race Conditions)

#### The Problem: The Out-of-Order Response Bug
When a user rapidly switches between items (e.g., clicking from User 1 to User 2), two network requests are fired in quick succession. If Request 1 takes 2 seconds (slow network) and Request 2 takes 200 milliseconds (fast cache), Request 1 will resolve **last**, overwriting User 2's screen with outdated data from User 1!

```text
  User clicks "User 1":  Fetch(User 1) starts -----------------------------> Resolves LAST! (State overwritten with User 1!)
                               \
  User clicks "User 2":         Fetch(User 2) starts ---> Resolves FIRST!
```

#### How the Cleanup Fixes It (Using `AbortController`)
When `userId` changes from 1 to 2, React immediately runs the cleanup returned by Render 1. That cleanup calls `controller.abort()`, canceling the in-flight HTTP request in the browser.

```text
  Render 1 (userId = 1):
    1. Creates controller_1
    2. Starts fetch("/api/user/1", { signal: controller_1.signal })
    3. Saves cleanup: () => controller_1.abort()
             |
             | (userId prop changes to 2)
             v
  Render 2 (userId = 2):
    1. React runs cleanup from Render 1: controller_1.abort()
       --> Browser instantly CANCELS request for User 1!
    2. Creates controller_2
    3. Starts fetch("/api/user/2", { signal: controller_2.signal })
    4. Only User 2's data sets state!
```

#### The Code:
```javascript
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // 1. Instantiate an AbortController for THIS render's request
    const controller = new AbortController();

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      signal: controller.signal, // Link request to the abort signal
    })
      .then((res) => res.json())
      .then((data) => {
        console.log(`✅ Loaded data for user ${userId}`);
        setUser(data);
      })
      .catch((err) => {
        if (err.name === "AbortError") {
          console.log(`🛑 Canceled in-flight request for user ${userId}`);
        } else {
          console.error("Fetch error:", err);
        }
      });

    // 2. Cleanup runs when userId changes OR component unmounts
    return () => {
      // Aborts the previous fetch before the next one starts
      controller.abort();
    };
  }, [userId]);

  return <div>{user ? user.name : "Loading..."}</div>;
}
```

---

### Case 2: Event Listener Teardown (Preventing Listener Accumulation)

#### The Problem: The Multiplied Listener Trap
If your event listener relies on a changing prop or state (like a hotkey handler), you must include that variable in the dependency array. 

Without a cleanup function, **every single render adds another event listener to `window`**. After 5 renders, a single keypress triggers the callback 5 times!

```text
  Render 1: window.addEventListener("keydown", fn1)  --> 1 active listener
  Render 2: window.addEventListener("keydown", fn2)  --> 2 active listeners!
  Render 3: window.addEventListener("keydown", fn3)  --> 3 active listeners!
  (User presses a key: ALL 3 listeners fire simultaneously!)
```

#### How the Cleanup Fixes It
When the dependency changes, React first executes `window.removeEventListener("keydown", fn1)` using the *exact function reference* stored from Render 1, then attaches `fn2`.

```text
  Render 1 (step = 1):
    1. Creates handleKeyDown_1 (captures step = 1)
    2. window.addEventListener("keydown", handleKeyDown_1)
             |
             | (step updates to 5)
             v
  Render 2 (step = 5):
    1. Cleanup: window.removeEventListener("keydown", handleKeyDown_1) (Removed!)
    2. Creates handleKeyDown_2 (captures step = 5)
    3. window.addEventListener("keydown", handleKeyDown_2) (Attached!)
    Result: ALWAYS exactly ONE listener active at any time.
```

#### The Code:
```javascript
import { useState, useEffect } from "react";

function KeyStepper({ step }) {
  const [value, setValue] = useState(0);

  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === "ArrowUp") {
        // Uses the current 'step' value captured in closure
        setValue((v) => v + step);
      }
    };

    console.log(`🟢 Attached keydown listener with step = ${step}`);
    window.addEventListener("keydown", handleKeyDown);

    // Runs BEFORE attaching a new listener on step change
    return () => {
      console.log(`🧹 Removed keydown listener with step = ${step}`);
      window.removeEventListener("keydown", handleKeyDown);
    };
  }, [step]); // Re-subscribes whenever 'step' changes

  return <div>Value: {value} (Step size: {step})</div>;
}
```

---

### Summary Checklist: What Happens on Dependency Change

| Step | Operation | What Happens to Network / DOM |
| :--- | :--- | :--- |
| **1. Prop/State Change** | Component re-evaluates | Virtual DOM diffed; Real DOM updated |
| **2. Browser Paint** | Screen updates | User sees new DOM on screen |
| **3. Previous Cleanup Runs** | Stored `return () => {}` executes | `controller.abort()` cancels old HTTP request; `removeEventListener` detaches old listener |
| **4. New Effect Runs** | New `useEffect` callback executes | New `fetch()` initiated; new `addEventListener` attached with fresh closure |




