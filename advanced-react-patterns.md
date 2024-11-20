1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. Advanced React Patterns
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# Advanced React Patterns

## Context Module Functions

The Context Module Functions Pattern allows you to encapsulate a complex set of
state changes into a utility function which can be tree-shaken and lazily
loaded.

## Compound Components

You can iterate with `React.Children.map()` and use
`React.cloneElement(child, {myProps})` to pass props to from the parent to the
children (in compound components)

## Flexible Compound Components

Flexible Compound Components have a similar API to Compound Components, but they
allow to have the subcomponents nested inside DOM components.

In this case, you cannot iterate the children and clone them to implicitly pass
the props from the parent component to the subcomponents, because you don't know
how nested they will be. Instead, you can use a Context provider in the parent
so all the children have access to what they need.

In the React.Components devtool, the context providers will appear as
`Context.Provider` if you want better indication you can add `.displayName` to
the const you create with `React.createContext()`. However, if you wrapp it in
your own `MyExampleProvider` component, that one will show, so maybe it's not
needed...

## Prop Collections and Getters

Your custom hook can have prop collections in an object that need to always be
passed to the component that uses it. It's a way of re-using.

Even better use a prop getter, so you can compose an object of default props +
passed by the user props. A prop getter is a function that gets an object of
props a parameters and returns an object with the default + passed as params
composed, be careful with overrides, for functions maybe you want to call the
default AND the passed as parameter.

## State Reducer

This pattern allows you to introduce inversion of control, so instead of adding
more and more props and use cases, you give control to the user of your API. In
hooks it goes like this:

1. End user does an action
2. Dev calls dispatch
3. Hook determines the necessary changes
4. Hook calls dev's code for further changes 👈 this is the inversion of control
   part
5. Hook makes the state changes

In a nutshell, we use `useReducer` in our hook, and we have a default reducer,
but we allow the user of our hook to pass their own reducer if they want to
change the behaviour so it matches their use case.

This makes our hook WAY more flexible, but it also means that the way we update
state is now part of the API and if we make changes to how that happens, then it
could be a breaking change for users. It's totally worth the trade-off for
complex hooks/components, but it's just good to keep that in mind.

Remember to export the default reducer, so if the user implements a custom one,
they can implement only the parts that change, and call our default reducer in
any other case.

---

Object defaults in functions. If you have this signature

```javascript
myFunc({initialOn = false, reducer = toggleReducer} = {})
```

And you call it with nothing

```javascript
myFunc()
```

you are calling it with

```javascript
myFunc({initialOn: false, reducer: toggleReducer})
```

not with `myFunc({})` 🤯

## Control Props

The Control Props pattern allows users to completely control state values within
your component. This differs from the state reducer pattern in the fact that you
can not only change the state changes based on actions dispatched but you also
can trigger state changes from outside the component or hook as well.

You normally need:

- to allow to pass a props to control the state
- to allow to pass a callback that will be called with suggestions to change the
  state (when would the component normally update the state if no-controll prop
  used?)

It might be a good idea, to keep it flexible, to allow not to pass a control
prop for the use cases where is not necessary.

You should add warnings so if you pass a control prop, you must pass an onChange
callback. You should also add warnings in case your code changes from
uncontrolled to controlled and viceversa. These 3 warnings happens for example
in the DOM component `<input>`

Remember you can use `process.env.NODE_ENV === 'production'` to not log warnings
in production. If you use it to call hooks conditionally, you are violating the
rule of not use hooks conditionally, but that's ok in this case because
`process.env.NODE_ENV` will never change so you will run the same hooks on every
render.
