# Flux architecture

- [Flux architecture](#flux-architecture)
  - [1. What is Flux and what problem does it solve?](#1-what-is-flux-and-what-problem-does-it-solve)
    - [The MVC Problem (Bidirectional Chaos)](#the-mvc-problem-bidirectional-chaos)
    - [The Flux Solution (Unidirectional Flow)](#the-flux-solution-unidirectional-flow)
  - [2. Why React is based on it](#2-why-react-is-based-on-it)
    - [Conceptual Implementation in React](#conceptual-implementation-in-react)
  - [3. Why Redux is based on it](#3-why-redux-is-based-on-it)
    - [Redux Architecture](#redux-architecture)
    - [Example A: The Problem Without Redux (Prop Drilling)](#example-a-the-problem-without-redux-prop-drilling)
      - [Why this is an architectural nightmare for scale:](#why-this-is-an-architectural-nightmare-for-scale)
    - [Example B: The Solution With Redux (Centralized Truth)](#example-b-the-solution-with-redux-centralized-truth)
  - [4. New Age Architectures (Simpler Alternatives)](#4-new-age-architectures-simpler-alternatives)
    - [Alternative 1: Atomic State (Jotai, Recoil)](#alternative-1-atomic-state-jotai-recoil)
    - [Alternative 2: Server-State Caching (React Query, SWR)](#alternative-2-server-state-caching-react-query-swr)
    - [Alternative 3: Proxy/Mutable State (Zustand, MobX)](#alternative-3-proxymutable-state-zustand-mobx)

Flux is an architectural pattern for building client-side web applications, introduced by Facebook. It is not a library, but a mental model built around a **unidirectional data flow**.

## 1. What is Flux and what problem does it solve?

In traditional MVC (Model-View-Controller) architectures on the frontend, models and views often have bidirectional relationships. A model updates a view, the user interacts with the view, which updates a model, which might trigger an update in _another_ model, updating _another_ view.

As applications scale, this creates cascading, unpredictable state changes. Facebook famously encountered this with their "unread messages phantom bug," where the message count would show "1" but reading the message wouldn't clear the counter because the models were out of sync.

Flux solves this by enforcing a strict, one-way pipeline. Data only ever flows in a single direction.

### The MVC Problem (Bidirectional Chaos)

```text
       +-------+               +-------+
       | Model | <-----------> | View  |
       +-------+               +-------+
         ^   |                   ^   |
         |   |                   |   |
         |   v                   |   v
       +-------+               +-------+
       | Model | <-----------> | View  |
       +-------+               +-------+

```

**Model-View-Controller (MVC) architecture**:

1. _The Model (The Chef)_: Manages the core data, logic, and raw ingredients in the background, completely independent of how the final interface looks.
2. _The View (The Table/Presentation)_: This is the beautifully set table, the menu, and the plate presentation. It is what the customer sees and interacts with.
3. _The Controller (The Waiter)_: Acts as the middleman. It takes user input, passes it to the Model (the Chef) for processing, fetches the result, and updates the View (the table).

**The Pros of MVC**

- _Separation of Concerns_: UI code stays away from business logic. You can completely redesign the View without touching how the Model saves data.
- _Parallel Development_: One developer can work on the database/logic (Model) while another builds the interface (View).
- _Reusability_: You could plug a MobileView or a WebView into the exact same Model.

```javascript
// 1. THE MODEL (Data & Logic)
const model = {
  todos: ["Buy milk", "Clean room"],
  add(text) {
    this.todos.push(text);
  },
};

// 2. THE VIEW (User Interface & Interaction)
const view = {
  render(todos) {
    console.log(
      `--- VIEW ---\n${todos.map((t) => `[ ] ${t}`).join("\n")}\n------------`,
    );
  },
  // Simulates user hitting enter/submit
  submitNewTodo(text) {
    console.log(`> User adds: "${text}"`);
    controller.handleAdd(text);
  },
};

// 3. THE CONTROLLER (The Brain / Middleman)
const controller = {
  init() {
    view.render(model.todos);
  },
  handleAdd(text) {
    if (!text.trim()) return;
    model.add(text); // 1. Update Model
    view.render(model.todos); // 2. Update View
  },
};

// --- EXECUTION ---
controller.init();
view.submitNewTodo("Learn Flux");
```

Example run:

```bash
--- VIEW ---
[ ] Buy milk
[ ] Clean room
------------
> User adds: "Learn Flux"
--- VIEW ---
[ ] Buy milk
[ ] Clean room
[ ] Learn Flux
------------
```

**⚠️ The Problem: The "Cascading Effect"**

- MVC works beautifully for _small_ or _traditional server-rendered apps_.
- **Problem**: As _single-page web applications_ (like Facebook) grew massive, traditional MVC broke down due to bidirectional (two-way) data flow

In a large complex system, Views can update Models, and Models can update multiple Views. Suddenly, you get a tangled web of dependencies:

- `View A` triggers an action that updates `Model X`.
- `Model X` updates, which triggers a change in `View B`.
- `View B` needs to update `Model Y` because of that change.
- `Model Y` updates, which loops back and triggers `View A` again.

```javascript
// ==========================================
// 1. THE MODELS (Data stores)
// ==========================================
const modelX = { data: 0 };
const modelY = { status: "Idle" };

// ==========================================
// 2. THE CONTROLLERS (Business logic routers)
// ==========================================
const controllerX = {
  updateX(val) {
    console.log(`[Controller X] Updating Model X to: ${val}`);
    modelX.data = val;
    viewB.render(); // Model X changes always update View B
  },
};

const controllerY = {
  updateY(status) {
    console.log(`[Controller Y] Updating Model Y to: ${status}`);
    modelY.status = status;
    viewA.render(); // Model Y changes always update View A
  },
};

// ==========================================
// 3. THE VIEWS (Tangled User Interfaces)
// ==========================================
const viewA = {
  render() {
    console.log(`[View A] Renders with Model Y Status: "${modelY.status}"`);
    console.log(`  ⚡ Cascade! View A triggers Controller X...`);
    controllerX.updateX(modelX.data + 1);
  },
};

const viewB = {
  render() {
    console.log(`[View B] Renders with Model X Data: ${modelX.data}`);
    console.log(`  ⚡ Cascade! View B triggers Controller Y...`);
    controllerY.updateY(`Running loop ${modelX.data}`);
  },
};

// ==========================================
// EXECUTION (The Initial Trigger)
// ==========================================
console.log("=== STARTING APPLICATION INTERACTION ===");
// User does an initial action that targets Controller X
controllerX.updateX(1);
```

This creates a **cascading effect**. A single user click can trigger a domino effect of data bouncing wildly across your app. It becomes impossible to debug because you can't easily track which component caused what change.

```bash
=== STARTING APPLICATION INTERACTION ===
[Controller X] Updating Model X to: 1
[View B] Renders with Model X Data: 1
  ⚡ Cascade! View B triggers Controller Y...
[Controller Y] Updating Model Y to: Running loop 1
[View A] Renders with Model Y Status: "Running loop 1"
  ⚡ Cascade! View A triggers Controller X...
[Controller X] Updating Model X to: 2
[View B] Renders with Model X Data: 2
  ⚡ Cascade! View B triggers Controller Y...
[Controller Y] Updating Model Y to: Running loop 2
[View A] Renders with Model Y Status: "Running loop 2"
  ⚡ Cascade! View A triggers Controller X...

... (repeats over and over until JavaScript crashes) ...

RangeError: Maximum call stack size exceeded
    at console.log (<anonymous>)
    at Object.updateX (mvc.js:12)
```

```
[ CALL STACK MEMORY ]
┌────────────────────────────────────────────────────────┐
│ 1. ControllerX() starts...                             │
│    │                                                   │
│    ├──► 2. ViewB() starts...                           │
│    │       │                                           │
│    │       ├──► 3. ControllerY() starts...             │
│    │       │       │                                   │
│    │       │       ├──► 4. ViewA() starts...           │
│    │       │       │       │                           │
│    │       │       │       └──► 5. ControllerX()... 💥 │ <-- STACK OVERFLOW!
└──────┴───────┴───────┴──────────────┴──────────────────┘
  None of these functions can finish. They are locked inside each other.
```

Facebook famously experienced this with their notification counter. Users would see a notification icon, click it, clear it, but the counter would randomly light up again because a rogue Model-View cycle was mismatching data somewhere in a cascade.

To kill the chaos of cascading updates, Facebook engineers threw out bidirectional MVC for client-side state and introduced **Flux, which forces a strictly unidirectional (one-way) data flow**.

```javascript
// ==========================================
// 1. THE DISPATCHER (With Action Queue)
// ==========================================
const dispatcher = {
  isDispatching: false,
  actionQueue: [],

  dispatch(action) {
    // 1. If an action comes in during a render, queue it up safely
    this.actionQueue.push(action);
    if (this.isDispatching) return;

    this.isDispatching = true;

    // 2. Process actions one by one until the queue is completely empty
    while (this.actionQueue.length > 0) {
      const currentAction = this.actionQueue.shift();
      console.log(`\n📢 [Dispatcher] Processing Action: ${currentAction.type}`);

      // Pass action to all stores
      storeX.update(currentAction);
      storeY.update(currentAction);

      // Render the views after each independent action completes
      viewA.render();
      viewB.render();
    }

    this.isDispatching = false;
  },
};

// ==========================================
// 2. THE STORES (Data Stores)
// ==========================================
const storeX = {
  data: 0,
  update(action) {
    if (action.type === "INCREMENT_X") {
      this.data = action.value;
      console.log(`📦 [Store X] Updated data to: ${this.data}`);
    }
  },
};

const storeY = {
  status: "Idle",
  update(action) {
    if (action.type === "UPDATE_Y") {
      this.status = action.status;
      console.log(`📦 [Store Y] Updated status to: "${this.status}"`);
    }
  },
};

// ==========================================
// 3. THE VIEWS (Can successfully trigger actions)
// ==========================================
const viewA = {
  render() {
    console.log(`[View A] Renders with Store Y Status: "${storeY.status}"`);

    // To match MVC, if data is less than 3, View A requests an update to X
    if (storeX.data < 3) {
      console.log(`  ➡️ [View A] Dispatches INCREMENT_X`);
      dispatcher.dispatch({ type: "INCREMENT_X", value: storeX.data + 1 });
    }
  },
};

const viewB = {
  render() {
    console.log(`[View B] Renders with Store X Data: ${storeX.data}`);

    // To match MVC, View B handles Store X updates by requesting an update to Y
    // (We look at the string to prevent a true business-logic infinite loop)
    if (storeY.status !== `Running loop ${storeX.data}`) {
      console.log(`  ➡️ [View B] Dispatches UPDATE_Y`);
      dispatcher.dispatch({
        type: "UPDATE_Y",
        status: `Running loop ${storeX.data}`,
      });
    }
  },
};

// ==========================================
// EXECUTION (The Initial Action)
// ==========================================
console.log("=== STARTING APPLICATION WITH QUEUED FLUX ===");
dispatcher.dispatch({ type: "INCREMENT_X", value: 1 });
```

```bash
=== STARTING APPLICATION WITH QUEUED FLUX ===

📢 [Dispatcher] Processing Action: INCREMENT_X
📦 [Store X] Updated data to: 1
[View A] Renders with Store Y Status: "Idle"
  ➡️ [View A] Dispatches INCREMENT_X
[View B] Renders with Store X Data: 1
  ➡️ [View B] Dispatches UPDATE_Y

📢 [Dispatcher] Processing Action: INCREMENT_X
📦 [Store X] Updated data to: 2
[View A] Renders with Store Y Status: "Idle"
  ➡️ [View A] Dispatches INCREMENT_X
[View B] Renders with Store X Data: 2
  ➡️ [View B] Dispatches UPDATE_Y

📢 [Dispatcher] Processing Action: UPDATE_Y
📦 [Store Y] Updated status to: "Running loop 1"
[View A] Renders with Store Y Status: "Running loop 1"
[View B] Renders with Store X Data: 2
...
```

Explaining how it all works:

```
                  [ THE CONVEYOR BELT (QUEUE) ]
       ┌───────────────────┬───────────────────┬───────────────────┐
 ◄───  │ 1st: Action #1    │ 2nd: Action #2    │ 3rd: Action #3    │  ◄─── [Views drop new
       │ (From User)       │ (From View A)     │ (From View B)     │       actions here]
       └─────────┬─────────┴───────────────────┴───────────────────┘
                 │
                 ▼
      [ COMPUTER EXECUTION BLOCK ]
       ┌────────────────────────┐
       │ Currently processing:  │
       │ Action #1              │
       └────────────────────────┘
```

```
[ CALL STACK TIMELINE ]

 Time 1: ┌───────────────────┐ (Action #1 runs)
         │ 1. Process #1     │ ──► Finishes and clears out of memory completely!
         └───────────────────┘

 Time 2: ┌───────────────────┐ (Action #2 runs)
         │ 1. Process #2     │ ──► Finishes and clears out of memory completely!
         └───────────────────┘

 Time 3: ┌───────────────────┐ (Action #3 runs)
         │ 1. Process #3     │ ──► Finishes and clears out of memory completely!
         └───────────────────┘
```

`Action 2` gets picked up automatically by a loop (while) running inside the Dispatcher, immediately after Action 1 finishes completely.

_The key trick is that the dispatcher remains "locked" in a busy state, running a loop that refuses to stop until the queue is completely empty._

```
User triggers Action 1 ──► [Queue: Action 1]
                                 │
                        ┌────────▼────────┐
                        │ dispatcher.dispatch()
                        │ sets isDispatching = true;
                        └────────┬────────┘
                                 │
       ┌─────────────────────────▼─────────────────────────┐
       │  LOOP START: while (actionQueue.length > 0)       │ ◄────┐
       └─────────────────────────┬─────────────────────────┘      │
                                 │ .shift()                       │
                                 ▼                                │
                       [ Processing Action 1 ]                    │
                        ├─ Store updates                          │
                        └─ View renders                           │
                                 │                                │
                       View triggers Action 2                     │ Loop cycles back
                       [Queue: Action 2]                          │ automatically
                       dispatcher.dispatch() called again         │
                       but hits `if (isDispatching) return;`      │
                                 │                                │
                                 ▼                                │
                       [ Action 1 Finishes ]                      │
                                 │                                │
                                 └────────────────────────────────┘
```

### The Flux Solution (Unidirectional Flow)

```text
                  +------------+
                  |            |
  +--------+      |            v      +-------+      +------+
  | Action | ---> | Dispatcher | ---> | Store | ---> | View |
  +--------+      |            ^      +-------+      +------+
      ^           +------------+                         |
      |                                                  |
      +--------------------------------------------------+
                      User Interaction

```

- **Action:** A simple object describing what happened (e.g., `{ type: 'MESSAGE_READ', id: 123 }`).
- **Dispatcher:** The central hub. It receives actions and broadcasts them to all registered Stores.
- **Store:** Holds the application state and logic. It updates itself based on actions and tells the Views to re-render.
- **View:** Renders the UI based on the Store's data.

## 2. Why React is based on it

React natively embraces unidirectional data flow. In React, data flows down the component tree via props, and events flow up via callbacks. React itself is technically just the "View" layer in the Flux pattern, but its internal hooks—specifically `useReducer`—are direct conceptual implementations of Flux.

React uses this pattern because it makes component rendering predictable. If the state is X, the UI will always look like Y.

### Conceptual Implementation in React

```text
  +---------+ (State down)  +-------+ (Events up)  +---------+
  | Parent  | ------------> | Child | -----------> | Parent  |
  | (Store) |               | (View)|              |(Action) |
  +---------+               +-------+              +---------+

```

Here is how React implements Flux conceptually using native hooks:

```javascript
import { useReducer } from "react";

// 1. Store Logic (Reducer)
// It takes the current state and an Action, returning the new state.
function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      return { ...state, items: [...state.items, action.payload] };
    default:
      return state;
  }
}

export default function App() {
  // 2. Dispatcher & Store initialization
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  // 3. Action Creator
  const handleAddItem = (item) => {
    dispatch({ type: "ADD_ITEM", payload: item });
  };

  // 4. View
  return (
    <div>
      <p>Items in cart: {state.items.length}</p>
      {/* User interaction triggers the action */}
      <button onClick={() => handleAddItem("Laptop")}>Add Laptop</button>
    </div>
  );
}
```

## 3. Why Redux is based on it

Redux is a direct evolution of Flux. It takes the core concept of unidirectional data flow but simplifies the architecture by making three specific changes:

1. **Single Source of Truth:** Instead of multiple Stores (like in original Flux), Redux has exactly one global Store.
2. **State is Read-Only:** The only way to change state is to emit an Action.
3. **Changes are made with Pure Functions:** Reducers take the previous state and an action, and return a new state object. (Redux eliminates the traditional Flux Dispatcher entirely in favor of these pure reducer functions).

### Redux Architecture

```text
  +---------+       +---------+       +-------+       +------+
  | Action  | ----> | Reducer | ----> | Store | ----> | View |
  +---------+       +---------+       +-------+       +------+
       ^                                                 |
       |_________________________________________________|

```

### Example A: The Problem Without Redux (Prop Drilling)

When state lives at the top of the tree, you have to pass it through every intermediate component to get it to the bottom. This makes your codebase brittle and causes intermediate components to re-render unnecessarily.

```text
                     +---------------------------+
                     |        App (State)        |
                     +---------------------------+
                       /                       \
             (Passes cartCount)           (Passes setCartCount)
                     /                           \
           +--------------+                +---------------+
           |    Header    |                |      Main     |
           +--------------+                +---------------+
                  |                                |
           +--------------+                +---------------+
           |    Navbar    |                |  ProductGrid  |
           +--------------+                +---------------+
                  |                                |
           +--------------+                +---------------+
           |   CartIcon   |                |  ProductCard  |
           |    (Reads)   |                |   (Updates)   |
           +--------------+                +---------------+

```

```javascript
// PROBLEM: Prop Drilling
// To share data between CartIcon and ProductCard, the state MUST live
// at the root level. Every component in between acts as a dumb pipe.

export default function App() {
  // 1. State is trapped at the top level
  const [cartCount, setCartCount] = useState(0);

  return (
    <div>
      {/* 2. Passing props through layers that don't care about them */}
      <Header cartCount={cartCount} />
      <Main setCartCount={setCartCount} />
    </div>
  );
}

// --- LEFT BRANCH (Reading State) ---

// Header doesn't care about the cart, it's just a conduit.
function Header({ cartCount }) {
  return <Navbar cartCount={cartCount} />;
}

// Navbar also doesn't care.
function Navbar({ cartCount }) {
  return <CartIcon cartCount={cartCount} />;
}

// Finally, 3 levels deep, the prop is actually used.
function CartIcon({ cartCount }) {
  return <div>🛒 Items: {cartCount}</div>;
}

// --- RIGHT BRANCH (Updating State) ---

// Main doesn't care about updating the cart.
function Main({ setCartCount }) {
  return <ProductGrid setCartCount={setCartCount} />;
}

// ProductGrid also doesn't care.
function ProductGrid({ setCartCount }) {
  return <ProductCard setCartCount={setCartCount} />;
}

// Finally, 3 levels deep, we trigger the update.
function ProductCard({ setCartCount }) {
  return (
    <button onClick={() => setCartCount((c) => c + 1)}>Add to Cart</button>
  );
}
```

#### Why this is an architectural nightmare for scale:

1. **Refactoring is painful:** If `CartIcon` moves to a different part of the UI (e.g., a Sidebar instead of a Header), you have to rip out the `cartCount` prop from the entire Header chain and thread it down a brand new chain of components.
2. **Zombie Code:** `Header` and `Main` now have props they don't use. If you eventually remove the `CartIcon`, you have to remember to clean up the unused props in all the parent components.
3. **Performance:** Every time `cartCount` changes, `Header`, `Navbar`, `Main`, and `ProductGrid` might re-render, even though they only exist to pass the data along.

_(Redux solves this by letting `CartIcon` and `ProductCard` tap directly into a global store, completely ignoring the components sitting above them)._

### Example B: The Solution With Redux (Centralized Truth)

Redux moves the state outside the component tree. Components only subscribe to the slices of data they care about.

```javascript
import { useSelector, useDispatch } from "react-redux";

// SOLUTION: Both components connect directly to the single Redux Store.

function Navbar() {
  // Subscribes only to the cart count.
  // Automatically re-renders when the count changes in the global store.
  const cartCount = useSelector((state) => state.cart.length);
  return <nav>Cart: {cartCount}</nav>;
}

function ProductPage() {
  const dispatch = useDispatch();

  const buy = () => {
    // We don't manage local state. We just broadcast an intent (Action).
    // The Reducer handles the logic, updates the Store, and the Navbar reacts.
    dispatch({ type: "ADD_TO_CART", payload: "item" });
  };

  return <button onClick={buy}>Buy</button>;
}
```

## 4. New Age Architectures (Simpler Alternatives)

While Flux and Redux enforce great patterns, they often introduce heavy boilerplate. Modern frontend architectures have shifted to simpler models that often bypass the strict Action -> Reducer pipeline for simpler use cases.

### Alternative 1: Atomic State (Jotai, Recoil)

Instead of one massive global object (Redux), state is broken down into tiny, independent "atoms". Components subscribe only to the specific atoms they need. There is no dispatcher or reducer; you just update the atom directly like `useState`.

```text
  +--------+      +--------+      +--------+
  | Atom A |      | Atom B |      | Atom C |
  +--------+      +--------+      +--------+
      ^               ^               ^
      |               |               |
   Component 1    Component 2     Component 3

```

```javascript
// Jotai Example: Extremely minimal, no reducers or actions.
import { atom, useAtom } from "jotai";

// Define the atom outside the component
const cartCountAtom = atom(0);

function Navbar() {
  const [count] = useAtom(cartCountAtom); // Read
  return <nav>Cart: {count}</nav>;
}

function ProductPage() {
  const [, setCount] = useAtom(cartCountAtom); // Write
  return <button onClick={() => setCount((c) => c + 1)}>Buy</button>;
}
```

### Alternative 2: Server-State Caching (React Query, SWR)

Historically, developers shoved API responses into Redux. We now realize that API data isn't "global state"—it's a server cache. Tools like React Query handle the caching, background fetching, and synchronization, removing the need for 80% of what we used to put in Flux/Redux stores.

```javascript
import { useQuery, useMutation } from "@tanstack/react-query";

function UserProfile({ userId }) {
  // No reducers needed. It fetches, caches, and shares this data
  // across any component that calls this hook with the same key.
  const { data, isLoading } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetchUser(userId),
  });

  if (isLoading) return <span>Loading...</span>;
  return <div>{data.name}</div>;
}
```

### Alternative 3: Proxy/Mutable State (Zustand, MobX)

Zustand is highly popular right now. It uses a single store like Redux, but ditches the strict Action/Reducer boilerplate. You define standard functions that mutate the state directly (behind the scenes, it ensures immutable updates).

```javascript
import { create } from "zustand";

// Store is just an object with data and functions to update that data
const useStore = create((set) => ({
  cart: 0,
  addToCart: () => set((state) => ({ cart: state.cart + 1 })),
}));

function ProductPage() {
  // Grab just the function you need
  const addToCart = useStore((state) => state.addToCart);
  return <button onClick={addToCart}>Buy</button>;
}
```
