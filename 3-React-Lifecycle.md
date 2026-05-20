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
    - [2. The Double Mount (The "Ghost Trigger")](#2-the-double-mount-the-ghost-trigger)
    - [3. Referential Inequality (The "Infinite Loop" Trap)](#3-referential-inequality-the-infinite-loop-trap)
    - [4. Batched State Updates (The "Lost Math")](#4-batched-state-updates-the-lost-math)
    - [5. The Render-Phase Update (The "Accidental Lifecycle")](#5-the-render-phase-update-the-accidental-lifecycle)

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
2. **Updating:** The user types something, fetches data, or receives new props. React re-runs the component function, computes the differences (the diffing algorithm), and updates *only* the modified parts of the DOM.
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

If you leave the dependency array completely empty (no brackets at all), the effect fires on the initial mount and after *every single update* to the screen.

```javascript
import { useEffect, useState } from 'react';

function LogEveryRender() {
  const [text, setText] = useState('');

  useEffect(() => {
    // Triggers on mount AND whenever "text" or any other state triggers a re-render
    console.log('The component rendered or updated!');
  });

  return <input value={text} onChange={(e) => setText(e.target.value)} />;
}

```

### Pattern B: Run ONLY on Initial Mount

Adding an empty array `[]` tells React: "This effect does not depend on any variables from the component scope." It runs exactly once when the component first appears.

```javascript
import { useEffect, useState } from 'react';

function UserProfile() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Triggers EXACTLY ONCE when the component is added to the screen
    fetch('/api/user/1')
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, []); // Empty array = mount only

  return <div>{user ? user.name : 'Loading user...'}</div>;
}

```

### Pattern C: Run on Mount AND when Specific Dependencies Change

Passing variables inside the array `[dependency1, dependency2]` tells React to skip executing the effect unless one of those values changes between renders.

```javascript
import { useEffect, useState } from 'react';

function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    // Triggers on mount AND whenever the "query" prop changes value
    fetch(`/api/search?q=${query}`)
      .then((res) => res.json())
      .then((data) => setResults(data));
  }, [query]); // Re-runs ONLY when query changes

  return <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>;
}

```

### Pattern D: Clean Up Before Unmount (The Cleanup Function)

If your effect returns a function, React will execute that function right before the component is destroyed (unmounted), or immediately before re-running the effect on a new update. This prevents memory leaks.

```javascript
import { useEffect, useState } from 'react';

function WindowTracker() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    // 1. Setup subscription on mount
    window.addEventListener('resize', handleResize);

    // 2. Return a cleanup function
    return () => {
      // Cleans up the event listener when the component disappears
      window.removeEventListener('resize', handleResize);
      console.log('Cleaned up window listener');
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

* **The Old Class Way:**

```javascript
class OldCounter extends React.Component {
  constructor(props) {
    super(props);
    // State initialization
    this.state = { count: 0 };
    // Event binding required for class methods
    this.increment = this.increment.bind(this);
  }
  increment() { this.setState({ count: this.state.count + 1 }); }
  render() { return <button onClick={this.increment}>{this.state.count}</button>; }
}

```

* **The Modern Hooks Way:**

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

* **The Old Class Way:**

```javascript
class OldFetch extends React.Component {
  componentDidMount() {
    console.log("Component is mounted to the actual screen!");
    // Trigger side effect
  }
  render() { return <div>Hello World</div>; }
}

```

* **The Modern Hooks Way:**

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

* **The Old Class Way:**

```javascript
class OldSync extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    // You had to manually write manual conditions to see if data changed
    if (prevProps.userId !== this.props.userId) {
      this.fetchNewUserData(this.props.userId);
    }
  }
  render() { return <div>User UI</div>; }
}

```

* **The Modern Hooks Way:**

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

* **The Old Class Way:**

```javascript
class OldClock extends React.Component {
  componentDidMount() {
    this.timerId = setInterval(() => this.tick(), 1000);
  }
  componentWillUnmount() {
    // Clean up interval before destruction
    clearInterval(this.timerId);
  }
  render() { return <div>Clock UI</div>; }
}

```

* **The Modern Hooks Way:**

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

* **The Old Class Way:**

```javascript
class OldPureCard extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Return true if you want a re-render, false if you want to skip it
    return nextProps.title !== this.props.title;
  }
  render() { return <h1>{this.props.title}</h1>; }
}

```

* **The Modern Hooks Way:**

```javascript
import React from 'react';

// Wrap the whole component function in React.memo()
const ModernPureCard = React.memo(function ModernPureCard({ title }) {
  return <h1>{title}</h1>;
}); 
// React.memo automatically performs a shallow comparison of props 
// and skips re-rendering if props are unchanged.

```

### F. Rare Case: `getDerivedStateFromProps`

This was a rare, complex class method used to update internal component state when incoming props changed over time. React core maintainers recommend bypassing this altogether in functional components by setting state directly during the rendering cycle.

* **The Modern Functional Way:**

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
  +--------------------+         +-----------------------------------+
  | Render 1 (count=0) | ----> | useEffect() runs                  |
  +--------------------+         | Creates interval()                |
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
      console.log('Count is:', count); 
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
setCount(prevCount => prevCount + 1); 

```

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
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(setData);

  return () => {
    // Cancels the fetch if the component unmounts (or during Strict Mode's ghost unmount)
    controller.abort(); 
  };
}, []);

```

### 3. Referential Inequality (The "Infinite Loop" Trap)

**The Question:** I passed an array/object to a child component, and now `useEffect` is caught in an infinite loop, crashing the browser. What happened?

**The Gotcha:** In JavaScript, `[1, 2] === [1, 2]` is `false` because they occupy different memory addresses. If you create a new array, object, or function directly inside the render body, it gets a new memory address on *every single render*. If that object is used in a dependency array, it triggers the effect, which updates state, which triggers a render, which creates a new object...

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
  const fetchOptions = { method: 'GET' }; 

  useEffect(() => {
    fetch('/api', fetchOptions).then(setData);
  }, [fetchOptions]); // Triggered continuously!
}

```

**The Solution:** Move static objects *outside* the component entirely so they are created once, or wrap them in `useMemo` (or `useCallback` for functions) so React caches the memory reference.

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
  setCount(prev => prev + 1); // Receives 0, returns 1
  setCount(prev => prev + 1); // Receives 1, returns 2
  setCount(prev => prev + 1); // Receives 2, returns 3
};

```

### 5. The Render-Phase Update (The "Accidental Lifecycle")

**The Question:** I'm getting a console warning: *"Cannot update a component while rendering a different component."* What causes this?

**The Gotcha:** You called a `setState` function directly in the main body of your component function, rather than inside a `useEffect` or an event handler (like `onClick`). Because calling `setState` tells React to re-render, calling it *during* the render tells React to start over immediately, causing a rapid loop.

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

**The Solution:** State updates must happen as a reaction to something. They either belong in an event handler (reaction to a user) or a `useEffect` (reaction to a lifecycle/prop change).

