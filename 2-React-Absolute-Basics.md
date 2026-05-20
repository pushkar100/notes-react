# React Absolute Basics

Here is a comprehensive, visually-driven breakdown of modern React (18+) basics.

- [React Absolute Basics](#react-absolute-basics)
  - [1. Introduction / Quick Start](#1-introduction-quick-start)
  - [2. Creating Components & Nesting Them](#2-creating-components-nesting-them)
  - [3. Writing Markup with JSX](#3-writing-markup-with-jsx)
  - [4. Adding Styles](#4-adding-styles)
  - [5. Displaying Data](#5-displaying-data)
  - [6. Conditional Rendering](#6-conditional-rendering)
  - [7. Rendering Lists](#7-rendering-lists)
  - [8. Responding to Events](#8-responding-to-events)
  - [9. Updating the Screen (State)](#9-updating-the-screen-state)
  - [10. Using Hooks](#10-using-hooks)
  - [11. Sharing Data Between Components (Props)](#11-sharing-data-between-components-props)
  - [12. React Developer Tools](#12-react-developer-tools)
  - [13. Creating Interactive Components](#13-creating-interactive-components)
  - [14. Thinking in React](#14-thinking-in-react)
    - [The Mockup](#the-mockup)
    - [Step 1: Break the UI into a component hierarchy](#step-1-break-the-ui-into-a-component-hierarchy)
    - [Step 2: Build a static version in React](#step-2-build-a-static-version-in-react)
    - [Step 3: Find the minimal but complete representation of UI state](#step-3-find-the-minimal-but-complete-representation-of-ui-state)
    - [Step 4: Identify where your state should live](#step-4-identify-where-your-state-should-live)
    - [Step 5: Add inverse data flow](#step-5-add-inverse-data-flow)

## 1. Introduction / Quick Start

React is a JavaScript library for building user interfaces out of individual pieces called **components**. It abstracts away manual DOM manipulation, allowing you to declare what the UI *should* look like based on current data.

```text
  +-----------------+       +----------------+       +-------------+
  | Application Data| ----> | React Elements | ----> | Browser DOM |
  |     (State)     |       |   (Virtual)    |       |  (Actual)   |
  +-----------------+       +----------------+       +-------------+

```

```javascript
import { createRoot } from 'react-dom/client';

// The root is the DOM element where React takes over
const root = createRoot(document.getElementById('root'));

// Rendering the main App component into the root
root.render(<App />);

```

## 2. Creating Components & Nesting Them

A React component is just a JavaScript function that returns markup. Components can be nested inside other components to build complex UIs.

```text
    <App>
      ├── <Header>
      │     └── <Logo>
      └── <MainContent>
            ├── <Sidebar>
            └── <Feed>

```

```javascript
function CustomButton() {
  // Component names MUST start with a capital letter
  return <button>Click me</button>;
}

export default function App() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      {/* Nesting the CustomButton inside App */}
      <CustomButton /> 
    </div>
  );
}

```

## 3. Writing Markup with JSX

JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside a JS file.

**JSX Rules:**

1. Return a single root element (wrap multiple elements in `<>...</>` Fragments).
2. Close all tags (e.g., `<img />`).
3. camelCase most attributes (e.g., `className` instead of `class`, `strokeWidth` instead of `stroke-width`).

```javascript
function Profile() {
  return (
    // Fragment (<>) groups elements without adding a DOM node
    <>
      <h1 className="title">John Doe</h1>
      {/* Self-closing tag is mandatory for void elements */}
      <img src="/avatar.png" alt="John" />
    </>
  );
}

```

## 4. Adding Styles

In React, you specify a CSS class with `className` (since `class` is a reserved word in JS). You can also pass inline styles as JavaScript objects.

```text
 CSS File: .btn { color: red; }  ---> JSX: className="btn"
 JS Object: { color: 'red' }     ---> JSX: style={{ color: 'red' }}

```

```javascript
import './styles.css'; // Assume this file defines .card

function StyledCard() {
  // Inline styles use camelCase keys
  const headerStyle = {
    backgroundColor: 'black',
    color: 'white'
  };

  return (
    <div className="card">
      {/* First {} evaluates JS, second {} is the object */}
      <h2 style={headerStyle}>Card Title</h2>
      <p style={{ marginTop: '10px' }}>Content goes here.</p>
    </div>
  );
}

```

## 5. Displaying Data

JSX lets you embed JavaScript expressions inside curly braces `{}`.

```javascript
function WelcomeMessage() {
  const user = { name: 'Alice', role: 'Admin' };
  
  return (
    <div>
      {/* Evaluates the JS expression and injects the string */}
      <h1>Hello, {user.name}!</h1>
      <p>Role: {user.role.toUpperCase()}</p>
    </div>
  );
}

```

## 6. Conditional Rendering

There is no special syntax for conditions in React. You use standard JavaScript like `if` statements, the logical `&&` operator, or the ternary operator `? :`.

```text
 [Condition True?] --- Yes ---> Render Component A
         |
         +------------ No  ---> Render Component B (or nothing)

```

```javascript
function AdminPanel({ isAdmin }) {
  // 1. Ternary operator (condition ? true : false)
  return (
    <div>
      {isAdmin ? <Dashboard /> : <AccessDenied />}
      
      {/* 2. Logical && (renders Button only if isAdmin is true) */}
      {isAdmin && <SettingsButton />}
    </div>
  );
}

```

## 7. Rendering Lists

You will rely on JavaScript features like the `map()` array method to render lists of components. Every item in a list MUST have a unique `key` prop so React can efficiently track updates.

```text
 Array: [ "A", "B", "C" ]
           |    |    |
          map() map() map()
           v    v    v
 UI:    <li>A</li> <li>B</li> <li>C</li>

```

```javascript
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];

function ShoppingList() {
  const listItems = products.map(product => (
    // 'key' must be unique among siblings
    <li key={product.id}>
      {product.title}
    </li>
  ));

  return <ul>{listItems}</ul>;
}

```

## 8. Responding to Events

You can respond to user interactions by passing a function as a prop to an element.

```javascript
function AlertButton() {
  // Define the event handler
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    // Pass the function (don't call it with ()!)
    <button onClick={handleClick}>
      Click me
    </button>
  );
}

```

## 9. Updating the Screen (State)

To make a component "remember" things (like user input or toggle status) and re-render when they change, add **state**.

```text
  User Clicks -> setState(newData) -> Component Re-evaluates -> UI Updates

```

```javascript
import { useState } from 'react';

function Counter() {
  // useState returns an array: [currentValue, setterFunction]
  const [count, setCount] = useState(0); // 0 is the initial state

  function increment() {
    // Calling the setter schedules a UI update
    setCount(count + 1);
  }

  return <button onClick={increment}>Clicked {count} times</button>;
}

```

## 10. Using Hooks

Functions starting with `use` are called **Hooks**. `useState` is a built-in Hook. Another common one is `useEffect`, which lets you perform side effects (like data fetching or subscribing to events) when a component mounts or when certain state changes.

**Rules of Hooks:**

1. Call them at the *top level* of your component.
2. Do not call them inside loops, conditions, or nested functions.

```javascript
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // This runs after the component mounts
    fetch('/api/data')
      .then(res => res.json())
      .then(result => setData(result));
      
    // Optional cleanup function
    return () => console.log('Component unmounted');
  }, []); // Empty array means run ONLY once on mount

  return <div>{data ? data.title : 'Loading...'}</div>;
}

```

## 11. Sharing Data Between Components (Props)

Data in React is passed strictly **downwards** from parent to child via **props** (short for properties).

```text
  +----------------+ 
  | Parent (State) |
  +----------------+
          |
    prop: data={}
          v
  +----------------+
  | Child  (Reads) |
  +----------------+

```

```javascript
// Child receives data via props
function ProfileAvatar({ imageUrl, size }) {
  return <img src={imageUrl} width={size} height={size} alt="Avatar" />;
}

// Parent passes data down
export default function App() {
  return <ProfileAvatar imageUrl="/alice.jpg" size={100} />;
}

```

## 12. React Developer Tools

React Developer Tools is a browser extension (for Chrome, Firefox, etc.) that allows you to inspect the React component tree.

* **Components Tab:** Lets you view the component hierarchy, inspect props, and view/edit local state.
* **Profiler Tab:** Lets you record performance information to identify bottlenecks and unnecessary re-renders.

## 13. Creating Interactive Components

Interactive components combine state (memory) and events (user actions) to create dynamic UIs.

```javascript
import { useState } from 'react';

function Accordion() {
  // State tracks whether the accordion is open
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="accordion">
      {/* Event flips the state boolean */}
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Collapse' : 'Expand'}
      </button>
      
      {/* Conditional rendering based on state */}
      {isOpen && <p>Here is the hidden content!</p>}
    </div>
  );
}

```

---

## 14. Thinking in React

Building a React app requires a shift in how you think about UI design. Imagine we have a mockup of a searchable product table.

### The Mockup

```text
+---------------------------------------+
|  [ Search... ]                    (1) |
|  [x] Only show products in stock  (2) |
|                                       |
|  Name               Price             |
|  Fruits                           (3) |
|    Apple            $1            (4) |
|    Passionfruit     $2                |
|  Vegetables                       (3) |
|    Spinach          $2            (4) |
|    Pumpkin          $4                |
+---------------------------------------+

```

### Step 1: Break the UI into a component hierarchy

Draw boxes around every component and subcomponent in the mockup and name them.

```text
  <FilterableProductTable>          (Wraps everything)
    ├── <SearchBar>                 (1 & 2: User input)
    └── <ProductTable>              (Lists products based on search)
          ├── <ProductCategoryRow>  (3: Category headers)
          └── <ProductRow>          (4: Individual products)

```

### Step 2: Build a static version in React

Build a version that renders the UI from your data model, but adds **no interactivity**.

* Build components that reuse other components and pass data using *props*.
* Do **not** use *state* at all in this step.

```javascript
function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label><input type="checkbox" /> Only show products in stock</label>
    </form>
  );
}
// ... ProductTable, ProductCategoryRow, ProductRow would also be static components

```

### Step 3: Find the minimal but complete representation of UI state

Think of all the pieces of data in your application. Ask three questions about each piece of data to figure out if it is state:

1. Is it passed in from a parent via props? If so, it isn’t state.
2. Does it remain unchanged over time? If so, it isn’t state.
3. Can you compute it based on any other state or props? If so, it isn’t state!

*Our Minimal State:*

1. The search text.
2. The value of the checkbox.
*(The filtered product list is NOT state, because it can be computed from the original data + search text + checkbox).*

### Step 4: Identify where your state should live

Identify every component that renders something based on that state. Find their **closest common parent component**. That is where the state should live.

```text
  <FilterableProductTable>    <-- STATE LIVES HERE (Search text & Checkbox)
    ├── <SearchBar>           <-- Needs to display state
    └── <ProductTable>        <-- Needs state to filter products

```

### Step 5: Add inverse data flow

Since state is passed down, the components at the bottom (like `<SearchBar>`) need a way to update the state at the top (`<FilterableProductTable>`). The parent passes down a callback function as a prop to allow the child to trigger an update.

```text
  [Parent]  Passes state AND setter functions down
     |      props: { filterText, onFilterTextChange }
     v
  [Child]   User types -> calls onFilterTextChange() -> Updates Parent

```

```javascript
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        // Passing down the ability to change state
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}
```
