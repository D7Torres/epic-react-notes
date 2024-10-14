1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. React Suspense
8. [Build an Epic React App](./build-an-epic-react-app.md)


# React Suspense

## Simple Data-fetching

NOTE: In the latest version of React, “Concurrent Mode” is not a thing. Instead
there are “Concurrent Features” that your components opt-into via the concurrent
APIs which you will learn about in this workshop. Learn more about some of the
changes in the notice at the top of this docs page:
https://react.dev/blog/2022/03/29/react-v18#gradually-adopting-concurrent-features

> The new rendering behavior in React 18 is only enabled in the parts of your
> app that use new features.

“Render as you fetch”: kicking off the request for the data as soon as you have
the information you need for the request.

Interesting `createResource` refactor, check the code.

## Render as you fetch

3 different ways we fetch in React

- Fetch on Render 👎: useEffect, fetch on it. (in the example,
  `<React.Suspense>` is only used for the lazy loaded component, not for the
  fetch). Waterfall effect: 1. Load the code. 2. Make the fetch. 3. Load the
  image the fetch data points to.
- Fetch then render 👎: still useEffect and lazy loading the component, but the
  fetch happens in the parent (so it happens before the component loads). Fetch
  and component code could be loaded in parallel, but you probably don't render
  the component unless you have the data (unless using placeholders) so you
  could still have the waterfall.
- Render as you fetch 👍: `createPokemonInfoResource` (responsible for loading
  data) lazy loaded, component also lazy loaded. No waterfall.

React.Suspense will also render the fallback if the children components are
being lazy-loaded

## useTransition for improved loading states

Use `useTransition` to not show the fallback, for a few moments, instead you can
show something else in the meanwhile, like blur or opacity. Then if it's still
taking to long show the fallback eventually.

Consider using CSS `transition-delay: 0.3s;` so fast connection don't see a
flickering blurr. Slow connection will have a 0.3s daley befor the blurr is
showed, though.

But what if the request takes 350ms, we are back to the same problem. The
solution here is no longer available because the `SUSPENSE_CONFIG` object is not
used anymore, I would have to research or test. Maybe react takes care of this
case under the hood now, hence not `SUSPENSE_CONFIG` needed as React has the
best defaults.

## Cache resources

Remember render can be called more than once (specially in development).

One of the points of React Query is to abstract catching.

Use `useRef()` when you want a variable to have something consistent between
renders instead of re-creating it each render.

## Suspense Image

You can preload images in JS, with the promise way below

```javascript
function preloadImage(src) {
  return new Promise(resolve => {
    const img = document.createElement('img')
    img.src = src
    img.onload = () => resolve(src)
  })
}
```

Or you can preload them in HTML like this:

```html
<link
  rel="preload"
  href="%PUBLIC_URL%/img/pokemon/fallback-pokemon.jpg"
  as="image"
/>
```

It looks like I'm better off using Kent's abstraction to suspend stuff
(`createResource()`), check examples (this exercis, for instance)

If the images URL is easy to guess (follows a pattern) try to use that rather
than get the url from an api call and then having to make another call for the
image.

## Suspense with a custom hook

With this hook

```javascript
const [pokemonResource, isPending] = usePokemonResource(pokemonName)
```

suspense is even more abstracted.

## Coordinate Suspending components with SuspenseList

`<React.SuspenseList>` is about loading stuff not as soon as you can but in a
certain order that is more predictable so it's not a jarring experience for the
user. It's experimental, didn't make it to React 18 yet (`<React.Suspense>`
did).

`revealOrder` "together", but "forward" and "backwards" didn't, as they need the
`<React.Suspense>`s to be direct children of `<React.SuspenseList>`, same for
some of the values of `tail`.

Remember you can use nested `<React.SuspenseList>`!!
