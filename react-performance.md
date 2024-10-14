1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. React Performance
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. [Build an Epic React App](./build-an-epic-react-app.md)

# React Performance

## Code splitting

Really low hanging fruit. Great impact. Lazily load your code at the time that
is needed.

You can lazily import a module with native JS now

```javascript
import('/some-module.js').then(
  module => {
    // do stuff with the module's exports
  },
  error => {
    // there was some error loading the module...
  },
)
```

You can also do it with React, but the module must default export a component,
and you have to use `<React.Suspense fallback={...}/> to render a fallback while
it's loading.

"Coverage" Dev Tool can help to find the need/benefit of code splitting for a
certain feature/page/interaction. Check the bottom line, where it says the total
MB loaded, and how much was run. It's more important that less amount in total
is run, that it is that the % run is higher. You can click on individual files
and it will show you the source code and which lines haver not be run (so maybe
you can optimise by more code splitting)

`Eager loading` You can preemptively load the component before actually needed
(when you get an indication the user is about to interact), so the user doesn't
need to wait so long. e.g.: `onMouseOver`, `onFocus` can import the module as
well, as it doesn't matter how many times is imported, webpack knows to import
it only once.

You can use `import(/* webpackPrefetch: true */ './some-module.js')` for the
browser to load the module in the cache ahead of time (it's like `eager loading`
but as soon as the page loads), so when it's time to import it it's already
there.

It'll translate to

```html
<link rel="prefetch" as="script" href="/static/js/1.chunk.js" />
```

Another thing which we won't cover in this workshop, but you should look into
later, is using the webpackChunkName magic comment which will allow webpack to
place common modules in the same chunk. This is good for components which you
want loaded together in the same chunk (to reduce multiple requests for multiple
modules which will likely be needed together).

In DevTool React.Components you can select any component and click the clock to
suspend it and see how the UI would look with the fallback.

Apply Suspense in a similary way to ErrorBoundaries, do not just wrap every
component that needs suspense as an immediate parent.

## useMemo for expensive calculations

Dev Tools > Performance. Use a 6x slow CPU. Record, and then check the Flame
chart below. Scroll with and without Shift.

Above `Main` you have `Timings` which is stuff executed by React, like a
component update.

Find where `App` is in the Flame chart, to know what takes long of your code.

If you want to have a 60fps experience, you need to nail your re-renders to
16ms. But profile it in prod, as dev has a lot of extra stuff running.

`Timings` is only in dev, not in prod.

Web Workers are like threads. JS is single threaded, but you can spin up
multiple JS threads in the browser. They can communicate between them with some
limitations. You can see the different workers in DevTools > Sources > Threads

## React.memo for reducing unnecessary re-renders

You can check for Unnecessary re-renders in DevTools > React.Profiler

A React Component can re-render for any of the following reasons:

1. Its props change
2. Its internal state changes
3. It is consuming context values which have changed
4. Its parent re-renders

You can opt-out state updates using these:

- React.PureComponent (class components)
- React.memo (function components)
- shouldComponentUpdate

do now confuse `React.memo(MyComponent)` (will not re-render if props have not
changed even if the parent re-rendered) with `React.useMemo()`

But they should not be used as band-aids over more problematic performance
problems:

> Fix the slow render before you fix the re-render

`React.memo` accepts a second argument which is a custom compare function that
allows us to compare the props and return true if rendering the component again
is unnecessary and false if it is necessary.

Trick so the above is not necessary: move your logic up so you pass only props
to a component that if they change, the DOM changes.

## Window large lists with react-virtual

Windowing: rendering only what appears in the window (or it's close to its
borders). You can achive that with libraries like `react-virtual`

Beware of the trick of having an empty `<li>` with the height of the whole
rendered list, so the scrolling is smooth and not buggy.

## Optimize context value

Context will rerender all consuming components when it changes the value,
**whether or not they are memoized**.

So, make sure the value does not change if it doesn't have to (as it can be an
array, or an object, make sure the same one it's returned if the values inside
did not change. You can achieve this by memoizing the value)

When a certain combinations of the following happens, then it's worth it to
apply the above optimisation:

1. Your context value changes frequently
2. Your context has many consumers
3. You are bothering to use React.memo (because things are legit slow)
4. You've actually measured things and you know it's slow and needs to be
   optimized

Alternative solution, even better, but consider the code complexity. To have 2
`useMyContext()`, one for consuming the value, and one for the update function.
That way, the components that only update the value don't need to re-render when
the value changes.

## Fix "perf death by a thousand cuts"

Sometimes, it looks like you cannot optimise more. Everything individually is
fast, but the app is still slow, because for example: a small change in the
state renders everything.

This can happen when the state has too much, it's too global. Collocating the
pieces of state to where it is actually needed can help with performance and
make your code easier to maintain.

You can also optimise by splitting different parts of the domain into different
contexts, so if the one that changes it's not the one I consume, I don't
re-render.

You can also optimise by splitting a component into 2 components: context state
getting + rest of the work, the first one renders the second one. That way if a
part of the context changes, but it doesn't change the render result, the first
component re-renders, but the child doesn't. **To take advantage of this
pattern**, just write a HOC: `withStateSlice` and reuse it.

Note: apply a displayName:

```javascript
withStateSlice(${Component.displayName || Component.name})
```

to the Wrapper your HOC returns (before memoizing) for it to appear nicely as a
tag in the DevTools, it's a convention to use the `with..`.

Check the course and add a forward ref if you are serious about using it for
your project.

State management in a re-render performance way: Recoil (consider whether you
need it)

## Production performance monitoring

Use `<React.Profiler onRender={myMethod}>` to have your method called with
useful data (time) on every render of any child component. Re-renders are very
frequent so you should batch the re-renders info and communicate with your
monitoring (e.g. graphana) service only every few seconds or minutes.

The performance impact is minimum, but FB only delivers a version with profiler
enabled (you need to enable it in the build) to a subset of the users to not
impact everyone.

With this, you can detect if a render goes up after a release.
