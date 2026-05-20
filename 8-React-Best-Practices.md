# React Best Practices

- [React Best Practice](#react-best-practices)
  - [1. JSX Best Practices: Writing Clean Markup](#1-jsx-best-practices-writing-clean-markup)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [2. Props Best Practices: Component Communication](#2-props-best-practices-component-communication)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [3. Rendering Lists: Keys and Performance](#3-rendering-lists-keys-and-performance)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [4. Interactivity Best Practices: Events and Handlers](#4-interactivity-best-practices-events-and-handlers)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [5. State Management: Sharing, Preserving, and useReducer](#5-state-management-sharing-preserving-and-usereducer)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [6. Updating Arrays and Objects Best Practices](#6-updating-arrays-and-objects-best-practices)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [7. Queueing a Series of State Updates](#7-queueing-a-series-of-state-updates)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)
  - [8. Form Handling: Controlled vs Uncontrolled](#8-form-handling-controlled-vs-uncontrolled)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
  - [9. Rules of React: Hooks](#9-rules-of-react-hooks)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
  - [10. Component Best Practices: Pure Components](#10-component-best-practices-pure-components)
    - [The Why, What, and How](#the-why-what-and-how)
    - [When to Use and Gotchas](#when-to-use-and-gotchas)
    - [Code Examples](#code-examples)
    - [Mastery & Interview Questions](#mastery-interview-questions)

This is a comprehensive deep dive into React's core best practices, mechanics, and common pitfalls. We will break down each of the 10 areas with architectural models, code, and interview-level insights.

## 1. JSX Best Practices: Writing Clean Markup

### The Why, What, and How

* **What it is:** JSX is a syntax extension for JavaScript that looks like HTML. It is syntactic sugar that compiles down to `React.createElement()` or (in modern React) the `_jsx` runtime.
* **Why it exists:** It allows developers to write UI logic and markup in the same file, making components highly cohesive.
* **How to use it:** Return a single root element, close all tags (even self-closing ones like `<img />`), and use camelCase for attributes.

```text
      JSX COMPILATION PIPELINE
      ------------------------
  <div className="box">Hello</div>
                 |
             (Babel)
                 |
  _jsx('div', { className: 'box', children: 'Hello' })
                 |
          React Virtual DOM Node

```

### When to Use and Gotchas

* **Gotchas:**
* Forgetting that JSX is JavaScript. You cannot use reserved JS keywords like `class` (use `className`) or `for` (use `htmlFor`).
* Returning multiple elements without a single wrapper. React functions must return a single object. Use `<Fragment>` or `<>...</>` to group elements without adding extra DOM nodes.



### Code Examples

```jsx
// ❌ BAD: Returning multiple siblings, using 'class'
const BadComponent = () => {
  return (
    <h1 class="title">Hello</h1>
    <p>World</p>
  ); // Syntax Error!
};

// ✅ GOOD: Wrapped in a Fragment, camelCase attributes
const GoodComponent = () => {
  return (
    <>
      <h1 className="title">Hello</h1>
      <p>World</p>
    </>
  );
};

```

### Mastery & Interview Questions

* **Interview Question:** *Why do we have to use `className` instead of `class` in JSX?*
* **Answer:** JSX is compiled into JavaScript objects. `class` is a reserved keyword in JavaScript (used for ES6 classes). While modern parsers can technically handle it now, `className` maps directly to the DOM API's `Element.className` property, keeping the abstraction consistent.



## 2. Props Best Practices: Component Communication

### The Why, What, and How

* **What it is:** Props (properties) are the mechanism for passing read-only data from a parent component to a child component.
* **Why it exists:** To make components reusable and decoupled.
* **How to use it:** Pass data as attributes in JSX, and receive them as an object argument in the child function. Always destructure them for readability.

```text
      DATA FLOW (ONE-WAY BINDING)
      ---------------------------
            [Parent (State)]
             |          |
      (prop: user)   (prop: onUpdate)
             |          |
             V          V
        [Child (UI)]  [Child (Button)]

```

### When to Use and Gotchas

* **Gotchas:**
* **Prop Drilling:** Passing props down 5+ levels just to reach a deeply nested child. Avoid this by using Context or component composition (passing children).
* **Mutating Props:** Props are strictly read-only. Trying to reassign a prop (`props.name = 'John'`) will throw an error in strict mode and violate React's core principles.



### Code Examples

```jsx
// ✅ GOOD: Destructuring and Default Values
const Avatar = ({ imageSrc, size = 50, altText }) => {
  return <img src={imageSrc} width={size} alt={altText} />;
};

// ✅ GOOD: Component Composition (Avoids Prop Drilling)
const Layout = ({ sidebar, content }) => (
  <div className="layout">
    <aside>{sidebar}</aside>
    <main>{content}</main>
  </div>
);

```

### Mastery & Interview Questions

* **Interview Question:** *How do you pass data from a child back to a parent?*
* **Answer:** You pass a callback function from the parent to the child as a prop. The child invokes this callback, passing the data as arguments, which updates the parent's state.



## 3. Rendering Lists: Keys and Performance

### The Why, What, and How

* **What it is:** Transforming an array of data into an array of JSX elements using JavaScript's `.map()` method.
* **Why it exists:** To render dynamic collections of data.
* **How to use it:** Every element generated inside the `map()` must have a unique, stable `key` prop attached to the outermost element being returned.

```text
      WHY KEYS MATTER (DOM DIFFING)
      -----------------------------
   No Keys (Bad)          With Keys (Good)
   -------------          ----------------
   [A, B, C]              [{id:1, v:A}, {id:2, v:B}]
   Insert 'X' at top      Insert 'X' at top
   
   React mutates A->X     React sees id:3 is new.
   React mutates B->A     React leaves id:1 alone.
   React mutates C->B     React leaves id:2 alone.
   React appends C        (Massive performance gain)

```

### When to Use and Gotchas

* **Gotchas:**
* **Using `index` as a key:** If the list can be reordered, filtered, or have items inserted/deleted, using the array index as a key will cause bugs. React will match the old index 0 to the new index 0, potentially keeping the wrong DOM state (like a checked checkbox) attached to the wrong data item.



### Code Examples

```jsx
// ❌ BAD: Using index as key for a dynamic list
const BadList = ({ items }) => (
  <ul>
    {items.map((item, index) => <li key={index}>{item.name}</li>)}
  </ul>
);

// ✅ GOOD: Using a stable, unique ID
const GoodList = ({ items }) => (
  <ul>
    {items.map(item => <li key={item.databaseId}>{item.name}</li>)}
  </ul>
);

```

### Mastery & Interview Questions

* **Interview Question:** *When is it acceptable to use the array index as a key?*
* **Answer:** Only when three conditions are met: 1) The list and items are static (they will never change, reorder, or be filtered). 2) The items have no unique IDs. 3) The list is never reordered or filtered.



## 4. Interactivity Best Practices: Events and Handlers

### The Why, What, and How

* **What it is:** Handling user interactions (clicks, typing, hovering) using React's Synthetic Event system.
* **Why it exists:** Cross-browser compatibility. React wraps native browser events in a unified API so `onChange` works exactly the same in Chrome, Safari, and Firefox.
* **How to use it:** Pass a function reference to event props like `onClick` or `onSubmit`.

```text
      EVENT DELEGATION MODEL
      ----------------------
      [ Document Root ] <--- React attaches ONE event listener here
            |
        [ Parent ]
            |
        [ Button ] <--- User clicks
            
   (React intercepts at root, determines target, and fires your onClick prop)

```

### When to Use and Gotchas

* **Gotchas:**
* **Calling the function instead of passing it:** Writing `onClick={handleClick()}` will execute the function immediately during render, often causing infinite loops if it sets state. It must be `onClick={handleClick}` or `onClick={() => handleClick(id)}`.
* **Stale Closures:** If your event handler is wrapped in a `useCallback` with missing dependencies, it might read outdated state variables.



### Code Examples

```jsx
// ❌ BAD: Function called on render
<button onClick={deleteItem(id)}>Delete</button>

// ✅ GOOD: Function passed as a reference
<button onClick={() => deleteItem(id)}>Delete</button>

// ✅ GOOD: Extracted handler for complex logic
const Form = () => {
  const handleSubmit = (e) => {
    e.preventDefault(); // Stop browser reload
    submitData();
  };
  return <form onSubmit={handleSubmit}>...</form>;
};

```

### Mastery & Interview Questions

* **Interview Question:** *What is a Synthetic Event in React?*
* **Answer:** It is a cross-browser wrapper around the browser's native event. React attaches a single event listener to the document root (event delegation) to improve memory performance, rather than attaching listeners to every individual DOM node.



## 5. State Management: Sharing, Preserving, and useReducer

### The Why, What, and How

* **What it is:** Managing data that changes over time and affects what is rendered on screen.
* **How it works:**
* **Sharing:** To share state between siblings, you "lift state up" to their closest common parent.
* **Preserving:** React ties state to a component's position in the UI tree. If a component unmounts or changes position, the state is destroyed.
* **useReducer:** A Hook for managing complex state objects with multiple sub-values, using a Redux-like action/dispatch pattern.



```text
      LIFTING STATE UP
      ----------------
         [Parent] ---> Holds `isOpen` state
         /      \
    [Button]   [Modal]
  (Toggles)    (Reads)

      STATE PRESERVATION BY POSITION
      ------------------------------
      {isToggled ? <Counter /> : <Counter />}
      // Gotcha: Because BOTH are a <Counter> at the exact 
      // same tree position, React PRESERVES the state when toggling!

```

### When to Use and Gotchas

* **Gotchas:**
* **Forcing State Reset:** If you *want* the `<Counter>` above to reset its state when toggled, you must pass a different `key` prop to each one. `key` tells React "these are conceptually different components."



### Code Examples

```jsx
// ✅ GOOD: Using useReducer for complex, interdependent state
const initialState = { count: 0, error: null };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1, error: null };
    case 'decrement':
      if (state.count === 0) return { ...state, error: 'Cannot go below 0' };
      return { count: state.count - 1, error: null };
    default:
      return state;
  }
}

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <span>{state.count}</span>
      {state.error && <p>{state.error}</p>}
    </>
  );
};

```

### Mastery & Interview Questions

* **Machine Coding Challenge:** *Build a component that toggles between two different user profiles, ensuring the input fields reset completely when the user switches.*
* **Solution:** Render `<ProfileForm key="{user.id}" user="{user}"/>`. The `key` forces React to destroy the old DOM nodes and state and mount fresh ones.



## 6. Updating Arrays and Objects Best Practices

### The Why, What, and How

* **What it is:** The strict rule that React state must be treated as **immutable**.
* **Why it exists:** React determines if a component needs to re-render by doing a strict equality check (`oldState === newState`). If you mutate an object, the memory address stays the same, the equality check returns `true`, and React skips the render.
* **How to use it:** Always create *new* arrays or objects when updating state, usually using the spread operator (`...`) or array methods like `map` and `filter`.

```text
      MUTATION VS IMMUTABILITY
      ------------------------
      // Mutation (React won't see this)
      Pointer A ---> [ Memory: { name: 'Dan' } ] 
                     (changes to 'Bob', pointer remains same)

      // Immutability (React triggers render)
      Pointer A ---> [ Memory: { name: 'Dan' } ]
      Pointer B ---> [ Memory: { name: 'Bob' } ] (New Object)

```

### When to Use and Gotchas

* **Gotchas:**
* **Nested Objects:** The spread operator only does a *shallow copy*. If you have a nested object, you must copy the inner object as well.
* **Banned Array Methods:** Never use `push()`, `pop()`, `splice()`, or `sort()` directly on state arrays. They mutate. Use `concat()`, `slice()`, `filter()`, or copy the array first (`[...arr].sort()`).



### Code Examples

```jsx
// ❌ BAD: Mutating state directly
const badUpdate = () => {
  user.address.city = 'London'; // Mutates!
  setUser(user); // React ignores this update
};

// ✅ GOOD: Immutability with nested objects
const goodUpdate = () => {
  setUser(prevUser => ({
    ...prevUser, // Copy outer object
    address: {
      ...prevUser.address, // Copy inner object
      city: 'London' // Overwrite target property
    }
  }));
};

```

### Mastery & Interview Questions

* **Interview Question:** *Why does React rely on immutability?*
* **Answer:** Performance. Doing a deep comparison of two large objects to find changes is computationally expensive (O(N)). By enforcing immutability, React only needs to do an O(1) strict equality check (`prev !== next`) by comparing memory addresses.



## 7. Queueing a Series of State Updates

### The Why, What, and How

* **What it is:** React 18+ automatically batches all state updates within the same event loop into a single re-render for performance.
* **Why it exists:** To prevent multiple UI flickers and unnecessary calculations if you call `setState` multiple times in one function.
* **How to use it:** When the next state depends on the previous state, use an **updater function** (`prev => prev + 1`) instead of passing the raw value.

```text
      STATE UPDATE QUEUE
      ------------------
   onClick handler runs:
   setCount(count + 1) ---> Queues "replace with 1"
   setCount(count + 1) ---> Queues "replace with 1"
   setCount(count + 1) ---> Queues "replace with 1"
   Result: Renders once with count = 1.

   Using Updater Function:
   setCount(c => c + 1) ---> Queues "take previous + 1"
   setCount(c => c + 1) ---> Queues "take previous + 1"
   setCount(c => c + 1) ---> Queues "take previous + 1"
   Result: Renders once with count = 3.

```

### When to Use and Gotchas

* **Gotchas:** Believing `setState` is synchronous. If you call `setCount(5)`, you cannot `console.log(count)` on the very next line and expect to see 5. The variable `count` is a constant snapshot of the *current* render.

### Code Examples

```jsx
const Counter = () => {
  const [score, setScore] = useState(0);

  const incrementThrice = () => {
    // ✅ GOOD: Next state relies on the exact previous state
    setScore(s => s + 1);
    setScore(s => s + 1);
    setScore(s => s + 1);
  };

  return <button onClick={incrementThrice}>Score: {score}</button>;
};

```

### Mastery & Interview Questions

* **Interview Question:** *Explain how closure affects `useState` values during a render cycle.*
* **Answer:** A component function is a snapshot in time. The `count` variable is locked to the value it had when the render started. Any asynchronous code (like a `setTimeout`) inside that render will "see" that specific snapshot's value, not any newly updated values, unless you use a ref or an updater function.



## 8. Form Handling: Controlled vs Uncontrolled

### The Why, What, and How

* **What it is:** Two paradigms for managing form data.
* **Controlled:** React state is the "single source of truth". The input's `value` is bound to a state variable.
* **Uncontrolled:** The DOM itself maintains the data. You read it when needed using a `ref`.



```text
      CONTROLLED COMPONENT LOOP
      -------------------------
   [ State: text = "A" ] -----> (Renders `<input value="A">`)
           ^                                |
           |                                V
   (Calls `setText("Ab")`) <--- (User types "b", triggers onChange)

```

### When to Use and Gotchas

* **Good for (Controlled):** Instant validation, masking inputs (like formatting a credit card), disabling submit buttons conditionally.
* **Good for (Uncontrolled):** Integrating with non-React libraries, handling file uploads (`<input type="file">` must be uncontrolled due to browser security), and extremely simple forms to reduce boilerplate.
* **Gotchas:** Setting an input to `value={state}` without an `onChange` handler will make the input permanently read-only and throw a React console warning.

### Code Examples

```jsx
// ✅ CONTROLLED (Recommended for complex UI)
const ControlledForm = () => {
  const [name, setName] = useState('');
  
  return (
    <input 
      value={name} 
      onChange={(e) => setName(e.target.value)} 
    />
  );
};

// ✅ UNCONTROLLED (Great for performance or simple forms)
const UncontrolledForm = () => {
  const inputRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(inputRef.current.value); // Read direct from DOM
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} type="text" />
      <button type="submit">Submit</button>
    </form>
  );
};

```

## 9. Rules of React: Hooks

### The Why, What, and How

* **What it is:** The strict architectural laws governing how Hooks (`useState`, `useEffect`, etc.) operate.
* **Rule 1:** Only call Hooks at the **top level**. Do not call them inside loops, conditions, or nested functions.
* **Rule 2:** Only call Hooks from **React Function Components** or Custom Hooks.
* **Why it exists:** React relies on the exact *call order* of Hooks to map state to the correct variables across re-renders.

```text
      HOOK LINKED LIST (Under the hood)
      ---------------------------------
   Render 1:
   1. useState('Dan')  ---> Hook node 1
   2. useEffect(...)   ---> Hook node 2
   3. useState(25)     ---> Hook node 3

   Render 2: (React matches by index)
   1. useState sees node 1 -> returns 'Dan'
   2. useEffect sees node 2 -> runs effect
   3. useState sees node 3 -> returns 25

```

### When to Use and Gotchas

* **Gotchas:** If you put a Hook inside an `if` statement, and that condition evaluates to `false` on the second render, React skips that Hook. Every subsequent Hook is now matched to the wrong internal node. Your app will crash with "Rendered fewer hooks than expected."

### Code Examples

```jsx
// ❌ BAD: Conditional Hook Call
const BadComponent = ({ show }) => {
  if (show) {
    const [name, setName] = useState(''); // Breaks React!
  }
  const [age, setAge] = useState(0); 
  return <div>...</div>;
};

// ✅ GOOD: Return early AFTER all hooks
const GoodComponent = ({ show }) => {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  
  if (!show) return null; // Safe, hooks already registered
  
  return <div>...</div>;
};

```

## 10. Component Best Practices: Pure Components

### The Why, What, and How

* **What it is:** A component is considered "pure" if it behaves like a mathematical function: given the exact same inputs (props and state), it always returns the exact same JSX, and it causes no side effects during the render phase.
* **Why it exists:** Predictability and performance. If a component is pure, React can safely skip rendering it (using `React.memo`) when its inputs haven't changed. It is also required for React 18's Concurrent Mode to work correctly.

```text
      THE REACT EQUATION
      ------------------
           UI = f(state)
           
   If 'f' mutates global variables, makes API calls directly, 
   or relies on Math.random() without a hook, the equation breaks.

```

### When to Use and Gotchas

* **Gotchas (Side Effects in Render):** Never fetch data, modify local storage, or mutate an external array directly inside the component body. All of these are "side effects" and must be placed inside `useEffect` or an event handler.
* **Gotchas (Object references in props):** Passing an inline object `style={{ margin: 0 }}` or inline function to a `React.memo` wrapped component will break its purity, because `{ margin: 0 } === { margin: 0 }` is false in JavaScript (different memory addresses).

### Code Examples

```jsx
let guestCount = 0;

// ❌ BAD: Impure Component (Mutates an external variable)
const BadGuest = () => {
  guestCount = guestCount + 1; // Side effect during render!
  return <div>Guest #{guestCount}</div>;
};

// ✅ GOOD: Pure Component (Relies only on inputs)
const GoodGuest = ({ index }) => {
  return <div>Guest #{index}</div>;
};

```

### Mastery & Interview Questions

* **Interview Question:** *Why is it dangerous to have side effects in the render body of a component in React 18+?*
* **Answer:** In Concurrent Mode, React might pause, abort, or restart rendering a component tree. If your component body contains side effects (like mutating an external variable), that mutation might happen multiple times before the UI is actually committed to the screen, leading to corrupted global state. Always isolate side effects to `useEffect` or event handlers.


