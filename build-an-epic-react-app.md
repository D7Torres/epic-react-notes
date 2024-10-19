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



## 6. Cache management



## 7. Context



## 8. Compound components



## 9. Performance



## 10. Render as you fetch



## 11. Unit testing



## 12. Testing hooks and components



## 13. Integration testing



## 14. E2e testing
