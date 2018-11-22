<!-- $width: 1500 -->
<!-- $height: 1500 -->

## Context
  - render props
    - provide a simple/real life example (w/o Redux)
  - What is React's context?
    - always there
    - hidden behind a undocumented API
    - used by Redux, Redux-Form, etc...
      - we can solve with context what we can solve with Redux
  - `Provider` / `Consumer`
    - define these two concepts
  - `createContext`
    - give a simple example (live coding?)
  - `defaultValue` is _not_ `initialValue`

---

## Hooks
  - `useEffect`
    - basic usage
    - inputs? (the array)
    - alternative to the lifecycle
    - how can I replace `componentDidMount`, `componentWillUpdate`, or `componentWillUnmount`
      - examples
    - no async code inside
    - only use to actually "trigger" an async stuff
  - alternatives `useMutationEffect` and `useLayoutEffect`
  - `useState` + `useReducer`
    - example
  - creating a custom hook
    - rules
    - real life example useFetch or localStorage

---

## A new architecture
  - move the logic into custom hooks
    - the logic _may_ be in the component but it _could_ lower the modularity
  - two kinds of components
    - Context
      - may use hooks, exposing a provider and a consumer
      - Abstract context
        - wraps a hook, so that it can be used with the React Context
    - Component
      - may use hooks, and contexts components
      - should be as small as possible
        - need for `useContext` or `Consumer`? Better create a new component
        - keep it small, modular and simple

---

## Cons so far
  - experimental
  - limitations
    - pyramid of doom with providers...
      - epitath
    - ...and consumers
      - `useContext` (Context + Hooks)
        - example
  - edge cases
    - mapping over provider/consumer at _two_ different places in the same components
      - example
      - will require the use of an "index" system
        - example
    - fetching the same data for several use cases
      - example (prefectures)
      - solution 1 - fetch in the provider, put the provider at the top level
      - solution 2 - cache
        - `useFetchPrefectures` cannot help (it's an "instance")
        - apollo, or fetch has to be cached
        - bottleneck schema
