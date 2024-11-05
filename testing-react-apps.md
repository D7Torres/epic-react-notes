1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. Testing React Apps
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# Testing React Apps

## simple test with ReactDOM

When I need to check things loosely (like to be called with an object with
certain structure, but not exact), check the jest -> expect docs page, check the
asymetric matchers.

In React v18, you're required to wrap all your interactions in
[`act`](https://reactjs.org/docs/test-utils.html#act), but RTL does it
automatically for you.

`.click()` (out of RTL) is not exactly what it happens in the browser, it's more
precise to use `dispatchEvent(event: Event)`

## simple test with React Testing Library

React needs changes to components wrapped in `act()`. RTL does it for you, but
only for any of your code that is running within the React callstack (like click
events where React calls into your event handler code which updates the
component), but it cannot handle this for any code running outside of it's own
callstack (like asynchronous code that runs as a result of a resolved promise
you are managing or if you're using jest fake timers). With those kinds of
situations you typically need to wrap that in act(...) or async act(...)
yourself. BUT, React Testing Library has async utilities that are wrapped in act
automatically!

If you're still experiencing the act warning, then the most likely reason is
something is happening after your test completes for which you should be waiting
(e.g. wait for something to appear or disappear)

You can also see the warning (and you have to manually add `act()`) when:

- When using `jest.useFakeTimers()`
- When testing custom hooks
  (`import { renderHook } from '@testing-library/react'`), when you call
  functions returned from your hooks which result in state updates
- When using `useImperativeHandle` (you only run into this if you're calling
  methods directly on a component which do internal state updates and you're
  outside of React's callstack)

```javascript
const {container} = render(<MyComponent />)
```

`container` is just the `div` RTL creates for you when you render and where it
renders your component nested on it.

Use [RTL custom jest matchers](https://github.com/testing-library/jest-dom)
instead of Jest ones for better error messages.

If you need to give a name to the object returned by `render()` use one of these

```javascript
const view = render(<MyComp />)
const utils = render(<MyComp />)
```


## Avoid implementation details

Check `within()` functionality, to not query the whole `document.body`

```javascript
const messages = document.getElementById('messages')
const helloMessage = within(messages).getByText('hello')
```

🚨🚨 Checkout the Dev Tools > Elements > Accessibility tab, where you can see the
role and name of elements.

[Testing-Playground](https://testing-playground.com) Check which RTL queries are best to
get elements in this sandbox. Even better, there's a Chrome extension.

Use RTL's `userEvent` instead of `fireEvent` to do what the user would do. For
example, if the user click, a lot of events will happen: mouseOver, mouseDown,
mouseUp, click, ... `userEvent` takes care to replicate all this when you do
`await userEvent.click()`. It returns promises so remember to use `await`.

## form testing

`screen.debug()` to check what the UI looks like after render

password inputs don't have a role for seccurity reasons, get them with
`getByLabelText()`

Use `jest.fn()` and `toHaveBeenCalledWith()`

Use `faker` to generate input which exact value is irrelevant to communicate
that is irrelevant to whoever reads the test.

Use `@jackfranklin/test-data-bot` to automate more data creation.

## mocking HTTP requests

Elaborate

Use MSW (Mock Service Worker) for that, intercepts instead of being a real
server, more convenient as you don't have to worry about ports and you can test
fetching from other domains.

`waitForElementToBeRemoved` RTL util

RTL `name` in getByRole can be an input label, the content of a button, or the
`aria-label` attribute of an element.

`toBeVisible` is like `toBeInTheDocument` with extra checks like

- display != none
- opacity greater than 0 etc.

`msw` was actually built for having a mock server during development, it's
convenient to have one so you can work with APIs that are not done yet. If you
do so, make sure you share your server handlers (the mocks) so you use the same
ones for development and testing.

Remember you can implement logic in your server mock, not only a response. So
you can return the normal response, but if some parameter is missing or it's a
certain value, then return an error, etc.

`.toMatchInlineSnapshot()` with no arguments will be auto-populated the first
time the test runs. Think whether you want to snapshot a whole div or only its
content.

Use `server.use` and `server.resetHandlers()` to add specific ones for a test
(like server unexpectedly failing) that would not make sense to have in the
handlers used for development.

## mocking Browser APIs and modules

Sometimes you have to use a polyfill or monkey patch some browser functionality
(e.g.: `window.resizeTo` and `window.matchMedia`), as what RTL uses instead of a
real browser (jsdom) doesn't support everything a browser does.

Mock modules: `jest.mock('./path')` to mock a module, all the exports are now
jest mock functions so you can `.mockImplementation()`,
`toHaveBeenCalledTimes()`, etc.

Use `requireActual()` to mock only parts of the module

```javascript
jest.mock('../math', () => {
  const actualMath = jest.requireActual('../math')
  return {
    ...actualMath,
    subtract: jest.fn(),
  }
})
```

You can use `jest.doMock` and `jest.dontMock` to opt-out of the hoisting
behaviour.

Useful util to resolve/reject a promis when you want

```javascript
function deferred() {
  let resolve, reject
  const promise = new Promise((res, rej) => {
    resolve = res
    reject = rej
  })
  return {promise, resolve, reject}
}
```

💰 Here's an example of how you use this:

```javascript
const {promise, resolve, reject} = deferred()
promise.then(() => {
  /* do something */
})
// do other setup stuff and assert on the pending state
resolve()
await promise
// assert on the resolved state
```

`act()` is needed when there is a state update that React was not expecting
(e.g. get location promise resolves, and the calls the callback which updates
the state). Wrapping that promis resolve in act ensures that React will flush
all the state changes so we are not left on a pending state when proceed with
the next assertions.

Careful defining the mockImplementations out of tests, as there are automatic
cleanings happening before each test.

Another way to test async stuff, apart of the `deferred` util above, when
mocking a hook, is to use something async inside like `useState` and asign the
`setX` method to a variable outside the mock implementation, that way the state
can be changed when desired to move from the pending state to the resolved one
and assert both states.

## testing with context and a custom render method

Use the `wrapper` option with the provider when using components which use
context, so you can re-render them without re-rendering the provider:

```javascript
function Wrapper({children}) {
  return <ContextProvider>{children}</ContextProvider>
}

const {rerender} = render(<ComponentToTest />, {wrapper: Wrapper})

rerender(<ComponentToTest newProp={true} />)
```

You can create a custom render function like we did in Atom Learning so you
don't have to use `wrapper` in each test.

To set it up, read
[this](https://testing-library.com/docs/react-testing-library/setup/)

To access your src files like they are modules (not using `../../myFile` etc.)
you can go to `jest.config.js` and in the option moduleDirectories add your
`src`

```javascript
moduleDirectories: ['node_modules', path.join(__dirname, 'src')]
```

## testing custom hooks

You shouldn't test custom hooks, you should test them via testing a component
which uses them.

However, if you are building a reusable hook, or you have a library of hooks or
something like that, it might be a good idea to test them. Still, your hook is
going to be used inside of a component, so just make a silly component that uses
it to test it.

When that's too complicated, make your silly component like this

```javascript
let result
function TestComponent(props) {
  result = useCustomHook(props)
  return null
}

// interact with and assert on results here
```

Even better, use `renderHook` from `@testing-library/react`, which is like the
above setup

In addition to `{result}` it returns a few more things

- Utility to "rerender" the component that's rendering the hook (to test effect
  dependency changes for example)
- Utility to "unmount" the component that's rendering the hook (to test effect
  cleanup functions for example)
- Several async utilities to wait an unspecified amount of time (to test async
  logic)

When I use `act` I indicate: I expect a change of state to happen, flush all the
changes so the component is stable, and then I continue asserting.

Remember `userEvent` utils wrap everything in `act` calls.

