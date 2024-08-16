<!-- TABLE OF CONTENTS -->

<a id="table-of-contents"></a>

# Table of Contents

  <ul>
    <li>
      <a href="#usedeepcompareeffect">useDeepCompareEffect</a>
    </li>
      <li><a href="#useeffectskipfirst">useEffectSkipFirst</a></li>
       <li><a href="#usetitle">useTitle</a></li>
      <li><a href="#usewindowresize">useWindowResize</a></li>
      <li><a href="#usetoastnotification">useToastNotification</a></li>
      <li><a href="#usehandleerror">useHandleError</a></li>
  </ul>

<br />

<!-- useDeepCompareEffect-->

## useDeepCompareEffect

```ts
// useDeepCompareEffect.ts
import { useEffect, useRef, DependencyList } from "react";
import _isEqual from "lodash/isEqual";

type EffectCallback = () => void;

const useDeepCompareEffect = (
  callback: EffectCallback,
  dependencies: DependencyList
) => {
  const previousDependencies = useRef<DependencyList | null>(null);

  useEffect(() => {
    if (
      previousDependencies.current === null ||
      !_isEqual(previousDependencies.current, dependencies)
    ) {
      callback();
    }
    previousDependencies.current = dependencies;
  }, [callback, dependencies]);
};

export default useDeepCompareEffect;
```

[🔝 Back to table of contents](#table-of-contents)
<br />

This hook works like `useEffect` but performs a deep comparison of the dependencies using `lodash` `isEqual`. It helps when you have complex objects or arrays as dependencies and want to avoid unnecessary re-renders.

### Usage:

```tsx
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

### When to use:

- When you need to compare complex objects or arrays deeply in dependency arrays of `useEffect`.

<br />

<!-- useEffectSkipFirst-->

## useEffectSkipFirst

```ts
// useEffectSkipFirst.ts
import { useEffect, useRef, EffectCallback, DependencyList } from "react";

const useEffectSkipFirst = (
  effect: EffectCallback,
  dependencies: DependencyList
) => {
  const isFirstRender = useRef<boolean>(true);

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

This hook skips the effect on the first render and only runs on subsequent renders when dependencies change. It is useful for scenarios where you don't want an effect to run during the initial render.

### Usage:

```tsx
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

### When to use:

- When you need to prevent an effect from running on the initial render.

<br />

<!-- useTitle-->

## useTitle

```js
// useTitle.ts
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

### When to use:

- When you want to update the browser's title dynamically based on a component's lifecycle.
  <br />

<!-- useWindowResize-->

## useWindowResize

```ts
// useWindowResize.ts
import { useState, useLayoutEffect } from "react";

const useWindowResize = () => {
  const [size, setSize] = useState<number[]>([0, 0]);

  useLayoutEffect(() => {
    function updateSize() {
      setSize([window.innerWidth, window.innerHeight]);
    }

    window.addEventListener("resize", updateSize);
    updateSize();

    return () => window.removeEventListener("resize", updateSize);
  }, []);

  return size;
};
export default useWindowResize;
```

[🔝 Back to table of contents](#table-of-contents)
<br />

This hook tracks the window's width and height and returns the current size in real-time as the window is resized.

### Usage:

```js
import React from "react";
import useWindowResize from "./useWindowResize";

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

### When to use:

- When you need to track the window's dimensions for responsive layouts or dynamic rendering based on window size.

<br />

<!-- useToastNotification-->

## useToastNotification

```ts
import { toast, ToastOptions } from "react-toastify";

const useToastNotification = () => {
  const options: ToastOptions = {
    position: "top-right",
    autoClose: 5000,
    hideProgressBar: false,
    closeOnClick: true,
    rtl: false,
    pauseOnFocusLoss: true,
    draggable: true,
    pauseOnHover: true,
  };

  // Function to display a success toast
  const toastSuccess = (message: string) => {
    toast.success(message, options);
  };

  // Function to display an error toast
  const toastError = (message: string) => {
    toast.error(message, options);
  };

  // Function to display an info toast
  const toastInfo = (message: string) => {
    toast.info(message, options);
  };

  // Function to display a warning toast
  const toastWarning = (message: string) => {
    toast.warning(message, options);
  };

  return { toastSuccess, toastError, toastInfo, toastWarning };
};

export default useToastNotification;
```

[🔝 Back to table of contents](#table-of-contents)
<br />

`useToastNotification` is a custom React hook that provides a set of functions to display toast notifications using the `react-toastify` library. It abstracts the configuration of toast notifications, allowing you to easily display success, error, info, and warning messages with consistent styling and behavior across your application.

### Usage:

```jsx
import React from "react";
import useToastNotification from "./useToastNotification";

const MyComponent = () => {
  const { toastSuccess, toastError, toastInfo, toastWarning } =
    useToastNotification();

  const handleClick = (type: "success" | "error" | "info" | "warning") => {
    switch (type) {
      case "success":
        toastSuccess("This is a success message!");
        break;
      case "error":
        toastError("This is an error message.");
        break;
      case "info":
        toastInfo("This is an informational message.");
        break;
      case "warning":
        toastWarning("This is a warning message.");
        break;
    }
  };

  return (
    <div>
      <button onClick={() => handleClick("success")}>Show Success</button>
      <button onClick={() => handleClick("error")}>Show Error</button>
      <button onClick={() => handleClick("info")}>Show Info</button>
      <button onClick={() => handleClick("warning")}>Show Warning</button>
    </div>
  );
};

export default MyComponent;
```

When to Use:

- User Feedback: Use this hook to provide immediate feedback to users about the outcome of their actions, such as form submissions, data processing, or other interactions.
- Error Handling: Display error messages when something goes wrong, such as failed API requests or validation errors, to inform users about issues that need their attention.
- Status Updates: Show information or updates about ongoing processes or statuses, such as successful data retrieval or warnings about potential issues.
- Consistent Notifications: Ensure that all toast notifications in your application follow a consistent style and behavior by centralizing the configuration and logic in this hook.
  <br />

<!-- useHandleError-->

## useHandleError

```ts
// useHandleError.ts
import axios from "axios";
const useHandleError = () => {
  const showError = (error: any, showErrorComponent: (msg: string) => void) => {
    let message: string = "";
    // Check if the error is an Axios error
    if (axios.isAxiosError(error)) {
      message = error?.response?.data?.message;
    } else {
      message = error?.message; // Fallback for non-Axios errors
    }

    if (message) {
      showErrorComponent(message);
    } else {
      showErrorComponent("Something went wrong please try again later!");
    }
  };
  return {
    showError,
  };
};

export default useHandleError;
```

[🔝 Back to table of contents](#table-of-contents)
<br />

`useHandleError` is a custom React hook designed to handle and display error messages. It provides a `showError` function that processes errors and displays appropriate messages based on the type of error encountered. It differentiates between errors originating from Axios (an HTTP client) and other types of errors, ensuring that meaningful messages are shown to the user.

### Usage:

```jsx
import React from 'react';
import axios from 'axios';
import useHandleError from './to/useHandleError';
import useToastNotification from './useToastNotification';

const MyComponent = () => {
  const { showError } = useHandleError();
  const { toastError } = useToastNotification();

  const handleApiCall = async () => {
    try {
      const response = await axios.get('https://api.example.com/data');
      // Process response
    } catch (error: unknown) {
      showError(error, toastError);
    }
  };

  return (
    <div>
      <button onClick={handleApiCall}>Fetch Data</button>
    </div>
  );
};

export default MyComponent;
```

### When to Use

- API Error Handling: Use this hook to handle errors from API requests, especially when using Axios, to provide users with clear and actionable error messages.
- Error Messaging: Use this hook in any component where you need to process and display error messages, ensuring consistency in how errors are communicated to users.
- Error Fallbacks: Use it to handle unexpected errors by providing a default error message when the specific error message is not available.
