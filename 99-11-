# Redux Fundamentals & Redux Toolkit

- [Redux Fundamentals & Redux Toolkit](#redux-fundamentals-redux-toolkit)
  - [Redux fundamentals](#redux-fundamentals)
    - [1. The Origin and The "Why"](#1-the-origin-and-the-why)
    - [2. Core Concepts & Terminology (The Bank Analogy)](#2-core-concepts-terminology-the-bank-analogy)
    - [3. The Redux Application Data Flow](#3-the-redux-application-data-flow)
    - [4. When to Use It (and When Not To)](#4-when-to-use-it-and-when-not-to)
    - [5. Standard Patterns: Redux Toolkit (RTK)](#5-standard-patterns-redux-toolkit-rtk)
      - [The Modern "Slice" Pattern](#the-modern-slice-pattern)
      - [Hooking it up to React](#hooking-it-up-to-react)
    - [6. Tricky Concepts & Gotchas](#6-tricky-concepts-gotchas)
    - [7. Mastery: Asynchronous Logic (Thunks)](#7-mastery-asynchronous-logic-thunks)
    - [8. Common Interview Questions](#8-common-interview-questions)
      - [Conceptual Questions](#conceptual-questions)
      - [Machine Coding Snippet](#machine-coding-snippet)
  - [Redux Toolkit (RTK)](#redux-toolkit-rtk)
    - [1. The Evolution: RTK vs. Legacy Redux](#1-the-evolution-rtk-vs-legacy-redux)
      - [What is a Slice?](#what-is-a-slice)
      - [Why RTK is Better](#why-rtk-is-better)
    - [2. Core Architecture: Store Setup & Usage Steps](#2-core-architecture-store-setup-usage-steps)
      - [Step 1: Create the Store](#step-1-create-the-store)
      - [Step 2: Provide the Store](#step-2-provide-the-store)
      - [Step 3: Use it in Components](#step-3-use-it-in-components)
    - [3. Writing Slices](#3-writing-slices)
    - [4. Thunks: Asynchronous Redux](#4-thunks-asynchronous-redux)
      - [The Old Way](#the-old-way)
      - [The RTK Way (`createAsyncThunk`)](#the-rtk-way-createasyncthunk)
    - [5. Normalizing State (Enterprise Level)](#5-normalizing-state-enterprise-level)
    - [6. Production Ready RTK & Best Practices](#6-production-ready-rtk-best-practices)
    - [7. When to Use It and When Not To](#7-when-to-use-it-and-when-not-to)
    - [8. Tricky Concepts & Gotchas](#8-tricky-concepts-gotchas)
    - [9. Common Interview Questions](#9-common-interview-questions)

## Redux fundamentals

Redux is a predictable state container for JavaScript applications. If React is the engine that renders your user interface, Redux is the centralized database that holds the information your interface needs to function.

Here is a comprehensive deep dive into Redux fundamentals, modernized for how it is actually written today using **Redux Toolkit (RTK)**.

### 1. The Origin and The "Why"

* **Introduced:** 2015 by Dan Abramov and Andrew Clark.
* **Why it exists:** Before Redux, managing state across complex React applications was chaotic. Passing data deep down the component tree (prop drilling) was messy, and having multiple components trying to update the same piece of data caused massive synchronization bugs. Redux introduced a **single source of truth** (one global state tree) and strict rules on how that state could be changed.

### 2. Core Concepts & Terminology (The Bank Analogy)

To master Redux, you must understand its strict vocabulary. The best mental model for Redux is a **Bank**.

| Redux Term | Bank Analogy | Technical Definition |
| --- | --- | --- |
| **Store** | **The Vault** | The single JavaScript object that holds the entire state of your application. |
| **Action** | **The Deposit Slip** | A plain JavaScript object describing *what* happened. It has a `type` (e.g., "DEPOSIT") and usually a `payload` (e.g., "$50"). |
| **Reducer** | **The Teller** | A pure function that takes the current state and the Action, calculates the new state, and returns it. |
| **Dispatch** | **Handing the slip to the teller** | The method used to send an Action to the Reducer. It is the *only* way to trigger a state change. |
| **Selector** | **Checking your balance** | A function that extracts a specific piece of data from the Store to use in your component. |

### 3. The Redux Application Data Flow

Redux enforces a strict **One-Way Data Flow**. State is never mutated directly by the UI.

```text
      THE STRICT ONE-WAY DATA FLOW
      ----------------------------
      
               (1) User clicks "Add to Cart"
                      |
                      V
      [ UI COMPONENT (React) ] 
          |                   ^
          | (2) dispatch()    | (6) Re-renders
          V                   |
      [ ACTION: { type: 'cart/add', payload: { id: 1 } } ]
          |                   |
          | (3) Handed to     | (5) Notifies UI of change
          V                   |
      [ REDUCER (The logic) ] --------> [ STORE (The State) ]
                      (4) Returns NEW state 

```

1. **Event:** A user interacts with the UI.
2. **Dispatch:** The UI dispatches an Action.
3. **Process:** The Store passes the current state and the Action to the Reducer.
4. **Calculate:** The Reducer calculates the new state and saves it in the Store.
5. **Subscribe:** The Store tells the UI that the data has changed.
6. **Update:** The UI reads the new data via a Selector and re-renders.

### 4. When to Use It (and When Not To)

Redux is incredibly powerful, but it is not necessary for every app.

**Good for:**
* Massive applications with complex, frequently updating state (e.g., a collaborative document editor, a complex dashboard, an e-commerce checkout flow).
* Apps where you need deep trace debugging (knowing exactly *when*, *why*, and *how* state changed).
* Apps requiring complex undo/redo functionality.


**Bad for:**
* **Simple Prop Drilling:** If you just need to pass the "current user" down three levels, use the **React Context API**.
* **Server Data Caching:** If you are using Redux *just* to store API responses, you are using the wrong tool. Use **React Query** (or RTK Query) instead.
* **Form State:** Local form inputs should stay in local `useState` or use React Hook Form. Putting every keystroke in Redux is a performance killer.

### 5. Standard Patterns: Redux Toolkit (RTK)

If you look at Redux tutorials from 2018, you will see massive switch statements, action creators, and immutable spread operators. **That pattern is dead.**

Today, the official standard is **Redux Toolkit (RTK)**. RTK abstracts away the boilerplate and uses a library called Immer under the hood, allowing you to write code that *looks* like it is mutating state directly, while remaining perfectly immutable.

#### The Modern "Slice" Pattern

A "Slice" is a collection of Redux reducer logic and actions for a single feature of your app (e.g., `cartSlice`, `userSlice`).

```javascript
// features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

export const counterSlice = createSlice({
  name: 'counter', // The namespace for your actions
  initialState: { value: 0 },
  reducers: {
    // These are your "Tellers". RTK automatically creates Actions for them.
    increment: (state) => {
      // It LOOKS like mutation, but RTK intercepts this and safely
      // creates a brand new state object under the hood!
      state.value += 1; 
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload; // Reading the deposit slip
    },
  },
});

// Export the actions to use in your UI
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// Export the reducer to configure the Store
export default counterSlice.reducer;

```

#### Hooking it up to React

```jsx
// CounterComponent.jsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, incrementByAmount } from './counterSlice';

export const Counter = () => {
  // 1. Selector: Read data from the store
  const count = useSelector((state) => state.counter.value);
  
  // 2. Dispatcher: Get the function to trigger actions
  const dispatch = useDispatch();

  return (
    <div>
      <span>Count: {count}</span>
      <button onClick={() => dispatch(increment())}>+1</button>
      {/* Dispatching an action with a payload */}
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
};

```

### 6. Tricky Concepts & Gotchas

1. **Non-Serializable Data:** You must *never* put Promises, DOM elements, or Functions into the Redux store or Actions. Redux relies on data being serializable (convertible to a JSON string) to power its Time Travel Debugging tools. Only put strings, numbers, booleans, arrays, and plain objects in Redux.
2. **Stale State in Async Logic:** If you read state, make an API call, and then dispatch based on that old state, you might overwrite changes that happened while the API call was flying. Always dispatch based on the freshest data, or let the reducer calculate the math.
3. **Selector Re-renders:** A component re-renders whenever the data returned by `useSelector` changes reference.
* *Bad:* `useSelector(state => state.users.filter(u => u.active))` (Returns a new array reference every render, causing infinite re-renders).
* *Good:* Create a memoized selector using RTK's `createSelector` for complex filtering.

### 7. Mastery: Asynchronous Logic (Thunks)

Reducers must be **pure functions**—they cannot fetch data or cause side effects. To handle asynchronous logic, Redux uses a middleware called a **Thunk**.

A Thunk is a function that wraps an asynchronous operation.

```text
      THE THUNK DATA FLOW (Async)
      ---------------------------
      [ UI ] -> dispatch(fetchUserThunk())
                     |
      [ Middleware ] Intercepts the Thunk!
                     |-- 1. dispatch({ type: 'user/pending' })
                     |-- 2. fetch('/api/user') (Waits...)
                     |-- 3. Success? dispatch({ type: 'user/fulfilled', payload: data })
                     |-- 4. Failed? dispatch({ type: 'user/rejected', error })
                     V
                [ Reducers process the resulting actions ]

```

With modern RTK, you use `createAsyncThunk`:

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

// 1. Define the async Thunk
export const fetchUser = createAsyncThunk(
  'users/fetchById', // Action name prefix
  async (userId) => {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    return await response.json(); // Becomes the 'payload'
  }
);

// 2. Handle the pending/fulfilled/rejected states in the Slice
const userSlice = createSlice({
  name: 'users',
  initialState: { data: null, status: 'idle' },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload; // The returned JSON
      })
      .addCase(fetchUser.rejected, (state) => {
        state.status = 'failed';
      });
  },
});

```

### 8. Common Interview Questions

#### Conceptual Questions

**Q: What is the difference between Redux and the React Context API?**
**Answer:** Context is a *dependency injection* tool; it passes data down the tree without prop-drilling, but it lacks strict state management rules and triggers a re-render of all consumers whenever the context value changes. Redux is an architecture that enforces how state is updated, offers extreme performance optimizations (components only re-render if their specific slice of state changes), and provides powerful middleware/debugging tools.

**Q: Why must a Reducer be a "pure function"?**
**Answer:** A pure function always returns the exact same output for the same input and produces no side effects. Redux relies on this purity for predictability and its "Time Travel Debugging" feature. If a reducer mutated a global variable or made an API call, you could not reliably replay actions to reconstruct the state.

**Q: Under the hood, how does Redux Toolkit allow you to "mutate" state safely?**
**Answer:** RTK uses a library called **Immer**. When you write mutating code (like `state.value += 1`), Immer intercepts the mutation, applies it to a "draft" proxy of the state, and then automatically generates a brand new, immutable state object behind the scenes.

#### Machine Coding Snippet

**Prompt:** *"Write a standard RTK slice for managing a shopping cart. Include an action to add an item, and an action to remove an item by its ID."*

**Solution:**

```javascript
import { createSlice } from '@reduxjs/toolkit';

export const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => {
      // action.payload is the new item object
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },
    removeItem: (state, action) => {
      // action.payload is the item ID
      state.items = state.items.filter(item => item.id !== action.payload);
    }
  }
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;

```

## Redux Toolkit (RTK)

Redux Toolkit (RTK) is the official, opinionated, "batteries-included" toolset for modern Redux development. It was created to solve the three biggest complaints about legacy Redux: the configuration was too complicated, it required too many packages (like `redux-thunk` and `reselect`), and it required too much boilerplate code.

Here is a comprehensive deep dive into RTK, structured for scalable, production-grade applications.

### 1. The Evolution: RTK vs. Legacy Redux

#### What is a Slice?

In legacy Redux, adding a single feature (like a "Cart") required touching three to four different files. RTK introduced the **Slice** architecture. A "Slice" is a single file that bundles the initial state, the reducer logic, and the actions for one specific feature of your app.

```text
      LEGACY REDUX (The Boilerplate Hell)           RTK (The Slice Architecture)
      -----------------------------------           ----------------------------
      /actions         /constants                   /features
        cartActions.js   cartTypes.js                 /cart
             \              /                               cartSlice.js
              \            /                                (Everything in one place!)
               \          /
                /reducers
                 cartReducer.js

```

#### Why RTK is Better

1. **Immer built-in:** Legacy Redux required strict immutability, meaning heavy use of the spread operator (`...state`). RTK uses the `Immer` library under the hood, allowing you to write "mutating" code (`state.value += 1`) that automatically compiles into safe, immutable updates.
2. **Auto-generated Actions:** You no longer write Action Creators. RTK generates them automatically based on the names of your reducer functions.
3. **Built-in Middleware:** `redux-thunk` (for async logic) and Redux DevTools are automatically configured.

### 2. Core Architecture: Store Setup & Usage Steps

Setting up RTK takes exactly three steps.

```text
      RTK STORE SETUP FLOW
      --------------------
   [ Slice A ]   [ Slice B ]
        \           /
    (Exported Reducers)
          \       /
    [ configureStore() ] ---> Creates the Global State Tree
              |
    [ <Provider> Wrapper ] -> Injects the Store into React
              |
    [ React Components ] ---> Uses `useSelector` & `useDispatch`

```

#### Step 1: Create the Store

Use `configureStore`. It automatically combines your slice reducers and sets up the DevTools.

```javascript
// app/store.js
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from '../features/cart/cartSlice';
import userReducer from '../features/user/userSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    user: userReducer,
  },
  // Middleware and DevTools are added automatically!
});

```

#### Step 2: Provide the Store

Wrap your application root.

```jsx
// index.js or main.jsx
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

const Root = () => (
  <Provider store={store}>
    <App />
  </Provider>
);

```

#### Step 3: Use it in Components

Never pass the store directly. Use the hooks.

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { addItem } from './cartSlice';

const ProductPage = ({ product }) => {
  // Read state. This component will ONLY re-render when `cart.items` changes.
  const cartItems = useSelector((state) => state.cart.items);
  
  // Get the dispatch function
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch(addItem(product))}>
      Add to Cart
    </button>
  );
};

```

### 3. Writing Slices

A slice requires a `name`, an `initialState`, and an object of `reducers`.

```javascript
// features/cart/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart', // This prefixes all generated actions (e.g., 'cart/addItem')
  initialState: {
    items: [],
    totalQuantity: 0,
  },
  reducers: {
    // Action: 'cart/addItem'
    addItem: (state, action) => {
      const newItem = action.payload;
      const existingItem = state.items.find(item => item.id === newItem.id);
      
      state.totalQuantity++;
      
      if (!existingItem) {
        // Looks like a mutation, but Immer makes it immutable!
        state.items.push({ ...newItem, quantity: 1 });
      } else {
        existingItem.quantity++;
      }
    },
    // Action: 'cart/removeItem'
    removeItem: (state, action) => {
      const id = action.payload;
      state.items = state.items.filter(item => item.id !== id);
    }
  }
});

// RTK automatically created these action creators for you!
export const { addItem, removeItem } = cartSlice.actions;

export default cartSlice.reducer;

```

### 4. Thunks: Asynchronous Redux

Reducers are purely synchronous. To handle API calls, we use **Thunks**.

#### The Old Way

Historically, you had to write three action types (`FETCH_START`, `FETCH_SUCCESS`, `FETCH_ERROR`), dispatch `FETCH_START`, make the `fetch()` call, and then manually dispatch success or error based on the result. It was 30+ lines of boilerplate.

#### The RTK Way (`createAsyncThunk`)

RTK handles the lifecycle automatically. You give it a string prefix and a payload creator (the async function). It automatically generates the `pending`, `fulfilled`, and `rejected` action types.

```text
      CREATE_ASYNC_THUNK SEQUENCE
      ---------------------------
      [ UI calls dispatch(fetchUser(id)) ]
                    |
      (RTK dispatches 'user/fetch/pending') -> Updates state (isLoading = true)
                    |
             [ Network Request ]
             /                 \
       (Success)             (Failed)
           |                     |
   (RTK dispatches       (RTK dispatches 
  'user/fetch/fulfilled') 'user/fetch/rejected')
           |                     |
     Updates state         Updates state
     with API data         with Error msg

```

```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 1. Define the Async Thunk
export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (userId, thunkAPI) => {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) throw new Error('Failed to fetch');
    return await response.json(); // This becomes the action.payload
  }
);

// 2. Handle the lifecycle in the Slice using extraReducers
const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, status: 'idle', error: null },
  reducers: {}, // Synchronous actions go here
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload; // The returned JSON
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

```

### 5. Normalizing State (Enterprise Level)

When managing complex relational data (like a list of users, where each user has a list of posts), storing arrays of objects in Redux causes severe performance issues. Updating an item in a massive array requires mapping over the entire array.

**Normalizing state** means treating your Redux store like a relational database: storing items in an object lookup dictionary (hash map) by their ID.

RTK provides `createEntityAdapter` to do this automatically.

```text
      NON-NORMALIZED (O(N) Lookups)       NORMALIZED (O(1) Lookups)
      -----------------------------       -------------------------
      users: [                            users: {
        { id: '1', name: 'A' },             ids: ['1', '2'],
        { id: '2', name: 'B' }              entities: {
      ]                                       '1': { id: '1', name: 'A' },
                                              '2': { id: '2', name: 'B' }
                                            }
                                          }

```

```javascript
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

// 1. Create the adapter
const usersAdapter = createEntityAdapter({
  // Assume entities have an 'id' field. If not, specify selectId: (user) => user.userId
  sortComparer: (a, b) => a.name.localeCompare(b.name),
});

// 2. Initialize state with { ids: [], entities: {} }
const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState({
    loading: false // You can still add custom state properties!
  }),
  reducers: {
    // RTK generates perfectly optimized CRUD reducers for you
    userAdded: usersAdapter.addOne,
    usersReceived: usersAdapter.setAll,
    userUpdated: usersAdapter.updateOne,
    userRemoved: usersAdapter.removeOne,
  }
});

// 3. Export highly optimized selectors
export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
} = usersAdapter.getSelectors((state) => state.users);

```

### 6. Production Ready RTK & Best Practices

1. **Feature-Folder Structure:** Group files by feature (`/features/auth/authSlice.ts`), not by type (`/reducers`, `/actions`).
2. **TypeScript Integration:** Always export typed versions of `useDispatch` and `useSelector` to avoid typing `(state: RootState)` in every component.

```typescript
// app/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

```

3. **RTK Query:** If you are only using Thunks to fetch data, cache it, and show loading spinners, **stop**. Use RTK Query (included in the `@reduxjs/toolkit` package). It replaces thunks entirely for data fetching, acting like React Query but deeply integrated into the Redux store.

### 7. When to Use It and When Not To

| Scenario | Recommendation | Why? |
| --- | --- | --- |
| **Complex Client State** | ✅ **USE** | E-commerce carts, multi-step wizards, collaborative canvases, or audio/video player states. |
| **Heavy Cross-Component Interaction** | ✅ **USE** | When Component A needs to update state that Components B, C, and D rely on, and they are located in completely different parts of the DOM tree. |
| **Server Data Caching Only** | ❌ **AVOID** | If your Redux store is just a mirror of your database responses, use RTK Query or React Query instead. They handle caching and invalidation automatically. |
| **Local UI State** | ❌ **AVOID** | Form inputs, "isDropdownOpen", or temporary tab selections should stay in local `useState`. Do not pollute the global store with ephemeral UI state. |

### 8. Tricky Concepts & Gotchas

* **The Immer "Return vs Mutate" Trap:** In a slice, you can *either* mutate the state directly (`state.value = 5`) OR return a brand new state object (`return { value: 5 }`). If you do both, Immer crashes.
* **Non-Serializable Data:** Never put `new Date()`, `Map()`, `Set()`, or functions into Redux state or actions. It breaks the Redux DevTools time-travel debugging. Always serialize to strings or arrays first.
* **Stale Closures in Async Thunks:** When using `thunkAPI.getState()` inside an async thunk, remember that the state might have changed during the `await` period. Always get the state *after* the `await` if you need to make decisions based on it.

### 9. Common Interview Questions

**Q: How does Redux Toolkit mutate state without breaking Redux's strict immutability rule?**
**Answer:** RTK uses the Immer library. When you write code that looks like a mutation in a slice, Immer actually applies that mutation to a temporary "draft" proxy object. Once the reducer finishes, Immer safely calculates the differences and creates a brand-new immutable object under the hood.

**Q: Explain the difference between `reducers` and `extraReducers` in a slice.**
**Answer:** `reducers` are used to define synchronous logic and automatically generate corresponding action creators based on the keys you provide. `extraReducers` are used to respond to actions defined *outside* of the slice, most commonly actions generated by `createAsyncThunk` or actions from other slices. They do not generate new action creators.

**Machine Coding Snippet:**
*Prompt: "Write a selector that gets all users from a state array, but memoize it so it doesn't cause re-renders if the underlying data hasn't changed."*

```javascript
import { createSelector } from '@reduxjs/toolkit';

// Basic input selector
const selectUsers = (state) => state.users.items;
const selectFilter = (state) => state.users.activeFilter;

// Memoized selector using createSelector
export const selectActiveUsers = createSelector(
  [selectUsers, selectFilter],
  (users, filter) => {
    // This calculation ONLY runs if 'users' or 'filter' reference changes.
    return users.filter(user => user.status === filter);
  }
);
```
