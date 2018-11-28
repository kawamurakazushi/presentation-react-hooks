# This is some presentation notes.

### Reusing stateful logic between components - Problem

### Reusing stateful logic between components - Solution

### Complex components, with logic all over the lifecyle

We know that components and top-down data flow help us organize a large UI into small, independent, reusable pieces.

However, we often can’t break complex components down any further because the logic is stateful and can’t be extracted to a function or another component.

Sometimes that’s what people mean when they say React doesn’t let them “separate concerns.”

## Some Solution

Today, there are a lot of ways to reuse logic in React apps. We can write simple functions and call them to calculate something. We can also write components (which themselves could be functions or classes). Components are more powerful, but they have to render some UI. This makes them inconvenient for sharing non-visual logic. This is how we end up with complex patterns like render props and higher-order components. Wouldn’t React be simpler if there was just one common way to reuse code instead of so many?

**Functions** seem to be a perfect mechanism for code reuse.
It's one of the first things to do when you start programming,
and it's something you might forget doing properly when you get used to it.

Moving logic between functions takes the least amount of effort, and its very **simple**.

However, functions **can’t have local React state** inside them.

You can’t extract behavior like “watch window size and update the state” or “animate a value over time” from a class component without restructuring your code or introducing an abstraction like Observables.

Hooks solve exactly that problem. Hooks let you use React features (like state) from a function — by doing a single function call. React provides a few built-in Hooks exposing the “building blocks” of React: state, lifecycle, and context.
