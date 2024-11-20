1. [React Fundamentals](./react-fundamentals.md)
2. React Hooks
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# React Hooks

## useState: greeting

There's a `useReducer` hook.

We need to use state because changing variables won't trigger a re-render.

Reminder: A "controlled" field / component / element..., is one which is
controlled by React (parent component) instead of by the browser/user.

## useEffect: persistent state

`useEffect` runs after the component renders (or re-renders), to run some custom
code, that interacts with the outside of the component. Like an HTTP request or
interacting with Local Storage. It runs after the DOM has been updated.

We can call what it does a "side effect".

`useState` will actually run whatever you pass as initial value on each
rerender!! But it only needs it on the first one. So, if it's an expensive
opperation, instead you should pass a function to it, instead of a value. And it
will use the value returned by the function, which will only run on first
render.

You can create custom Hooks for any logic you think it can be reused on multiple
components. A custom Hook is a function which uses other hooks. By convention,
they always start with `use...`

## Lifting state

Sharing state by placing it in the closest common ancestor is called "lifting
state up", You then pass the state down as props, and also pass down the
mechanism to update it.

Moving the state down, when no longer is needed shared is called "state
colocation"

## useState: tic tac toe

We called "derived state" to values we calculate from the state (but they are
not in `useState`), just normal variables after.

I did my own useLocalStorageState hook to practice.

It's a good idea to set up the UI before you get state management and
interactivity.

Derive state when you can, don't have 2 pieces of state if one can be derived
from the other.

## useRef and useEffect: DOM interaction

Reminder React function components don't create DOM elements until they render,
so we don't have access to them just with JSX.

The way you ask React to give you access to DOM elements is by using the prop
`ref`.

You pass `myRef` (`cons myRef = React.useRef()`) as a `ref` prop of an element,
and React will put the DOM element on `myRef.current`.

`useEffect` runs after the component has rendered, so you have access to it by
then. So you'll usually use the DOM inside `useEffect`.

Think about `useEffect` as synchronizing the state of the world with the state
of the app.

## useEffect: HTTP requests

You can do error handling in `then()` in a different way than with `catch`

```javascript
// option 1: using .catch
fetchPokemon(pokemonName)
  .then(pokemon => setPokemon(pokemon))
  .catch(error => setError(error))

// option 2: using the second argument to .then
fetchPokemon(pokemonName).then(
  pokemon => setPokemon(pokemon),
  error => setError(error),
)
```

But it's important to remember that they are not the same, `option 2` will catch
error happening in `fetchPokemon` but not errors happening in `setPokemon`.

React normally batches setStates, but not if they are in async callbacks
(`.then()`). Actually, since React 18, it does batch them too. However, it's
better to keep related state together in an object.

When an error is thrown and it's not handled, your application will be removed
from the screen, leaving the user with a blank page. That's why we use
ErrorBoundary components.

To implement an ErrorBoundary you must provide
`static getDerivedStateFromError`, which lets you update the state in response
to an error. Optionally, you can implement `componentDidCatch` to add some extra
logic like logging the error to an analytics service.

Different ways to recover from errors, like

- `resetErrorBoundary` function passed to your fallback component (e.g.: for a
  Retry button)
- `onReset` prop in `ErrorBoundary` to manage your parent component state
- `resetKeys` for automatically resetting when a dependency changes

To manage errors that cannot get managed by `ErrorBoundary` component,
`react-error-boundary` exposes a hook:

```javascript
const { showBoundary } = useErrorBoundary()

...

showBoudnary(error)
```

Alternatively, you can just throw and have the component where you are throwing
wrapped by an `ErrorBoundary`
