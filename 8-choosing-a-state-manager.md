# React state management

- [React state management](#react-state-management)
   * [TLDR](#tldr)
   * [Comparing Redux with MobX](#comparing-redux-with-mobx)
      + [Redux](#redux)
         - [Pros](#pros)
         - [Cons](#cons)
      + [MobX](#mobx)
         - [Pros](#pros-1)
         - [Cons](#cons-1)
      + [Why MobX cannot scale like Redux?](#why-mobx-cannot-scale-like-redux)
      + [Conclusion: When to Use Which?](#conclusion-when-to-use-which)
      + [Bundle sizes](#bundle-sizes)
   * [Comparing Zustand with Redux](#comparing-zustand-with-redux)
      + [Zustand benefits](#zustand-benefits)
      + [Drawbacks of Zustand](#drawbacks-of-zustand)
      + [How is Zustand scalable?](#how-is-zustand-scalable)
      + [Conclusion](#conclusion)
   * [Comparing Jotai with Redux](#comparing-jotai-with-redux)
      + [Differences with Redux](#differences-with-redux)
      + [Redux is still better than Jotai for large scale apps](#redux-is-still-better-than-jotai-for-large-scale-apps)
   * [Comparing Xstate with Redux ](#comparing-xstate-with-redux)

Types of *client state*:
1. Local state (Ex: `useState`)
2. Global state (Ex: `useContext`)
3. Server state (Ex: API data)
4. URL parameters

## TLDR

[Source](https://www.youtube.com/watch?v=8hi3lbkR0GI)

![Selection flowchart for react sstate management libraries](https://i.imgur.com/8mk2oAd.png)

Tips:
* Start with the simplest libraries
* Do not over-engineer
* Use the heaviest ones like redux only for enterprise large-scale apps that are complex

Use **Redux** for 
1. Enterprise level complex apps i.e large apps 
	* Huge ecosystem of developers
	* Huge ecosystem of tools i.e libraries

Use **alternative** libraries for:
1. Less boilerplate (However, Redux Toolkit solves this problem)
2. Easier learning curve (Redux has a steep one)

 Use **MobX** for:
1. When you need a simple Singleton pattern
2. Minimal setup
 
**Cons of MobX**:
1. Bi-directional data flow
2. Has mutable state

Use **Zustand** for:
1. Minimal setup
2. Simple to use
3. Lightweight

**Cons of Zustand**:
1. Less structured for large apps

Use **Jotai** (Or Recoil) for:
1. Atom based setup for granular state control
2. Minimal setup
3. Works well with React suspense

**Cons of Jotai**:
1. Limited features for larger apps

Use **TANStack query** (Formerly React Query) for:
1. Handling API data: i.e API heavy apps
2. Provides caching of API data on the client-side
3. Syncs data between server and client

## Comparing Redux with MobX

**MobX automatically tracks state changes** and mutates state **directly**, making debugging more challenging in complex cases. 

**Redux forces explicit updates**, making it more predictable.

[Source](https://www.youtube.com/watch?v=83v8cdvGfeA)

### Redux

#### Pros
1. **Manual**: Have to define everything **explicitly**
2. Actions are dispatched > Handled by middleware (Ex: Redux Thunk) > Pass through a reducer > Final State
3. Subscribers listen to the state changes and react
4. This whole setup gives us **immense control** and makes it very **predictable**: **Excellent for testing!**

#### Cons
1. Everything is manual so there is a **lot of boilerplate code** (Ex: dispatching actions, subscribing, etc)
2. Use of **multiple libraries** developed to get over this boilerplate code

**Note**: `redux-toolkit` library helps reduce the amount of boilerplate code we need to write with Redux

### MobX

#### Pros
1. **Automatic**: Same goals as redux but does it in a more **magical way**
2. MobX has values called **observable values**. When it updates (mutates), everything that depends on it updates as well!
3. Do not need to dispatch actions, maintain a central store, etc. No such concepts (Usually, MobX does this internally for you so it is smarter)

#### Cons
1. **Not as predictable** and has **less developer control** i.e cannot test how the mutations happen internally - probably do not need to as well. But, this makes the MobX instance itself **hard to test** (We can only test the mutating changes)

| Feature | MobX | React |
| ---------- |--------- | ---------| 
| Philosophy | Reactive & automatic | Preditable with Unidirectional data flow |
| Performance | Fine-grained (auto) | Can be optimized (more control but also prone to developer errors) |
| Scalability | Difficult to debug with scale | Structured, easy to manage | 
| Debugging Tools | Less built-in support | Time travel debugging with Redux Devtools |
| Middleware Support | Not required but supports custom middleware | Supports middleware like `redux-thunk` or `redux-saga` |
| Asynchronous Handling	| In-built (smooth) | Need to use a middleware (Ex: `redux-thunk`) |
| Best For | Small to medium projects | Large applications |

### Why MobX cannot scale like Redux?

MobX is not ideal for large-scale applications for the following reasons:
1. Automatically tracks state updates: For large apps, implicit updates can make debugging harder. This can lead to **accidental re-renders** and **unexpected side effects**
2. Unlike Redux or Zustand, MobX relies on **observable state** and automatically tracks dependencies. While this is great for small apps, in large applications, it can become unpredictable
3. **No Time-Travel Debugging** & **action tracking**
```js
// REACT
dispatch({ type: "LOGIN", payload: { name: "Alice" } });
```
```js
// MobX
userStore.user = { name: "Alice" }; // No action tracking
```
4. Performance Issues in Large Apps: MobX is optimized for fine-grained updates, but in **large applications with many observers**, it can lead to **unnecessary reactivity and performance bottlenecks**.
5. Redux enforces **strict architecture** (`actions → reducers → store`), making it predictable and maintainable. MobX, on the other hand, **allows direct state mutations,** which can make codebases **harder to manage** in large teams.
```js
// MobX
userStore.user.name = "Charlie"; // Direct mutation, no structure
```
```js
// Redux
dispatch(updateUserName("Charlie")); // Clear action → reducer → state update
```
6. MobX can handle async actions, but it **lacks strong middleware support**. Hence, **Harder to control async flows** (e.g., race conditions, retries).
```js
// MobX
async function fetchUser() {
  const res = await fetch("/api/user");
  userStore.user = await res.json(); // Directly mutating state
}
```
```js
// Redux
export const fetchUser = () => async (dispatch) => {
  const res = await fetch("/api/user");
  dispatch(setUser(await res.json())); // Dispatching an explicit action
};
```

### Conclusion: When to Use Which?
✅ Use MobX if you prefer less boilerplate, automatic reactivity, and mutability.
✅ Use Redux if you need strict structure, time-travel debugging, or predictable state flow in large-scale applications.

Redux is better for large scale applications:
Ex: **"Authentication"** with log in, log out, and status check components: 
* Redux ensures that updates follow a predictable, traceable flow.
* Can integrate authentication logic with `redux-thunk` or `redux-saga`
* Useful for managing user state across multiple components.
* Easily debug authentication state changes using Redux DevTools.

### Bundle sizes

* Mobx: 64kB minified, 18kB gzipped
* Redux: 4kB minified, 1.4kB gzipped
* React-redux: 11kB minified, 4kB gzipped
* Redux-toolkit: 10kB minified, 3.6kB gzipped
(Total size of redux libraries collection: 25kB minified)

## Comparing Zustand with Redux

[Source](https://www.youtube.com/watch?v=viLIjOxdzpk)

### Zustand benefits
1. **Less boilerplate code** (Easier to use)
2. Can have **multiple stores** (Ex: data store and auth store)
	* *Note*: We can *only have one* store in Redux
3. **Hooks based architecture** of Zustand
	* Provides hooks out of the box
4. **Extremely lightweight** compared to React:
	* Zustand: 1.2kB minified, 588B gzipped
	* Redux (+ react-redux + redux-saga): 25kB minified
5. **Do not need to wrap our application in a provider**. Single library which provides hooks which work without a provider
6. **Performance**: Zustand internally **uses useMemo and auto subscription** for better performance
7. **No middleware needed** in Zustand i.e async by nature

### Drawbacks of Zustand
1. **Mutable state and automatic subscriptions** can become a debugging bottleneck (Redux is strictly immutable, unidirectional data flow)
2. It **lacks powerful debugging tools** (which Redux has with Redux DevTools)
3. It **lacks integration complex middleware** (e.g., with Redux you have `redux-thunk`, `redux-saga`).

| Feature | Zustand | Redux |
| --- | --- | --- |
| State Structure | Flexible, mutable state	| Immutable state with strict reducers |
| Performance | Fine-grained updates, avoids unnecessary re-renders | Can cause unnecessary re-renders without optimizations |
| Middleware | Supports simple middleware but less ecosystem support | Strong middleware ecosystem (e.g., redux-thunk, redux-saga) |
| Asynchronous Handling	| Works with async functions out of the box | Requires middleware (`redux-thunk`, `redux-saga`) |
| DevTools Support | Experimental DevTools support | Powerful Redux DevTools |
| Best For | Small to medium projects, local state | Large-scale applications requiring strict state management |

### How is Zustand scalable?

1. No unnecessary state re-renders: Zustand **updates only the components that depend on specific slices of state**.
	* In Redux, we need to be careful about what the `useSelector` picks 
```js
import { create } from "zustand";

// Store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Component using selector to prevent unnecessary re-renders
const Counter = () => {
  const count = useStore((state) => state.count); // Only re-renders when count changes
  return <p>Count: {count}</p>;
};

const IncrementButton = () => {
  const increment = useStore((state) => state.increment);
  return <button onClick={increment}>Increment</button>;
};
```
-   `Counter` **only re-renders** when `count` changes.
-   `IncrementButton` **never re-renders** unless `increment` changes.
2. **No need for boilerplate**: Zustand can add new features without touching reducers, store, etc.
	* Redux requires **actions, reducers, dispatch, and middleware**, which can become **complex in large projects**. However, a **`redux-toolkit`** library helps us reduce and manage this boilerplate.
3. Zustand has a **multi-store architecture** unlike Redux: Again, components only subscribe to **relevant stores**, reducing unnecessary updates.
```js
const useUserStore = create((set) => ({
  name: "Alice",
  setName: (name) => set({ name }),
}));

const useThemeStore = create((set) => ({
  darkMode: false,
  toggleDarkMode: () => set((state) => ({ darkMode: !state.darkMode })),
}));

// Using separate stores in components
const UserProfile = () => {
  const name = useUserStore((state) => state.name);
  return <p>Welcome, {name}!</p>;
};

const ThemeToggle = () => {
  const toggleDarkMode = useThemeStore((state) => state.toggleDarkMode);
  return <button onClick={toggleDarkMode}>Toggle Theme</button>;
};
```
4. Zustand supports **middleware for logging, persistence, and async operations**, making it suitable for large applications. Zustand allows async functions inside stores without extra middleware.
```js
const useStore = create((set) => ({
  data: null,
  fetchData: async () => {
    const response = await fetch("https://jsonplaceholder.typicode.com/todos/1");
    const json = await response.json();
    set({ data: json });
  },
}));

const DataComponent = () => {
  const { data, fetchData } = useStore();
  return (
    <div>
      <button onClick={fetchData}>Load Data</button>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};
```
5. **Server-Side Rendering (SSR) Friendly in Next.js**: Works without hydration mismatches. Uses vanilla Zustand to avoid client-server hydration issues.
```js
import { createStore } from "zustand/vanilla";

export const store = createStore(() => ({
  count: 0,
  increment: () => store.setState((state) => ({ count: state.count + 1 })),
}));

// ...

"use client";
import { useStore } from "./store";

const Counter = () => {
  const count = useStore((state) => state.count);
  return <p>Count: {count}</p>;
};
```

### Conclusion

✅ Why Zustand?

- No need for reducers, actions, or selectors.
- Simple set function for state updates.

✅ Why Redux?

-   **More structure, better debugging** with Redux DevTools.
-   Useful for **complex applications** with **strict state logic**.
- **Zustand does not natively support time travel debugging** like Redux, which has built-in integration with Redux DevTools to replay past states. However, you can implement time travel manually in Zustand by saving previous states in an array.

---

* Use Redux for enterprise apps needing strict state control. 
* Use Zustand if you want simplicity, performance, and scalability without complexity


| Middleware Type | Redux | Zustand |
| ----- | ----- | ---- |
| Persistence | Needs `redux-persist` | ✅ `persist()` (built-in) |
| DevTools | Needs `redux-devtools` extension | ✅ `devtools()` (built-in) |
| Logging | Needs `redux-logger` | ✅ Simple function wrapping in store |
| Async (Thunks, Sagas) | Needs `redux-thunk` or `redux-saga`	| ✅ Native support (async functions in store) |
| State Freezing (`Immutable.js`, `Immer`) | Needs immer | ✅ immer() (built-in middleware) |

## Comparing Jotai with Redux

[Source](https://www.youtube.com/watch?v=0BaqAEkvws8)

Jotai is different from other libraries:
1. It is composed of **atoms** ("values")
2. We can combine atoms, i.e mix and match values, and create new states. We compose state from the **bottom-up** rather than top-down like in Redux
3. The way we set and get state is via a React's `useState()` like interface within components

### Differences with Redux
- It has a lower boilerplate code than Redux. 
- Jotai library size: 9kB minified, 3.5kB gzipped
- Built-in async atoms: No need for middleware (also, no support for custom middleware)

### Redux is still better than Jotai for large scale apps

Reasons:
1. It has a **global immutable store** (single source of truth)
2. It has a **rich middleware ecosystem** (e.g., `redux-thunk`, `redux-saga`) whereas Jotai has none (but can be extended)
3. It has **advanced Redux DevTools (time-travel debugging)**

**Note**: If you like atom based state and fine-grained control, you can check out **Jotai** or **Recoil**

## Comparing Xstate with Redux 

Xstate is based on **finite state machines (FSMs)**
- Events trigger transitions in a finite state
- Reacts based on state transitions
- Less traditional middleware, but supports effects
- Best for **complex component states**
- Debugging: Visual statechart debugger

Debugging:
- Redux is better: Redux DevTools (time-travel debugging)
- However, Xstate has visual state changes' debugging and time-travel debugging is possible (due to statehcart history) but not fully supported

🚀 Use XState for UI-heavy state logic & complex workflows.  
🔥 Use Redux for large-scale global state management.

