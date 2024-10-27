1. [React Fundamentals](./react-fundamentals.md)
2. [React Hooks](./react-hooks.md)
3. [Advanced React Hooks](./advanced-react-hooks.md)
4. [Advanced React Patterns](./advanced-react-patterns.md)
5. [React Performance](./react-performance.md)
6. [Testing React Apps](./testing-react-apps.md)
7. [React Suspense](./react-suspense.md)
8. Build an Epic React App

# Build an Epic React App

## 1. Bootstrap

```javascript
import * as React from 'react'
import {createRoot} from 'react-dom/client'
...
const rootElement = document.getElementById('root')
const root = createRoot(rootElement)
root.render(<App />)
```

(React.createRoot did not work)

Use `aria-label` in Dialogs to describe the content, for a11y purposes.

## 2. Styles

With Emotion css, this is needed
```javascript
/** @jsx jsx */
import {jsx} from '@emotion/core'
```

so the `css` prop can work. Instead of transpiling something like `<div css={{...}}>` to `React.createElement('div')`, it will transpile it to `jsx('div')` which will do `React.createElement('div')` but will also take care of the `css` prop.

Example of styled component with variants:
```javascript
const buttonVariants = {
  primary: {
    color: 'white',
    background: '#3f51b5',
  },
  secondary: {
    color: '#434449',
    background: '#f1f2f7',
  },
}

const Button = styled.button(
  {
    padding: '10px 15px',
    border: '0',
    lineHeight: '1',
    borderRadius: '3px',
  },
  ({variant = 'primary'}) => buttonVariants[variant],
)
```

To style a regular component instead of an element
```javascript
const StyledSpinner = styled(Spinner)({/* styles here */})
```

The `css` prop will generate a class with the component name, great for debugging, but the styled component will not. To have it for the styled component, you need to import from `styled/macro`

```javascript
import styled from '@emotion/styled/macro'
```

This will also make the components have a name without displayName set, in the React.Component devtools tab.

It's a good idea to give a spinner an `aria-label="loading"`, even to give it a default prop 
```javascript
Spinner.defaultProps = {
  'aria-label': 'loading'
}
```

## 3. Data fetching

Copy the hook `useAsync` in this exercise to do anything async, so you don't have to manage status, data, and error states.

## 4. Authentication

I found this configuration mixing example interesting. When you are mixing an object with defaults and overrides, remember that if you fear some override, you can always destructure that part (see `headers`) and have the rest with `...`. Also, if a field is there only under certain circumstances, don't be afraid of using `undefined`

```javascript
function client(endpoint, {token, headers, ...customConfig} = {}) {
  const config = {
    method: 'GET',
    headers: {
      Authorization: token ? `Bearer ${token}` : undefined,
      ...headers,
    },
    ...customConfig,
  }
```

## 5. Routing

With React Router, you can change the URL without reloading the browser. You can react to URL changes and render what is relevant.

Example
```javascript
  <BrowserRouter>
    <Routes>
      <Route path="/discover" element={<DiscoverBooksScreen user={user} />} />
      <Route path="/book/:bookId" element={<BookScreen user={user} />} />
      <Route path="*" element={<NotFoundScreen />} />
    </Routes>
  </BrowserRouter>
```
Redirect:
Often, developers will use their client-side router to redirect users, but it's not possible for the
browser and search engines to get the proper status codes for redirect (301
or 302) so that's *not optimal*.

To *configure it in the server side*, you'll probably have to do different things for local dev (e.g. CreateReactApp), local build (e.g. serve npm module, which has a `serve.json` config) and prod (e.g. Netlify config file `_redirects`).

React Router `useMatch` to know if the current URL matches something (e.g. to highlight the current nav item)

## 6. Cache management

An design error is to try to have the application state (e.g. UI things are open/close) and server cache state (e.g. data obtained from the server) on the same store. This tends to remove colocation, and makes things more difficult to maintain.

So, if we are going to separate the server data, a common way is with React Query. You use `useQuery` for GETs and `useMutation` for the rest. React Query has sensible defaults for caching, retrying, etc. and a lot of flexibility.

- Remember to invalidate cache in mutations if it suits your case. You can modify the cache yourself to avoid a re-fetch, but that can be bug-prone than to re-fetch and trust the source of truth (the  server)
- In the RQ config, you can modify it to do a different number of retries. Instead of a number, you can pass a function where you can do things like: this number or retries, but if it's a 404, no retries. I'm not pasting the code because in the new version of RQ the api is very different so I'll have to research.


## 7. Context



## 8. Compound components



## 9. Performance



## 10. Render as you fetch



## 11. Unit testing



## 12. Testing hooks and components



## 13. Integration testing



## 14. E2e testing
