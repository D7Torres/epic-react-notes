1. React Fundamentals
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# React Fundamentals

## Basic JavaScript-rendered Hello World

React is used to manipulate the DOM, here we use what React uses to do it
ourselves (the DOM api), so we know what React does under the hood.

## Intro to raw React APIs

I learnt about why React has `React` and `ReactDOM`: Basically, React can render
in other places, not only in the Browser (VR, native...), so the render part
(`ReactDOM`) which interacts with the specific platform rendering API is
separated, so the non-render part can be reused (`React`).

Here we use the raw ReactAPI, so we now know one level of abstraction. Of course
in the future we'll do it declaratively with JSX, but this is what React will be
doing behind the scene.

## Using JSX

Babel compiles JSX to things like `React.createElement('div', { ... })`

"Interpolation" is defined as "the insertion of something of a different nature
into something else"

Reminder of prop spread like `{...props}` in JSX. You can override whatever is
in `props`, not only when defining the object, but also as extra attributes
after putting `{...props}` in the JSX.

Expression vs Statement. Expression evaluates to a value Statement are
imperative logic

## Creating custom components

A React component is very similar to just a function that returns something
renderable. It's capitalised so Babel can correctly transform it into a call to
the function instead of wrapped by quotes when passed to `React.createElement`

I learned how PropTypes works inside, by implementint it. As it's just extra
code doing checkings and this has an impact in performance, it doesn't render in
production.

Reminder of what Fragments do, and that you can also use them like
`<React.Fragment>`

You can define your own functions in propTypes for a very specific type or very
specific validation. (Imagine: you can only pass a number that is odd an
divisible by 3)

## Styling

The reason why the css attributes in JSX are written in camelCase (and in an
object instead of a string), is so it maches the `style` property of DOM nodes.
Similarly, the DOM attribute is called `className`, so we use that instead of
`class` (in addition of `class` being a reserved word in JS)

`$0` to reference selected element in Chrome devtools 🤯

Importance of using defaults for variables that end up rendering, so they don't
render `undefined`

## Forms

`console.dir()` shows the object being logged in an interactive way so you can
explore it.

You should usually do `event.preventDefault()` in handleSubmit functions so the
page doesn't refresh due to the default POST to the same URL with the form input
as query params.

The main way to access values is `event.target.inputNameOrId.value`

In HTML we link labels to inputs with the `for` attribute, but in JSX is called
`htmlFor`. When they are linked (by input's `id`), clicking the label will give focus 
to the input.

A ref is an object that stays consistent between renders of your React
component. It has a current property on it which can be updated to any value at
any time.

By using a ref, you ensure that:

- You can store information between re-renders (unlike regular variables, which
  reset on every render).
- Changing it does not trigger a re-render (unlike state variables, which
  trigger a re-render).
- The information is local to each copy of your component (unlike the variables
  outside, which are shared).

`useRef` is a React Hook that lets you reference a value that’s not needed for
rendering.

You can pass your ref to a React element

```javascript
<MyComponent ref={myRef}>
```

React will store the DOM element on `myRef.current`

You should add `role="alert"` when displaying errors, for a11y.

Controlled Form Input === input which we stablish it's value as
`value={ourValue}`, so React controls the value, rather than the DOM controlling
the value.

## Rendering Arrays

When you don't give a `key` prop, React assumes the key is the index in the
array (but this is not great!).

Reminder, you can intentionally change the `key` of an element to reset the
state of a component (because it will be unmounted and mounted)
