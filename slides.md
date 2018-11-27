class: middle, center

# Hello React!

---

class: middle, center

## A new Architecture

---

class: middle, center

### Some rules

- Try to split as much as possible the logic and the view by using `hooks`
- Doing so is not _mandatory_ but it will make our code easier to reuse
- It may become necessary to "wrap" a `hook` with a `ContextfulComponent`

---

### Example (without hooks)

Let's say we want to fetch some posts from a server with this API: https://jsonplaceholder.typicode.com/posts.

Using the Context API, we can naively implement this kind of solution:

#### Types

```typescript
import React, { Reducer, useEffect, useReducer } from "react";

// Define some types related to the posts

interface State {
  isFetchingPosts: boolean;
  posts: Post[] | null;
}

type Actions =
  | { type: "FETCH_POSTS_INIT" }
  | { type: "FETCH_POSTS_FAILURE"; payload: Error }
  | { type: "FETCH_POSTS_SUCCESS"; payload: Post[] };
```

---

#### Logic - reducer

```typescript
const reducer: Reducer<State, Actions> = (state, action) => {
  switch (action.type) {
    case "FETCH_POSTS_INIT": {
      return { ...state, isFetchingPosts: true };
    }

    case "FETCH_POSTS_FAILURE": {
      return { ...state, isFetchingPosts: false };
    }

    case "FETCH_POSTS_SUCCESS": {
      return { ...state, isFetchingPosts: false, posts: action.payload };
    }
  }
};
```

---

#### Logic - Component

```typescript
const Component = (/* some props */) => {
  const [state, dispatch] = useReducer(reducer, { isFetchingPosts: false, posts: [] });

  const fetchPosts = async () => {
    dispatch({ type: 'FETCH_POSTS_INIT' });
    // Actually fetch data, building a `response` value
    if (!response.ok) {
      return dispatch({ type: 'FETCH_POSTS_FAILURE', payload: new Error('...') });
    }

    return dispatch({ type: 'FETCH_POSTS_SUCCESS', payload: await response.json() });
  };

  useEffect(() => {
    fetchPosts();
  }, []);

  return (
    // UI
  );
};
```

---

class: middle, center

## What if I want to fetch an other resource?

---

class: middle, center

### I will need to copy paste most of the logic...

---

class: middle, center

### Let's try to split the View and the Hook

---

### Example (with hook)

We will now try to implement the same logic using Hooks

In `/src/hooks/useFetch.ts`:

#### Types (everything is now dynamic)

```typescript
import { Reducer, useEffect, useReducer } from "react";

interface State<T> {
  data: T | null;
  isFetching: boolean;
}

type Actions<T> =
  | { type: "FETCH_INIT" }
  | { type: "FETCH_FAILURE"; payload: Error }
  | { type: "FETCH_SUCCESS"; payload: T };
```

---

#### Logic - reducer

```typescript
const reducer: Reducer<State<any>, Actions<any>> = (state, action) => {
  switch (action.type) {
    case "FETCH_INIT": {
      return { ...state, isFetching: true };
    }

    case "FETCH_FAILURE": {
      return { ...state, isFetching: false };
    }

    case "FETCH_SUCCESS": {
      return { ...state, isFetching: false, data: action.payload };
    }
  }
};
```

---

### Logic - Hook body

```typescript
// The return type has to be explicitly a Tuple
export type UseFetchReturnType<T> = [State<T>, Dispatch<Actions<T>>];

export function useFetch<T>(url: string): UseFetchReturnType<T> {
  const [state, dispatch] = useReducer<State<T>, Actions<T>>(reducer, {
    isFetching: false,
    data: null
  });

  const fetchData = async () => {
    dispatch({ type: "FETCH_INIT" });
    // Fetch data...
    if (!response.ok) {
      return dispatch({ type: "FETCH_FAILURE", payload: new Error("...") });
    }
    return dispatch({ type: "FETCH_SUCCESS", payload: await response.json() });
  };

  useEffect(() => {
    fetchData();
  }, []);

  // We can also only return the state, to avoid the user to accidentaly change the state
  return [state, dispatch];
}
```

---

#### Logic - Component

In `src/components/Posts.tsx`:

```typescript
// Some types related to Post

const Component = (/* some props */) => {
  const [state, dispatch] = useFetch<Post[]>(
    "https://jsonplaceholder.typicode.com/posts"
  );

  return; // UI
};
```

In `src/components/Users.tsx`:

```typescript
// Some types related to User

const Component = (/* some props */) => {
  const [state, dispatch] = useFetch<User[]>(
    "https://jsonplaceholder.typicode.com/users"
  );

  return; // UI
};
```

---

class: middle, center

### What about the components?

Two kinds of components:

- **ContextfulComponent**: those using `Context`, they are equivalent to the `StatefulComponent` we have without Redux, or the `ConnectedComponent` with Redux.
- **SimpleComponent**: the simple "dumb" Components. They will just use `ContextfulComponents` and other "dumb" components.

---

class: middle, center

### ContextfulComponents

- Will more likely use one or several Hooks
- Expose the `Context`, a (possibly) customized `Provider` and a (possibly) customized `Consumer`
- May be "Abstract". If they are, they will just wrap some hook(s) to ease their use

---

### Hooks limitation

In our previous example, we needed to fetch some posts, and some users dynamically. We had a `/posts` route and a `/users` route.

Now, what if we needed some data when initializing the application? Like the connected user data for instance?

If we use the `useFetch` hook as is at the top level it may become tedious to share the user data with the children, like this:

```typescript
// Toplevel component
const App = (/* some props */) => {
  // `state.data` will be null then the current user
  const [state, dispatch] = useFetch("https://www.myapi/current-user");
  // ...

  return (
    // ...
    <Layout
      user={state.data} /* <- We need to pass down the user object here */
    />
    // ...
  );
};
```

---

class: middle, center

### A solution: The "Abstract Contextful Component"

In order to solve this kind of problems, as we already said, we need a context.

Then we can use, or the context's consumer, or the `useContext` hook.

---

### Here comes the Abstract Contextful Component:

```typescript
import React, { createContext } from "react";

import { useFetch } from "../hooks/useFetch";

// Some types related to User

// We know the Value passed to the Context will be exactly the value returned by the Hook
type Value = UseFetchReturnType<User>;

// We export the Context so it can be used with `useContext`
export const Context = createContext<Value>(null as any);

// The props are similar to the hook arguments
interface Props {
  children?: React.ReactNode;
  url: string;
}

// The consumer does not need any customization here
export const Consumer = Context.Consumer;

export const Provider = ({ children, url }: Props) => {
  const value: Value = useFetch(url);

  return <Context.Provider value={value}>{children}</Context.Provider>;
};
```

---

### Example - Abstract Contextful Component - App

```typescript
// Toplevel component
const App = (/* some props */) => {
  // ...
  return (
    // ...
    <Fetch.Provider url="https://www.myapi/current-user">
      <Layout
      /* No need to pass props anymore */
      />
    </Fetch.Provider>
  );
};
```

---

### Example - Abstract Contextful Component - Use with Consumer

```typescript
const DeeplyNestedComponent = (/* some props */) => {
  // `state.data` being null or the current user here
  return <Fetch.Consumer>{([state]) => state.data}</Fetch.Consumer>;
};
```

---

### Example - Abstract Contextful Component - Use with useContext

```typescript
const DeeplyNestedComponent = (/* some props */) => {
  const [state] = useContext(Fetch.Context);

  // `state.data` being null or the current user here
  return <>{state.data}</>;
};
```

---

class: middle, center

### SimpleComponents

- May use some hooks (`useContext` for instance)
- No context, no state
- As small as possible (if when writing a rather big component you need at some point to `useContext` or some `Consumer`, you better create a new component)
- Modular, UI oriented

---

### Folders organization (example)

```
  /src
    /hooks
    /contextfulComponents
    /simpleComponents
```

---
