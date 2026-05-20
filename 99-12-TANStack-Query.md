# TANStack Query

- [TANStack Query](#tanstack-query)
  - [1. The Origins: What is the "TanStack"?](#1-the-origins-what-is-the-tanstack)
  - [2. What is TanStack Query?](#2-what-is-tanstack-query)
  - [3. The Core Concept: Stale-While-Revalidate](#3-the-core-concept-stale-while-revalidate)
  - [4. Installation](#4-installation)
  - [5. Quick Start: The 3 Steps to Querying](#5-quick-start-the-3-steps-to-querying)
    - [Step 1: Create the Client & Wrap the App](#step-1-create-the-client-wrap-the-app)
    - [Step 2: Write an Asynchronous Fetcher Function](#step-2-write-an-asynchronous-fetcher-function)
    - [Step 3: Call `useQuery`](#step-3-call-usequery)
  - [6. Production-Grade Best Practices](#6-production-grade-best-practices)
    - [A. The Query Key Factory](#a-the-query-key-factory)
    - [B. Custom Hooks for Everything](#b-custom-hooks-for-everything)
    - [C. Mutations & Invalidation (Updating Data)](#c-mutations-invalidation-updating-data)
  - [7. Tricky Concepts & Massive Gotchas](#7-tricky-concepts-massive-gotchas)
    - [Gotcha 1: `staleTime` vs `gcTime` (formerly `cacheTime`)](#gotcha-1-staletime-vs-gctime-formerly-cachetime)
    - [Gotcha 2: V5 API Changes (Callbacks Removed)](#gotcha-2-v5-api-changes-callbacks-removed)
    - [Gotcha 3: The `enabled` flag for Dependent Queries](#gotcha-3-the-enabled-flag-for-dependent-queries)
  - [8. When to Use It and When Not To](#8-when-to-use-it-and-when-not-to)
  - [9. Comparison with Other Tools](#9-comparison-with-other-tools)
  - [10. DevTools Mastery](#10-devtools-mastery)
  - [11. Common Interview Questions](#11-common-interview-questions)
    - [Conceptual Questions](#conceptual-questions)
    - [Machine Coding Snippet](#machine-coding-snippet)
  - [Sharing data fetched via TANStack Query](#sharing-data-fetched-via-tanstack-query)
    - [1. The Native Way: Sharing Across the Tree](#1-the-native-way-sharing-across-the-tree)
      - [The Concept: The Global Cache](#the-concept-the-global-cache)
      - [The Best Practice: Custom Hooks](#the-best-practice-custom-hooks)
    - [2. Snippets & Fragments: The `select` Option](#2-snippets-fragments-the-select-option)
      - [Code Example: Snippet Extraction](#code-example-snippet-extraction)
    - [3. The Redux Dilemma: Should You Sync Server Data to a Global Store?](#3-the-redux-dilemma-should-you-sync-server-data-to-a-global-store)
      - [The Exception: When IS it useful?](#the-exception-when-is-it-useful)
      - [How to Sync to Redux (The Proper Way)](#how-to-sync-to-redux-the-proper-way)
    - [4. Tricky Concepts & Gotchas](#4-tricky-concepts-gotchas)
      - [Gotcha 1: Unstable `select` Functions](#gotcha-1-unstable-select-functions)
      - [Gotcha 2: The "Undefined" Flash](#gotcha-2-the-undefined-flash)
      - [Gotcha 3: Stale Time vs. Background Fetching](#gotcha-3-stale-time-vs-background-fetching)
    - [5. Common Interview Questions](#5-common-interview-questions)
      - [Conceptual Questions](#conceptual-questions)
      - [Machine Coding Snippet](#machine-coding-snippet)

TanStack Query (formerly known as React Query) has fundamentally changed how developers think about state management in front-end applications. By drawing a hard line between "Client State" and "Server State," it eliminates massive amounts of boilerplate code and solves complex caching problems out of the box.

Here is a comprehensive deep dive into TanStack Query, optimized for production-grade React architecture.

## 1. The Origins: What is the "TanStack"?

**Tanner Linsley** (an open-source developer) originally created React Query around 2019. At the time, developers were stuffing all their API responses into Redux, requiring dozens of action creators, reducers, and thunks just to fetch and cache a list of users. React Query solved this by abstracting the fetch/cache lifecycle into a single Hook.

As the library exploded in popularity, the core logic was decoupled from React so it could be used with Vue, Svelte, Solid, and Vanilla JS. It was rebranded as **TanStack Query** (a play on "Tanner's Stack").

## 2. What is TanStack Query?

TanStack Query is an **asynchronous state management library** specifically designed for **Server State**.

* **Client State:** Ephemeral data that your app controls (e.g., "Is this modal open?", "What is typed in this form?"). This belongs in `useState`, Zustand, or Context.
* **Server State:** Data persisted remotely that you do not control (e.g., a database of users). It is asynchronous, requires shared ownership, and can become out-of-date at any moment.

TanStack Query acts as a localized cache for your Server State. It handles caching, background updates (stale-while-revalidate), deduplication of identical requests, pagination, and garbage collection automatically.

## 3. The Core Concept: Stale-While-Revalidate

TanStack Query is built on the "Stale-While-Revalidate" caching strategy. When a component asks for data, the library checks its cache first.

```text
      THE TANSTACK QUERY LIFECYCLE
      ----------------------------
      
   [ Component ] calls useQuery({ queryKey: ['user', 1] })
          |
   Is data in Cache?
      /         \
    YES         NO -----> [ Network Request ]
     |                       |
 Is it Stale?            (Saves to Cache)
   /      \                  |
  NO      YES                V
  |        |             [ Component Renders ]
  |        V             
  |  1. Return Cache instantly (UI feels lightning fast)
  |  2. Fire Background Fetch silently
  |  3. When new data arrives, update Cache
  |  4. If data changed, trigger Re-render
  V
[ Component Renders immediately ]

```

## 4. Installation

Install the core package and the official DevTools (which are highly recommended).

```bash
# Using npm
npm i @tanstack/react-query
npm i -D @tanstack/react-query-devtools

# Using yarn
yarn add @tanstack/react-query
yarn add -D @tanstack/react-query-devtools

```

## 5. Quick Start: The 3 Steps to Querying

To use TanStack Query, you need three things: a Client, a Provider, and a Hook.

### Step 1: Create the Client & Wrap the App

```jsx
// app.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import UserProfile from './UserProfile';

// 1. Create a client (holds the cache)
const queryClient = new QueryClient();

export default function App() {
  return (
    // 2. Provide the client to your App
    <QueryClientProvider client={queryClient}>
      <UserProfile />
      {/* Devtools only bundle in development mode */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

```

### Step 2: Write an Asynchronous Fetcher Function

TanStack Query doesn't care *how* you fetch data (Fetch, Axios, GraphQL). It just needs a function that returns a Promise containing data (or throws an error).

```jsx
// api.ts
export const fetchUser = async (userId: number) => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
  if (!response.ok) throw new Error('Network response was not ok');
  return response.json();
};

```

### Step 3: Call `useQuery`

```jsx
// UserProfile.tsx
import { useQuery } from '@tanstack/react-query';
import { fetchUser } from './api';

export default function UserProfile() {
  // 3. Bind the query key to the fetcher function
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['user', 1], // Unique identifier for this exact data
    queryFn: () => fetchUser(1), // The function that gets the data
  });

  if (isLoading) return <span>Loading...</span>;
  if (isError) return <span>Error: {error.message}</span>;

  return <h1>{data.name}</h1>;
}

```

## 6. Production-Grade Best Practices

When building enterprise apps, do not scatter `useQuery` calls everywhere. Follow these architectural patterns.

### A. The Query Key Factory

Query Keys are arrays (e.g., `['users', 'list', { filters: 'active' }]`). If you mistype a key, the cache breaks. Centralize them in a factory.

```jsx
// userKeys.ts
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: string) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: number) => [...userKeys.details(), id] as const,
};

```

### B. Custom Hooks for Everything

Never write `useQuery` directly in a UI component. Wrap them in custom hooks to ensure consistent keys and configuration.

```jsx
// useUser.ts
import { useQuery } from '@tanstack/react-query';
import { userKeys } from './userKeys';
import { fetchUser } from './api';

export const useUser = (userId: number) => {
  return useQuery({
    queryKey: userKeys.detail(userId),
    queryFn: () => fetchUser(userId),
    // Production configs often go here
    staleTime: 1000 * 60 * 5, // Data is fresh for 5 minutes
  });
};

```

### C. Mutations & Invalidation (Updating Data)

When you update data on the server (POST/PUT/DELETE), the local cache becomes invalid. You use `useMutation` to perform the update, and `queryClient.invalidateQueries` to wipe the cache and trigger an automatic background refetch.

```text
      MUTATION AND INVALIDATION FLOW
      ------------------------------
      
  [ Component ] calls mutate({ name: "Alice" })
          |
  [ useMutation ] sends POST to Server
          |
     (Success!)
          |
  [ onSuccess callback ] calls invalidateQueries(['users'])
          |
  [ QueryClient ] marks all 'users' cache as stale
          |
  [ Active useQuery hooks ] instantly refetch from Server
          |
  [ UI Updates Automatically ]

```

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

export const useUpdateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (updatedUser) => axios.put(`/users/${updatedUser.id}`, updatedUser),
    onSuccess: (data, variables) => {
      // Wipes the cache for this specific user, forcing a refetch
      queryClient.invalidateQueries({ queryKey: userKeys.detail(variables.id) });
      // Wipes the cache for the entire users list
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
};

```

## 7. Tricky Concepts & Massive Gotchas

### Gotcha 1: `staleTime` vs `gcTime` (formerly `cacheTime`)

This is the #1 misunderstanding in TanStack Query.

* **`staleTime` (Default: 0 ms):** How long data is considered "fresh". By default, data is instantly stale. This means the next time a component mounts, a background refetch is triggered.
* **`gcTime` (Default: 5 minutes):** Garbage Collection Time. How long data remains in memory *after all components using it have unmounted*.

**The Fix:** In production, you almost always want to set a global `staleTime` to avoid spamming your API every time a user switches tabs.

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute before background refetching
    },
  },
});

```

### Gotcha 2: V5 API Changes (Callbacks Removed)

In TanStack Query v5, the `onSuccess` and `onError` callbacks were **removed** from `useQuery`. You can no longer trigger side effects directly from a query (like showing a toast notification when a fetch succeeds). You must handle side effects via `useEffect` tracking the `isSuccess` flag, or handle them at the fetcher level. (Note: `onSuccess` still exists for `useMutation`).

### Gotcha 3: The `enabled` flag for Dependent Queries

If Query B requires an ID returned by Query A, Query B will fail if it runs immediately. Use the `enabled` flag to hold it back.

```jsx
const { data: user } = useUser(userId);

const { data: projects } = useQuery({
  queryKey: ['projects', user?.teamId],
  queryFn: () => fetchTeamProjects(user!.teamId),
  // ONLY run this query if user and teamId exist
  enabled: !!user?.teamId, 
});

```

## 8. When to Use It and When Not To

| Scenario | Recommendation | Why? |
| --- | --- | --- |
| **REST APIs & GraphQL** | ✅ **USE** | Replaces hundreds of lines of Redux Thunk boilerplate with a declarative hook. |
| **Frequently Updated Dashboards** | ✅ **USE** | Background refetching and window-focus refetching keep data seamlessly synchronized. |
| **Local Form State** | ❌ **AVOID** | Form inputs (what the user is typing right now) should be managed by React Hook Form or local `useState`. |
| **Global Theme / UI Toggles** | ❌ **AVOID** | TanStack is for *Server* State. If you just need a Dark Mode toggle shared across the app, use the Context API or Zustand. |

## 9. Comparison with Other Tools

| Tool | Focus | Difference vs TanStack Query |
| --- | --- | --- |
| **Redux (RTK)** | Client & Server State | Redux requires manual cache management and heavy boilerplate. RTK Query is Redux's answer to TanStack Query and is highly comparable, but TanStack is framework-agnostic and slightly easier to set up. |
| **SWR (by Vercel)** | Server State | SWR is lighter and simpler, but TanStack Query has significantly more powerful DevTools, better Mutation handling, and finer control over garbage collection. |
| **Apollo Client** | GraphQL Server State | If your entire stack is GraphQL, Apollo is purpose-built for it (and normalizes its cache). TanStack Query works with any Promise (REST, RPC, GraphQL) but does not do deep graph normalization. |

## 10. DevTools Mastery

The `@tanstack/react-query-devtools` provide a floating widget in your app. It is your best friend for debugging.

* **The Cache Explorer:** See exactly what keys exist in the cache and their current state (`Fresh`, `Fetching`, `Stale`, `Inactive`).
* **Data Viewer:** Inspect the raw JSON currently held in memory for any query key.
* **Action Buttons:** You can manually trigger a `Refetch`, `Invalidate`, or `Remove` on any query to test how your UI reacts to cache wiping.
* **Network Simulator:** Simulate offline mode to test your app's resilience.

## 11. Common Interview Questions

### Conceptual Questions

**Q: Explain how TanStack Query handles data deduplication.**
**Answer:** If five different components on the same page all call `useQuery({ queryKey: ['user', 1] })`, TanStack Query intercepts them. It fires only *one* network request. When the Promise resolves, it broadcasts the data to all five components simultaneously.

**Q: What is an Optimistic Update and how do you implement it in TanStack Query?**
**Answer:** An optimistic update assumes a mutation will succeed and updates the UI instantly, before the server responds. This makes the app feel incredibly fast. In TanStack Query, you use the `onMutate` callback in `useMutation` to manually update the cache using `queryClient.setQueryData()`. If the mutation fails, you use the `onError` callback to roll the cache back to its previous state.

### Machine Coding Snippet

**Prompt:** *"Write a custom hook that fetches a paginated list of posts. It should keep the previous data on the screen while the next page loads to avoid layout shift."*

**Solution:**
In TanStack Query, we use the `placeholderData: keepPreviousData` (v5 syntax) to achieve seamless pagination.

```jsx
import { useQuery, keepPreviousData } from '@tanstack/react-query';

const fetchPosts = async (page: number) => {
  const res = await fetch(`/api/posts?page=${page}`);
  return res.json();
};

export const usePosts = (page: number) => {
  return useQuery({
    queryKey: ['posts', { page }],
    queryFn: () => fetchPosts(page),
    // This keeps the old page's data visible while the new page fetches,
    // avoiding the flickering "Loading..." screen.
    placeholderData: keepPreviousData,
  });
};

```

## Sharing data fetched via TANStack Query

The most fundamental mental shift required when adopting TanStack Query is this: **You no longer pass server data through the component tree.**

You do not pass it down as props. You do not lift it up to parents. You do not put it in a global Context.

Instead, TanStack Query acts as an invisible, omnipresent data layer hovering above your entire application. Any component, anywhere in the tree, can simply reach up and grab the exact data snippet it needs.

Here is a comprehensive deep dive into how to share API data across your app, extract specific snippets, and the highly debated topic of syncing this data with Redux.

### 1. The Native Way: Sharing Across the Tree

#### The Concept: The Global Cache

When you call `useQuery` with a specific `queryKey`, TanStack Query creates a dedicated slot in its central cache. If you call `useQuery` with the **exact same key** in a sibling, a parent, or a component 10 levels deep, TanStack Query will *intercept* the call.

Instead of making a new API request, it instantly hands that component the data from the cache.

```text
      THE OMNIPRESENT CACHE MODEL
      ---------------------------
      
           [ TANSTACK QUERY CACHE ]
           Key: ['user', 1] -> { id: 1, name: "Alice", role: "Admin" }
             |             |             |
     (Reads Cache)    (Reads Cache)  (Reads Cache)
             |             |             |
      [Header UI]     [Sidebar UI]  [Deeply Nested Child]
      (Parent)        (Sibling)     (Far-off Component)

      * Result: 1 Network Request, 3 Components instantly synced.

```

#### The Best Practice: Custom Hooks

To share data perfectly, never write `useQuery` directly in your UI components. If you mistype the query key array in a distant component, you will trigger a duplicate API call. Always wrap it in a custom hook.

```jsx
// hooks/useUser.ts
import { useQuery } from '@tanstack/react-query';

const fetchUser = async (id) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
};

export const useUser = (id) => {
  return useQuery({
    queryKey: ['user', id], // The single source of truth for this key
    queryFn: () => fetchUser(id),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
};

```

**Sharing it everywhere:**

```jsx
// Component A (Far top of the tree)
const Header = ({ id }) => {
  const { data } = useUser(id); 
  return <nav>Welcome, {data?.name}</nav>;
};

// Component B (Deeply nested)
const UserRoleBadge = ({ id }) => {
  const { data } = useUser(id); // Instantly gets data from cache! No network call.
  return <span>{data?.role}</span>;
};

```

### 2. Snippets & Fragments: The `select` Option

What if your API returns a massive user object (100+ properties), but your `UserRoleBadge` component ONLY needs the `role` string?

If you use the standard hook, every time *any* part of the user object updates, the `UserRoleBadge` will unnecessarily re-render. To extract "snippets" of data and optimize performance, you use the `select` option.

```text
      THE 'SELECT' TRANSFORMATION
      ---------------------------
      
      [ API RESPONSE ] 
      { id: 1, name: "Alice", role: "Admin", history: [...1000 items] }
             |
        (Saved to Cache)
             |
      [ useQuery with 'select' ]
             |
             +---> select: (data) => data.role
             |
      [ UserRoleBadge Component ]
      (Receives ONLY "Admin". Ignores changes to 'history' or 'name')

```

#### Code Example: Snippet Extraction

```jsx
export const useUserRole = (id) => {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
    // The select function transforms the data BEFORE giving it to the component.
    // The component will ONLY re-render if 'data.role' specifically changes.
    select: (data) => data.role, 
  });
};

export const useUserPermissions = (id) => {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
    // You can filter arrays or compute derived state here!
    select: (data) => data.permissions.filter(p => p.isActive),
  });
};

```

### 3. The Redux Dilemma: Should You Sync Server Data to a Global Store?

**The General Rule:** **NO.**
Syncing TanStack Query data into Redux, Zustand, or Context is heavily considered an **anti-pattern**.

TanStack Query *is* your global state manager for server data. If you copy data from the query cache into Redux, you create a **Double Source of Truth**. When data gets updated in the background, your Redux store won't know, and you will render stale, buggy UI.

#### The Exception: When IS it useful?

There is exactly **one** major use case for copying API data into Redux/Zustand: **Complex Client-Side Drafting (The "Save for Later" pattern).**

If the user needs to heavily mutate the data locally, across multiple screens, without saving it to the server immediately, you need a local sandbox.

* **Valid Use Case:** A complex multi-step checkout wizard, a drag-and-drop canvas (like Figma), or an offline-first drafting tool.
* **Workflow:** Fetch the baseline data from the server -> Copy it to Redux -> Let the user mutate the Redux state locally for 20 minutes -> Send the final Redux state back to the server via a Mutation.

```text
      THE "DRAFTING SANDBOX" ARCHITECTURE
      -----------------------------------
      
  [ Server Database ]
          |
  [ TanStack Query ] (Fetches baseline data: "Draft V1")
          |
  (Copies via useEffect)
          |
  [ REDUX STORE ] ---> User edits locally (Title, Colors, Layout)
          |            State becomes "Draft V2" (Server doesn't know yet)
          |
  (User clicks "Publish")
          |
  [ TanStack useMutation ] -> Sends "Draft V2" to Server

```

#### How to Sync to Redux (The Proper Way)

Because TanStack Query v5 removed the `onSuccess` callback from `useQuery`, the most reliable way to sync data to a global store is by using a `useEffect` to watch the query data.

```jsx
import { useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { useDispatch } from 'react-redux';
import { setDraftData } from './draftSlice';

export const DraftEditorWrapper = ({ documentId }) => {
  const dispatch = useDispatch();
  
  // 1. Fetch the baseline data from the server
  const { data: serverDocument, isSuccess } = useQuery({
    queryKey: ['document', documentId],
    queryFn: () => fetchDocument(documentId),
    // IMPORTANT: When syncing to Redux for drafting, you often don't want 
    // background refetches wiping out the user's local progress!
    staleTime: Infinity, 
  });

  // 2. Sync it to Redux ONCE when the fetch succeeds
  useEffect(() => {
    if (isSuccess && serverDocument) {
      // Copy the server data into the Redux sandbox
      dispatch(setDraftData(serverDocument));
    }
  }, [isSuccess, serverDocument, dispatch]);

  // 3. The actual editor components will now read/write STRICTLY from Redux,
  // completely ignoring TanStack Query until it's time to save.
  return <ComplexReduxEditor />;
};

```

### 4. Tricky Concepts & Gotchas

#### Gotcha 1: Unstable `select` Functions

If you write your `select` function inline and it returns a *new array or object*, you will cause infinite rendering loops. TanStack Query checks if the selected data changed via strict equality (`===`).

* **Bad:** `select: (data) => data.items.map(i => i.name)` (Creates a new array reference every render).
* **Good:** Wrap the select function in `useCallback`, or define it outside the component so its reference is stable.

#### Gotcha 2: The "Undefined" Flash

When a component mounts and grabs data from the cache, there is a microsecond where `data` might be undefined while the cache resolves. Always defensively check `if (!data) return null;` or use React Suspense.

#### Gotcha 3: Stale Time vs. Background Fetching

If Component A fetches Data X, and 10 seconds later Component B mounts and asks for Data X... TanStack Query will instantly give Component B the cached data, **but it will also silently fire a background fetch** (because default `staleTime` is 0). If you have 50 components mounting at different times, you might spam your API. Set a reasonable global `staleTime` (e.g., 20 seconds).

### 5. Common Interview Questions

#### Conceptual Questions

**Q: "If a child component deeply nested in the tree needs to trigger a refetch of data used by the parent, how do you pass the refetch function up?"**
**Answer:** You don't pass anything up. The child component simply imports `useQueryClient()` and calls `queryClient.invalidateQueries({ queryKey: ['sharedData'] })`. TanStack Query will automatically refetch the data and update the parent, the child, and any other component listening to that key.

**Q: "Why is copying React Query data into local `useState` or Redux considered bad practice for standard read-only data?"**
**Answer:** It breaks the synchronization link. If you copy query data into local state, and then the query performs a background refetch (Stale-While-Revalidate) and gets fresh data from the server, your local state will be completely unaware. The UI will show outdated, stale data, defeating the entire purpose of the library.

#### Machine Coding Snippet

**Prompt:** *"Write a custom hook that fetches a large configuration object, but exports two separate selectors: one for user settings, and one for admin settings. Ensure components using the user settings don't re-render if the admin settings change."*

**Solution:**

```jsx
import { useQuery } from '@tanstack/react-query';
import { useCallback } from 'react';

const fetchConfig = async () => (await fetch('/api/config')).json();

// Base hook with the query definition
const useBaseConfig = (selectFn) => {
  return useQuery({
    queryKey: ['appConfig'],
    queryFn: fetchConfig,
    select: selectFn, // Accept the selector dynamically
  });
};

// Extracted hook for User Settings
export const useUserSettings = () => {
  // Stable reference prevents unnecessary re-renders
  const selectUser = useCallback((data) => data.userSettings, []);
  return useBaseConfig(selectUser);
};

// Extracted hook for Admin Settings
export const useAdminSettings = () => {
  const selectAdmin = useCallback((data) => data.adminSettings, []);
  return useBaseConfig(selectAdmin);
};
```
