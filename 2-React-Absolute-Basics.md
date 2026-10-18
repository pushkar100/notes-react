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

React is a JavaScript library for building user interfaces out of individual pieces called **components**. It abstracts away manual DOM manipulation, allowing you to declare what the UI _should_ look like based on current data.

```text
  +-----------------+       +----------------+       +-------------+
  | Application Data| ----> | React Elements | ----> | Browser DOM |
  |     (State)     |       |   (Virtual)    |       |  (Actual)   |
  +-----------------+       +----------------+       +-------------+

```

```javascript
import { createRoot } from "react-dom/client";

// The root is the DOM element where React takes over
const root = createRoot(document.getElementById("root"));

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

In React, you can style elements using standard CSS classes (`className`), CSS Modules, utility frameworks (like Tailwind), or dynamic inline style objects.

### Basic Syntax: Classes vs Inline Styles

```text
 CSS File:  .card { padding: 16px; }  ---> JSX: className="card"
 JS Object: { padding: '16px' }       ---> JSX: style={{ padding: '16px' }}
```

- **`className`**: Standard way to apply CSS classes (since `class` is a reserved keyword in JavaScript).
- **`style={{ ... }}`**: Passes an object of camelCase CSS properties directly to the element. **Use this only for dynamic values computed at runtime** (e.g., progress bar widths or live coordinates).

```javascript
// Static styles come from className; dynamic width from style
function ProgressBar({ percent }) {
  return (
    <div className="progress-track">
      <div className="progress-fill" style={{ width: `${percent}%` }} />
    </div>
  );
}
```

---

### CSS Modules: Scoped Stylesheets

#### What are CSS Modules?

CSS Modules are regular CSS files named `[name].module.css` (e.g., `Button.module.css`). During build time, your bundler automatically converts class names into unique identifiers (e.g., `Button_btn__x8k2q`), completely preventing naming collisions across your project.

#### Regular CSS vs CSS Modules

- **Regular CSS (`import './styles.css'`):** All classes are **global**. If two files define `.btn`, one will accidentally overwrite the other.
- **CSS Modules (`import styles from './Button.module.css'`):** Classes are scoped locally and accessed as properties on an imported object.

```javascript
/* Button.module.css */
/* .btn { background: blue; color: white; } */

import styles from "./Button.module.css";

function Button() {
  // Rendered in DOM as: class="Button_btn__x8k2q"
  return <button className={styles.btn}>Click Me</button>;
}
```

#### Benefits & Drawbacks of CSS Modules

- **Benefits:**
  - **No naming conflicts:** Zero risk of style bleed across components.
  - **Standard CSS:** Full support for pseudo-classes (`:hover`), media queries (`@media`), and animations.
  - **Zero runtime cost:** Compiles down to plain static `.css` files cached by the browser.
- **Drawbacks:**
  - Separate files: Requires context switching between the `.jsx` file and `.module.css`.
  - Dynamic styles: Styling based on props requires toggling classes or combining with inline styles.

#### When to Use / When Not to Use CSS Modules

- **When to Use:**
  - **Over Regular CSS:** Almost always in production apps—eliminates accidental global style overrides while letting you write vanilla CSS.
  - **Over Styled JS (CSS-in-JS):** When targeting modern Server Components (RSC) or prioritizing zero-runtime performance and tiny bundle sizes.
  - **Over Tailwind:** When your team prefers traditional CSS syntax or wants clean, semantic JSX markup without long utility class strings.
- **When NOT to Use:**
  - When your team prioritizes fast prototyping without jumping between files (use **Tailwind** instead).
  - For values continuously calculated at runtime via JavaScript (e.g., dynamic coordinates—use **inline styles**).
  - For throwaway scripts or simple sandboxes where a single global `.css` file is faster to spin up.

---

### Tailwind CSS: Utility-First Styling

#### What is Tailwind CSS?

Tailwind is a utility-first CSS framework where styling is composed using pre-defined helper classes directly in the `className` attribute.

```javascript
function Button({ isPrimary }) {
  // Styles are colocated directly within JSX
  return (
    <button
      className={`px-4 py-2 rounded-md font-medium ${
        isPrimary ? "bg-blue-600 text-white" : "bg-gray-200 text-black"
      }`}
    >
      Click Me
    </button>
  );
}
```

#### Benefits & Drawbacks of Tailwind

- **Benefits:**
  - **Rapid UI building:** No switching between CSS files and JSX; styles are immediately visible where used.
  - **Enforced design tokens:** Guarantees consistent spacing, typography scales, and color palettes.
  - **Zero runtime overhead:** Purges unused styles at build time, resulting in tiny, highly cacheable CSS bundles with full RSC support.
- **Drawbacks:**
  - Cluttered JSX: Markup can become hard to read due to long class lists.
  - Learning curve: Requires memorizing utility class shorthand names.

#### When to Use / When Not to Use Tailwind

- **When to Use:**
  - **Over Regular CSS / CSS Modules:** When building modern web apps fast, maintaining a unified design system, or eliminating file-hopping.
  - **Over Styled JS (CSS-in-JS):** For modern React 18/19 and Next.js projects requiring zero-runtime overhead and full Server Component compatibility.
- **When NOT to Use:**
  - When you or your team prefer clean separation of concerns and classic CSS stylesheets (use **CSS Modules** instead).
  - Content-heavy sites (e.g., long-form Markdown articles) where standard cascading HTML tags are easier to style globally.

---

### CSS-in-JS: The Rise, Fall, and Evolution

#### What is CSS-in-JS & Why Was It Created?

Libraries like **Styled Components** and **Emotion** let developers write CSS directly inside JavaScript files:

```javascript
// Example using styled-components
const Button = styled.button`
  background: ${(props) => (props.primary ? "blue" : "gray")};
  color: white;
`;
```

**Why it was created:**
Early React lacked clean ways to scope styles and share dynamic JavaScript props with CSS. CSS-in-JS solved this by colocating styles with components and making styles react directly to props.

#### The Costs & Why Runtime CSS-in-JS is Considered Legacy

1. **Runtime Performance Overhead:** The browser has to parse JavaScript, evaluate styles, hash them, and inject `<style>` tags into the DOM on every render.
2. **Incompatible with React Server Components (RSC):** Modern React (React 18/19 and Next.js App Router) renders components on the server where runtime `<style>` injection does not work.
3. **Larger Bundle Size:** The runtime styling library adds extra JavaScript weight to client downloads.

#### Do They Still Have a Use Case?

- **Yes:** Existing legacy codebases, client-only Single Page Apps (Vite/CRA), or mature internal component libraries built around Styled Components.

#### What Replaces Them?

1. **Zero-Runtime CSS-in-JS:** Libraries like **StyleX** (by Meta), **Vanilla Extract**, and **Panda CSS**. You write styles in JS/TS with type safety, but the compiler extracts them into plain static `.css` at build time with **zero runtime overhead** and full RSC support.
2. **Utility-First CSS (Tailwind CSS):** The most popular modern choice for speed and colocation with zero runtime cost.
3. **CSS Modules:** The reliable, native, zero-runtime standard for traditional stylesheet authoring.

---

### Quick Summary: Which to Choose?

| Method                                      | Best For                                                      | Runtime Cost             | RSC Compatible? |
| :------------------------------------------ | :------------------------------------------------------------ | :----------------------- | :-------------- |
| **Tailwind CSS**                            | Fast UI development, design systems, colocation in JSX        | None (build-time)        | Yes             |
| **CSS Modules**                             | Traditional CSS syntax with safe local scoping                | None (build-time)        | Yes             |
| **Zero-Runtime (StyleX / Vanilla Extract)** | Type-safe styling in JS without performance penalty           | None (build-time)        | Yes             |
| **Inline Styles (`style`)**                 | Dynamic values computed at runtime (percentages, coordinates) | Minimal                  | Yes             |
| **Runtime CSS-in-JS (Styled Components)**   | Legacy apps / Client-only SPAs (avoid for new projects)       | High (runtime injection) | No              |

## 5. Displaying Data

JSX lets you embed JavaScript expressions inside curly braces `{}`.

```javascript
function WelcomeMessage() {
  const user = { name: "Alice", role: "Admin" };

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
function AdminPanel({ isAdmin, unreadCount, isLoading }) {
  // 1. Early return: Best when entire view changes (e.g. loading or error screen)
  if (isLoading) {
    return <Spinner />;
  }

  return (
    <div>
      {/* 2. Ternary (? :): Best for choosing between two elements */}
      {isAdmin ? <Dashboard /> : <AccessDenied />}

      {/* 3. Logical &&: Best for render-or-nothing */}
      {isAdmin && <SettingsButton />}

      {/* Watch out: use count > 0 so '0' is not accidentally printed */}
      {unreadCount > 0 && <Badge count={unreadCount} />}
    </div>
  );
}
```

### Best Ways to Render Conditionally: When to Use Which?

| Pattern                | Best Used When                                                    | Example                                       |
| :--------------------- | :---------------------------------------------------------------- | :-------------------------------------------- |
| **Early `return`**     | Guard clauses (blocking renders, loading spinners, error states). | `if (!user) return <Login />;`                |
| **Ternary (`? :`)**    | Choosing between two alternate elements (A vs B).                 | `{isLoggedIn ? <LogoutBtn /> : <LoginBtn />}` |
| **Logical AND (`&&`)** | Rendering an element or nothing at all.                           | `{hasError && <ErrorMessage />}`              |

> [!CAUTION]
> **The `0` Trap with `&&`:** Never place a raw number on the left side of `&&` (e.g., `{items.length && <List />}`). In JavaScript, `0 && ...` evaluates directly to `0`, causing React to render the literal digit `0` on your screen. Always coerce to boolean: `{items.length > 0 && <List />}` or `{Boolean(items.length) && <List />}`.

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
  { title: "Cabbage", id: 1 },
  { title: "Garlic", id: 2 },
  { title: "Apple", id: 3 },
];

function ShoppingList() {
  const listItems = products.map((product) => (
    // 'key' must be unique among siblings
    <li key={product.id}>{product.title}</li>
  ));

  return <ul>{listItems}</ul>;
}
```

### Things to Keep in Mind When Rendering Lists

1. **Keys must be unique and stable:**
   - Always use an ID that stays permanently with the item (e.g., database ID `item.id`).
   - **Never generate keys on the fly:** Writing `key={Math.random()}` forces React to recreate the entire DOM tree on every render, destroying internal component state (like text input focus).

2. **Avoid using array index (`key={index}`):**
   - If list items can be added, deleted, reordered, or filtered, using the array index as the key causes subtle rendering bugs (e.g., input values or checkboxes remaining attached to the wrong item). Only use `index` if the list is strictly static and never changes order.

3. **Keys belong on the outermost element in `map()`:**
   - The key must be placed on the top-level element returned inside the `map()` callback.
   - If returning multiple sibling elements per item, use `<React.Fragment key={item.id}>`:
     ```javascript
     items.map((item) => (
       <React.Fragment key={item.id}>
         <dt>{item.term}</dt>
         <dd>{item.description}</dd>
       </React.Fragment>
     ));
     ```

## 8. Responding to Events

You can respond to user interactions by passing a function as a prop to an element.

```javascript
function AlertButton() {
  // Define the event handler
  function handleClick() {
    alert("You clicked me!");
  }

  return (
    // Pass the function (don't call it with ()!)
    <button onClick={handleClick}>Click me</button>
  );
}
```

## 9. Updating the Screen (State)

To make a component "remember" things (like user input or toggle status) and re-render when they change, add **state**.

```text
  User Clicks -> setState(newData) -> Component Re-evaluates -> UI Updates

```

```javascript
import { useState } from "react";

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

Functions starting with `use` are called **Hooks**. Built-in examples include `useState` (for memory) and `useEffect` (for synchronizing with external systems).

### The Golden Rules of Hooks

1. **Only call Hooks at the top level of your component.**
2. **Do not call Hooks inside loops, conditions, or nested functions.**

#### What do the rules actually mean?

It means calling Hooks directly in the main body of your component function—before any `return` statements **and** outside any nested blocks (`if`, `for`, or helper functions).

```javascript
// ✅ CORRECT: Hooks are at the top level before early returns
function Profile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // fetch user logic
  }, [userId]);

  if (!userId) return <p>No user ID provided</p>;
  return <div>{user?.name}</div>;
}

// ❌ WRONG: Hook placed inside a condition / after early return
function Profile({ userId }) {
  if (!userId) return <p>No user ID provided</p>;

  // BUG! Hook is skipped when userId is falsy
  const [user, setUser] = useState(null);
}
```

#### Why does this rule exist? (Call Order Matters!)

React does **not** identify hooks by their variable names. Instead, React tracks hooks using an internal ordered array (slot 0, slot 1, slot 2).

```text
Render 1 (Normal run):
  Hook 1 [useState: name]  ---> Slot 0: "Alice"
  Hook 2 [useState: age]   ---> Slot 1: 25
  Hook 3 [useEffect]       ---> Slot 2: Timer subscription

Render 2 (If Hook 1 was inside an "if" and skipped):
  Hook 2 runs first!       ---> Reads Slot 0! ("Alice" assigned to age -> BUG / CRASH)
```

By ensuring Hooks are always called unconditionally at the top level, React is guaranteed to execute every Hook in the exact same sequence on every single render.

---

### Understanding `useEffect`: When to Use & Common Pitfalls

`useEffect` allows you to synchronize your component with an **external system** (APIs, window events, timers, or browser DOM widgets) after rendering.

```javascript
import { useState, useEffect } from "react";

function ChatRoom({ roomId }) {
  useEffect(() => {
    // 1. Setup: Connect to external service
    const connection = createConnection(roomId);
    connection.connect();

    // 2. Cleanup: Disconnect when roomId changes or component unmounts
    return () => connection.disconnect();
  }, [roomId]); // Dependency array: Re-runs only when roomId changes

  return <h1>Welcome to room {roomId}</h1>;
}
```

#### When to Use `useEffect`

- Connecting to external services (WebSockets, analytics trackers).
- Subscribing to global browser events (e.g. `window.addEventListener('resize', ...)`).
- Manual timers (`setInterval`, `setTimeout`).

#### Major Pitfalls & When NOT to Use `useEffect`

| Bad Pattern (Pitfall)               | Why It's Dangerous                                                                                                | The Better Fix                                                                                  |
| :---------------------------------- | :---------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------- |
| **Transforming data for rendering** | Setting state inside an effect triggers an unnecessary second render and layout flicker.                          | Calculate values directly in the component body: `const fullName = `${first} ${last}`;`.        |
| **Handling user interactions**      | Syncing button clicks via an effect (`onClick -> setState(true) -> useEffect -> fetch`) leads to race conditions. | Run action logic directly inside the event handler (`onClick={handleSubmit}`).                  |
| **Missing cleanup functions**       | Adding event listeners or intervals without returning a cleanup function causes memory leaks.                     | Always return a cleanup callback: `return () => window.removeEventListener(...)`.               |
| **Infinite render loops**           | Calling `setState` inside `useEffect` without a dependency array or using unmemoized objects in dependencies.     | Be precise with dependencies: `[]` runs once on mount; `[dep]` re-runs only when `dep` changes. |

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

- **Components Tab:** Lets you view the component hierarchy, inspect props, and view/edit local state.
- **Profiler Tab:** Lets you record performance information to identify bottlenecks and unnecessary re-renders.

## 13. Creating Interactive Components

Interactive components combine state (memory) and events (user actions) to create dynamic UIs.

```javascript
import { useState } from "react";

function Accordion() {
  // State tracks whether the accordion is open
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="accordion">
      {/* Event flips the state boolean */}
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? "Collapse" : "Expand"}
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

- Build components that reuse other components and pass data using _props_.
- Do **not** use _state_ at all in this step.

```javascript
function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" /> Only show products in stock
      </label>
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

_Our Minimal State:_

1. The search text.
2. The value of the checkbox.
   _(The filtered product list is NOT state, because it can be computed from the original data + search text + checkbox)._

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
  const [filterText, setFilterText] = useState("");
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        // Passing down the ability to change state
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly}
      />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```
