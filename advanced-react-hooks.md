1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. Advanced React Hooks
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# Advanced React Hooks

## useReducer: simple Counter

The main purpose of `useReducer` (or even Redux) is to separate the state logic
from the components that make the state changes.

Also, it's a good idea to use `useReducer` instead of `useState` when in the
updateds (reducers) a piece of the state will depend on another piece of the
state.

You create reducers with a _reducer function_ and _initial value_ Then
`useReducer` returns the _current value_ and the _dispatch function_, which you
can call with an _action_.

Reminder: you can call `setState` with a function.

Same like with `useState` you can do lazy initialization with `useReducer`, but
it has 3 parameters instead: if you pass a 3rd (the initialization function, it
passes the 2nd parameter to the function).

## useCallback: custom hooks

`useCallback` is like `useMemo` but for a function, it makes the code a tiny bit
simply, but we could use `useMemo` all the time.

`useCallback` could give you a different function even if the dependencies
didn't change... althought it actually doesn't happen it could happen in the
future, (something to do with allowing garbage collection to destroy the
function if the component is no longer on the screen)

Using `useCallback` and `useMemo` comes with a cost (_performance_, and code
readability), so it's important to make sure you have a good reason to use them
(and measure performance). Here are 2 main reasons

- Referential equality: make sure the function is not created again if used in
  dependencies, so if it doesn't change is understood as not changing in the
  dependencies array.
- Computationally expensive calculations

`useEffect` with no dependency array runs on every render, but the difference
between that and just writing the code without `useEffect` is that `useEffect`
runs _after_ the render, whereas the code you would write outsid runs _before_
the render.

## useContext: simple Counter

Context is better suited for libraries than for your application, it's better to
resort to composition first.

What does it mean composition here? https://www.youtube.com/watch?v=3XaXKiXtNjw
Not using black-box components, have the freedom to choose which subcomponents a
parent component will render Instead of

```javascript
<Dashboard user={user} />
{/* Dashboard renders DashboardNav and DashboardContent, which renders Dashboard Message*/}
```

Do

```javascript
<Dashboard>
  <DashboardNav />
  <DashboardContent>
    <WelcomeMessage user={user}>
  </DashboardContent>
</Dashboard>
```

so you don't need Context to avoid prop drilling (in the 2nd example `user` is
not drilled, in the first example is drilled 2 times until it reaches
`WelcomeMessage`)

---

Most of the time, it's not recommended to use a default value because it's
probably a mistake to try and use context outside a provider.

Don't use `<MyExampleContext.Provider>`, create a wrapper component
`<MyExampleProvider>` which returns `<MyExampleContext.Provider>`, so you can
have state, useEffects, etc. on it.

You can also create your own hook `useMyExample()` instead of using
`React.useContext(MyExampleContext)`. Among other stuff, on it you can check
that the `value` is defined (component using `useMyExample` is wrapped in
`MyExampleProvider`) and throw an error if it's not. **React will tell us in
which components the error happened**

## useLayoutEffect: auto-scrolling textarea

The order is: Render (return JSX) -> Dom is modified -> Layout Effects -> Screen
Painted -> Effects

`useEffect` and `useLayoutEffect` have them same API

If your effect is mutating the DOM (via a DOM node ref) and the DOM mutation
will change the appearance of the DOM node between the time that it is rendered
(return JSX) and your effect mutates it, then you don't want to use `useEffect`.
You'll want to use `useLayoutEffect`. Otherwise the user could see a flicker
when your DOM mutations take effect. This is pretty much the only time you want
to avoid `useEffect` and use `useLayoutEffect` instead.

You also want to use `useLayoutEffect` if you are updating a value (like a
`ref`) and you want to make sure it's up-to-date before any other code runs
(like another `useLayoutEffect` reading from that ref)

Another way to put it: use `useLayoutEffect` everytime that the effect modifies
the DOM in a way that's observable to the users.

## useImperativeHandle: scroll to top/bottom

`useImperativeHandle` allows us to expose imperative methods to developers who
pass a ref prop to our component which can be useful when you have something
that needs to happen and is hard to deal with declaratively.

NOTE: most of the time you should not need useImperativeHandle. Before you reach
for it, really ask yourself whether there's ANY other way to accomplish what
you're trying to do. Imperative code can sometimes be really hard to follow and
it's much better to make your APIs declarative if possible.

## useDebugValue: useMedia

Note: useDebugValue values will not show in production, because the production
build of useDebugValue does nothing.

useDebugValue changes what is showed in React.Components devtool, when seeing
the hooks used by a component. If a hook is used multiple times it can help to
identify which one is which. It will not work on components, only in custom
hooks.

You can pass a second parameter, a formatter function, then the first parameter
would be an object with the values you are interested in login, which is what is
passed to that formatter function. This is only really useful for situations
where computing the debug value is computationally expensive (and therefore you
only want it calculated when the DevTools are open)
