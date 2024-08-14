<!-- TABLE OF CONTENTS -->

<a id="table-of-contents"></a>

<details>
  <summary>Table of Contents</summary>
  <ul>
    <li>
      <a href="#usedeepcompareeffect">useDeepCompareEffect</a>
    </li>
      <li><a href="#useeffectskipfirst">useEffectSkipFirst</a></li>
       <li><a href="#usetitle">useTitle</a></li>
      <li><a href="#usewindowresize">useWindowResize</a></li>
  </ul>
</details>

<br />

<!-- useDeepCompareEffect-->

## useDeepCompareEffect

```js
import { useEffect, useRef, DependencyList } from "react";
import _isEqual from "lodash/isEqual";

type EffectCallback = () => void;

const useDeepCompareEffect = (
  callback: EffectCallback,
  dependencies: DependencyList
) => {
  const previousDependencies = useRef < DependencyList > dependencies;

  useEffect(() => {
    if (!_isEqual(previousDependencies.current, dependencies)) {
      callback();
    }
    previousDependencies.current = dependencies;
  }, [callback, dependencies]);
};

export default useDeepCompareEffect;
```

[🔝 Back to table of contents](#table-of-contents)
<br />
<br />
This hook works like `useEffect` but performs a deep comparison of the dependencies using `lodash` `isEqual`. It helps when you have complex objects or arrays as dependencies and want to avoid unnecessary re-renders.

### Usage:

```jsx
import React, { useState } from "react";
import useDeepCompareEffect from "./useDeepCompareEffect";

const MyComponent = () => {
  const [data, setData] = useState({ name: "Alice" });

  useDeepCompareEffect(() => {
    console.log("Effect called due to deep comparison of dependencies");
  }, [data]);

  return (
    <div>
      <button onClick={() => setData({ name: "Alice" })}>Update Name</button>
    </div>
  );
};
```

- When to use: When you need to compare complex objects or arrays deeply in dependency arrays of `useEffect`.

<br />

<!-- useEffectSkipFirst-->

## useEffectSkipFirst

```js
import { useEffect, useRef, EffectCallback, DependencyList } from "react";

const useEffectSkipFirst = (
  effect: EffectCallback,
  dependencies: DependencyList
) => {
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
    } else {
      return effect();
    }
  }, dependencies);
};

export default useEffectSkipFirst;
```

[🔝 Back to table of contents](#table-of-contents)
<br />
<br />

This hook skips the effect on the first render and only runs on subsequent renders when dependencies change. It is useful for scenarios where you don't want an effect to run during the initial render.

### Usage:

```js
import React, { useState } from "react";
import useEffectSkipFirst from "./useEffectSkipFirst";

const MyComponent = () => {
  const [count, setCount] = useState(0);

  useEffectSkipFirst(() => {
    console.log("Effect ran after the first render");
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

- When to use: When you need to prevent an effect from running on the initial render.

<br />

<!-- useTitle-->

## useTitle

```js
import { useEffect } from "react";

const useTitle = (title = "Default title") => {
  useEffect(() => {
    const prevTitle = document.title;
    document.title = title;

    return () => {
      document.title = prevTitle;
    };
  }, []);
};
export default useTitle;
```

[🔝 Back to table of contents](#table-of-contents)
<br />
<br />

This hook updates the document's title and reverts it to the previous title when the component is unmounted.

### Usage:

```js
import React from "react";
import useTitle from "./useTitle";

const MyComponent = () => {
  useTitle("New Page Title");

  return <div>Check the document title!</div>;
};
```

- When to use: When you want to update the browser's title dynamically based on a component's lifecycle.

<br />

<!-- useWindowResize-->

## useWindowResize

```js
import { useState, useLayoutEffect } from 'react';

export const useWindowResize = () => {
  const [size, setSize] = useState<number[]>([0, 0]);

  useLayoutEffect(() => {
    function updateSize() {
      setSize([window.innerWidth, window.innerHeight]);
    }

    window.addEventListener('resize', updateSize);
    updateSize();

    return () => window.removeEventListener('resize', updateSize);
  }, []);

  return size;
};
```

[🔝 Back to table of contents](#table-of-contents)
<br />
<br />

This hook tracks the window's width and height and returns the current size in real-time as the window is resized.

### Usage:

```js
import React from "react";
import { useWindowResize } from "./useWindowResize";

const MyComponent = () => {
  const [width, height] = useWindowResize();

  return (
    <div>
      <p>Window width: {width}px</p>
      <p>Window height: {height}px</p>
    </div>
  );
};
```

- When to use: When you need to track the window's dimensions for responsive layouts or dynamic rendering based on window size.

<br />
