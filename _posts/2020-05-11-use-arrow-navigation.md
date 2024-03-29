---
title: "Implementing a custom useArrowNavigation hook"
excerpt: "Or, how to make anything a carousel"
categories:
  - React
tags:
  - react
  - hooks
---

We are implementing a "carousel" design (AKA [a content slider](https://inclusive-components.design/a-content-slider/)) using hooks. In the most general terms, the pattern will consist of the carousel parent and its children, the items. With a hook, we can abstract away the carousel-like logic to suit any use case. The only requirement is that the items are navigable with the arrow keys.

## The `Carousel` component

In this tutorial I'll start as I would in real life: test the idea out inside a functional component first, and then migrate the behavior into its own hook.

We'll bring in the `useState` hook to track our roving focus, or what I'm calling our **cursor**.. We'll initialize it with a position of `0`, indicating the cursor hasn't moved yet.

```jsx
import React, { useState } from "react";

function Carousel({ items }) {
  const { cursor, setCursor } = useState(0);

  return (
    <ul>
      {items.map((item, i) => (<li>Item #{`${i}`}</li>)}
    </ul>
  );
}
```

This renders something like the following:

```html
<ul>
  <li>Item #0</li>
  <li>Item #1</li>
  <li>Item #2</li>
  <li>Item #3</li>
</ul>
```

## Our event handler

Next, we'll describe the function for handling our keyboard behavior. We are calling it `handleKeyDown`. This hints at the challenge we face here. "Keyboard events are only generated by `<inputs>`, `<textarea>` and anything with the contentEditable attribute or with `tabindex='-1'`," [MDN tells us](https://developer.mozilla.org/en-US/docs/Web/API/Element/keydown_event). **We can assume this function must be callable from the carousel's children, the items, not the parent.** Put another way, the result of `e.target` in the handler below should be the whichever of our carousel's children was in focus when a key was pressed.

```jsx
import React, { useState } from "react";

function Carousel({ items }) {
  const { cursor, setCursor } = useState(0);

+  function handleKeyDown(e) {
+    if (e.key === "ArrowRight") {
+      e.preventDefault();
+      setCursor(); // TBD
+    } else if (e.key === "ArrowLeft") {
+      e.preventDefault();
+      setCursor(); // TBD
+    }
+  }

  return; // ...;
}
```

Now, what is the new index to feed to the cursor? Well, the item that calls it will certainly know its own index. So let's update the `handleKeyDown` function to take a second argument: the current item's index. Then, based on which key was pressed, we can add or subtract to find the new index.

```jsx
import React, { useState } from "react";

function Carousel({ items }) {
  const { cursor, setCursor } = useState(0);

  function handleKeyDown(e, index) {
    if (e.key === "ArrowRight") {
      const idxToSet = index + 1;
      setCursor(idxToSet);
    } else if (e.key === "ArrowLeft") {
      e.preventDefault();
      const idxToSet = index - 1;
      setCursor(idxToSet);
    }
  }

  return; // ...;
}
```

Now let's take a first pass at implementing this in our functional component. Remember, according to MDN, we must give our items a `tabIndex` to make them keyboard-navigable.

```jsx
import React, { useState } from "react";

function Carousel({ items }) {
  const { cursor, setCursor } = useState(0);

  function handleKeyDown(e, index) {
    if (e.key === "ArrowRight") {
      const idxToSet = index + 1;
      setCursor(idxToSet);
    } else if (e.key === "ArrowLeft") {
      e.preventDefault();
      const idxToSet = index - 1;
      setCursor(idxToSet);
    }
  }

  return (
+    <ul>
+      {items.map((item, i) => {
+        return (
+          <li
+            tabIndex={cursor === i ? 0 : -1}
+            onKeyDown={(e) => handleKeyDown(e, i)}
+          >
+            Item #{`${i}`}
+          </li>
+        );
+      }}
+    </ul>
  );
}
```

Now our problem is that as soon as the index either a) drops below `0` or b) goes beyond the length of our array of items, our cursor will simply disappear. To account for that, we compare the incoming cursor value to the length of our array.

```jsx
import React, { useState } from "react";

function Carousel({ items }) {
  const { cursor, setCursor } = useState(0);

+  function handleKeyDown(e, index) {
+    const length = items.length;
+    if (e.key === "ArrowRight") {
+      e.preventDefault();
+      const isLastItem = index === length - 1;
+      const nextIdx = index + 1;
+      const idxToSet = isLastItem ? 0 : nextIdx;
+      setCursor(idxtoSet);
+    } else if (e.key === "ArrowLeft") {
+      e.preventDefault();
+      const isFirstItem = index === length - 1;
+      const nextIdx = index - 1;
+      const idxToSet = isFirstItem ? length : nextIdx;
+      setCursor(idxToSet);
+    }
+  }

  return; // ...;
}
```

As we can see, each navigable item needs access to two things: the handler itself and the cursor. It needs the function so that our handler is dispatched with the right event, and it needs the current cursor value to conduct any side effects, like setting/unsettings its own `tabIndex`.

## Writing our hook

Now we should have a working prototype! Let's refactor this into a hook that we can use anywhere.

```jsx
import React, { useState } from "react";

function useArrowNavigation(items) {
  const { cursor, setCursor } = useState(0);

  function handleKeyDown(e, index) {
    const length = items.length;
    if (e.key === "ArrowRight") {
      e.preventDefault();
      const isLastItem = index === length - 1;
      const nextIdx = index + 1;
      const idxToSet = isLastItem ? 0 : nextIdx;
      setCursor(idxtoSet);
    } else if (e.key === "ArrowLeft") {
      e.preventDefault();
      const isFirstItem = index === length - 1;
      const nextIdx = index - 1;
      const idxToSet = isFirstItem ? length : nextIdx;
      setCursor(idxToSet);
    }
  }

  return { handleKeyDown, cursor };
}
```

## Optimizing our hook

We may want to limit how often we instantiate a new `onKeyDown` function. We can take advantage of the `useCallback` hook to capture a stable reference to the function. That way, we aren't passing new function instances around on every render. We can take advantage of `useCallback`'s dependency array to make sure it registers a new function should the number of items change on us.

```jsx
+ import React, { useCallback, useState } from "react";

function useArrowNavigation(items) {
  const { cursor, setCursor } = useState(0);

+  const handleKeyDown = useCallback((e, index) => {
+    const length = items.length;
+    if (e.key === "ArrowRight") {
+      e.preventDefault();
+      const isLastItem = index === length - 1;
+      const nextIdx = index + 1;
+      const idxToSet = isLastItem ? 0 : nextIdx;
+      setCursor(idxtoSet);
+    } else if (e.key === "ArrowLeft") {
+      e.preventDefault();
+      const isFirstItem = index === length - 1;
+      const nextIdx = index - 1;
+      const idxToSet = isFirstItem ? length : nextIdx;
+      setCursor(idxToSet);
+    }
+  }, [items.length]);

  return {handleKeyDown, cursor}
}
```

## Putting it all together

Let's check on what our carousel component looks like now, using our new hook.

```jsx
import React, { useState } from "react";
import useArrowNavigation from "./useArrowNavigation";

function Carousel({ items }) {
  const {cursor, handleKeyDown} = useArrowNavigation(items);

  return (
    <ul>
      {items.map((item, i) => {
        return (
          <li
            tabIndex={cursor === i ? 0 : -1}
            onKeyDown={(e) => handleKeyDown(e, i)}
          >
            Item #{`${i}`}
          </li>
        );
      }}
    </ul>
  );
}
```
