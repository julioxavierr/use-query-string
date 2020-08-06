# useQueryString

[![Build Status](https://github.com/trevorblades/use-query-string/workflows/Node%20CI/badge.svg)](https://github.com/trevorblades/use-query-string/actions)

A React hook that serializes state into the URL query string

- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
  - [`parseOptions`](#parseoptions)
  - [`stringifyOptions`](#stringifyoptions)
- [Examples](#examples)
  - [Gatsby example](#gatsby-example)
  - [Practical example](#practical-example)
  - [An example using context](#an-example-using-context)
- [License](#license)

## Installation

```bash
$ npm install use-query-string
```

## Usage

Given a location object and a history updater function, this hook will return an array who's first element is an object representing the current URL query string. The second element in the array is a function that serializes an object into the query string and updates the former `query` object.

```js
import useQueryString from 'use-query-string';

const [query, setQuery] = useQueryString(location, updateQuery);
```

The first argument passed to the hook is a [`Location`](https://developer.mozilla.org/en-US/docs/Web/API/Location) object, and the second is a history-updating function with the following signature:

```ts
(path: string): void => {
  // update the browser history
}
```

## Configuration

### `parseOptions`

You can supply an optional third argument to the hook that gets passed along as options to the `parse` function. These allow you to do things like automatically convert values to numbers or booleans, when appropriate. See [the `query-string` docs](https://github.com/sindresorhus/query-string#parsestring-options) for all of the accepted options.

```js
const [query, setQuery] = useQueryString(
  location,
  navigate,
  {
    parseNumbers: true,
    parseBooleans: true
  }
);
```

### `stringifyOptions`

You can also pass a fourth argument to the hook that gets used as options for the `stringify` function that serializes your state. This is especially useful if you need to serialize/deserialize arrays a way other than the default. See [the `query-string` docs](https://github.com/sindresorhus/query-string#stringifyobject-options) for all of the accepted options.

```js
const arrayFormat = 'comma';
const [query, setQuery] = useQueryString(
  location,
  navigate,
  {arrayFormat},
  {
    skipNull: true,
    arrayFormat
  }
);
```

## Examples

In this example, you'll see a component using the query string to serialize some state about a selected color. The component uses the global [`Location`](https://developer.mozilla.org/en-US/docs/Web/API/Location) object, and a function that calls [`History.pushState`](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) to update the page URL.

```jsx
import React from 'react';
import useQueryString from 'use-query-string';

function updateQuery(path) {
  window.history.pushState(null, document.title, path);
}

function ColorPicker() {
  const [{color}, setQuery] = useQueryString(
    window.location,
    updateQuery
  );

  function handleColorChange(event) {
    setQuery({color: event.target.value});
  }

  return (
    <div>
      <p style={{color}}>Color is {color}</p>
      <select value={color} onChange={handleColorChange}>
        <option value="red">Red</option>
        <option value="blue">Blue</option>
      </select>
    </div>
  );
}
```

### Gatsby example

If you're using Gatsby, you could pass `props.location` and the `navigate` helper function from [Gatsby Link](https://www.gatsbyjs.org/docs/gatsby-link/) as arguments to the hook.

```js
// pages/index.js
import React from 'react';
import useQueryString from 'use-query-string';
import {navigate} from 'gatsby';

function Home(props) {
  const [query, setQuery] = useQueryString(
    props.location, // pages are given a location object via props
    navigate
  );

  // ...the rest of your page
}
```

### Practical example

The following CodeSandbox contains an example for working with multiple boolean filters that change something in the page and persist between reloads.

[![Edit zen-stallman-6r908](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/zen-stallman-6r908?fontsize=14&hidenavigation=1&theme=dark)

### An example using context

When building a complex app, you may have multiple components within a page that need to read from and write to the query string. In these cases, using a `useQueryString` hook in each component will cause your query string to fall out of sync, since each invocation of the hook [manages its own internal state](./src/index.ts#L14).

To avoid this issue, use **context** to pass `query` and `setQuery` to descendant components within a page.

```js
// src/pages/billing.js
import React, {createContext, useContext} from 'react';
import useQueryString from 'use-query-string';
import {navigate} from 'gatsby';

// create context to use in parent and child components
const QueryStringContext = createContext();

export default function Billing(props) {
  const [query, setQuery] = useQueryString(props.location, navigate);
  return (
    <QueryStringContext.Provider value={{query, setQuery}}>
      <div>
        <FilterInput name="client" />
        <FilterInput name="industry" />
        <FilterInput name="email" />
      </div>
      {/* render table of filtered data */}
    </QueryStringContext.Provider>
  );
}

function FilterInput(props) {
  const {query, setQuery} = useContext(QueryStringContext);

  function handleChange(event) {
    const {name, value} = event.target;
    setQuery({[name]: value});
  }

  return (
    <input
      name={props.name}
      value={query[props.name]}
      onChange={handleChange}
    />
  );
}
```

[![Edit Nested components example](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/nested-components-example-fyed0?fontsize=14&hidenavigation=1&theme=dark)

## License

[MIT](./LICENSE)
