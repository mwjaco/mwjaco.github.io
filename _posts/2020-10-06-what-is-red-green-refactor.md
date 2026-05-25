---
title: 'What is "Red Green Refactor"?'
excerpt: "Doing TDD the right way"
categories:
  - Testing
tags:
  - testing
  - react
---

I used to wait until I was finished with my features before writing tests for my components. Not once did I feel motivated to write the tests--the work was done! Not to mention components rarely stand alone; by the time we complete a feature, the dependency tree is very complex. To write my tests, I had to reverse-engineer all my work.

In other words, the biggest problem with writing tests at the end of a feature is that it's hard. That's where the ["red-green-refactor"](https://youtu.be/EZ05e7EMOLM?t=2402) (RGR) methodology comes in.

It's true that adapting RGR as a habit is not at all simple, especially if you are new to many of the tools involved (like we all are at some point). You will pay an upfront cost as you refactor and refine your tests, but the more acquainted you are with the ecosystem, the more quickly you'll find your way around. Plus, I'm going to point out some common pitfalls below.

## Red: write the tests first, one for each requirement

First described by Kent Beck in _Test-Driven Development: By Example_, RGR suggests we write the tests first, without any code to execute. The tests will fail at first--that's the "red" part.

But what to test? Start with the your product requirements. Here is a test for a hypothetical `<select>` menu that displays some items fetched from an API, but also inserts some items of its own, like an "All Items" option.

```javascript
import { screen, render, fireEvent } from "@testing-library/react";
import ItemPicker from "../ItemPicker";

describe("ItemPicker", () => {
  test("displays 'ALL ITEMS' by default", () => {
    render(<ItemPicker />);

    const select = await screen.findByRole("combobox", { name: /item/i });
    expect(select).toHaveDisplayValue("ALL ITEMS");
  });

  test("updates internal state", () => {
      render(<ItemPicker />);

    const select = await screen.findByRole("combobox", { name: /item/i });
    fireEvent.change(select, { target: { value: "Foo" } });
    expect(select).toHaveDisplayValue("Foo");
  });
});
```

## Green: Write the bare minimum required for the tests to pass

Let's pause and ask why we should rush the code just to get the tests to pass.

The answer lies in the final step, "refactor." When it's time to write the code as intended, we can use our tests--not our local application--to guide us. Personally, this has saved me a _lot_ of time because not all build systems are built equally; some are fast, some are slow, some are missing hot module reloading altogether. None are as fast as Jest.

Here, then, is the bare minimum:

```javascript
function ItemPicker() {
  return (
    <label>
      Item:
      <select>
        <option value="ALL ITEMS">ALL ITEMS</option>
        <option value="Foo">Foo</option>
        <option value="Bar">Bar</option>
        <option value="Baz">Baz</option>
      </select>
    </label>
  );
}

export ItemPicker;
```

So, what shortcuts did we take?

1. We used no advanced or third-party component libraries, just semantic HTML to satisfy [React Testing Library's queries](https://testing-library.com/docs/queries/about/).
2. The component isn't stateful. It simply uses the default behavior afforded to us by the browser.
3. We hard-coded the data of our menu into the component itself.

### Refactor: where the rubber meets the road

Our component satisfies our tests, and our tests are providing instant feedback on our code. But it's still very contrived. And how resilient are these tests anyway?

If there is a false promise to RGR, it's this: although you should expect to iterate on your component many times (after all, the third letter in RGR stands for "refactor") your tests will likely change too, depending on your familiarity with Jest and its ecosystem.

For example, what if I want my component not to be merely presentational, but to function more like a [container component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). I.e., I don't want it to consume props from a parent; I want it to be self-hydrating from a GraphQL endpoint.

Here's what that would look like:

```javascript
+ import {useState} from 'react';
+ import { gql, useQuery } from "@apollo/client";

+ const ITEMS = gql`
+   query Items {
+     items {
+       id
+       name
+     }
+   }
+ `;

function ItemPicker() {
+  const [value, setValue] = useState('ALL ITEMS')
+  const { data, loading } = useQuery(ITEMS);
+
+  if (loading) {
+    return null;
+  }

  return (
    <label>
      Item:
      <select
        onChange={(e) => setValue(e.target.value)}
        value={value}
      >
+        <option value="ALL ITEMS">ALL ITEMS</option>
+        {data.items.map((item) => (
+          <option value={item.id}>[item.name]</option>
+        ))}
      </select>
    </label>
  );
}

+ export {ItemPicker, ITEMS};
```

Now, when my component mounts, I make a request to a backend. I've replaced the hard-coded options with an iterator that cycles through the items I get back.

This will obviously break my tests, but that doesn't mean there is anything wrong with my component--my tests aren't set up to handle the backend dependency I've introduced.

So let's mock the backend. Conveniently, Apollo provides a component for this exact purpose, called `MockedProvider`. Pass it an object that resembles as closely as possible a real-life response from your API. For example, don't forget the `__typename` property from your schema!

```javascript
import {screen, render, fireEvent} from "@testing-library/react";
+ import {ItemPicker, ITEMS} from "../ItemPicker";
+ import { MockedProvider } from "@apollo/client/testing";

+ const data = {
+   items: [
+     {
+       id: "Foo",
+       name: "Foo",
+       __typename: "Item"
+     },
+     {
+       id: "Bar",
+       name: "Bar",
+       __typename: "Item"
+     },
+     {
+       id: "Baz",
+       name: "Baz",
+       __typename: "Item"
+     },
+   ]
+ };

describe("ItemPicker", () => {
  test("displays 'ALL ITEMS' by default", () => {
    render(
+      <MockedProvider
+        mocks={
+          [
+            {
+              request: {
+                query: ITEMS
+              },
+              result: {
+                data
+              }
+            }
+          ]}
+      >
+        <ItemPicker />
+      </MockedProvider>
    );

    const select = await screen.findByRole("combobox", { name: /item/i });
    expect(select).toHaveDisplayValue("ALL ITEMS");
  });

  test("updates internal state", () => {
      render(
+        <MockedProvider
+          mocks={
+            [
+              {
+                request: {
+                  query: ITEMS
+                },
+                result: {
+                  data
+                }
+              }
+            ]}
+        >
+          <ItemPicker />
+        </MockedProvider>
      );

    const select = await screen.findByRole("combobox", { name: /item/i });
    fireEvent.change(select, { target: { value: "Foo" } });
    expect(select).toHaveDisplayValue("Foo");
  });
});
```

My tests will now intercept the component's calls to the API--but more important, the component is done! This is RGR in a nutshell: using tests not to verify your code works (although they should do that too) but as a virtual assistant guiding your code to completion.

## Recommended resources

Kent Beck, "Test-Driven Development: By Example"

Ian Cooper, ["TDD: Where did it all go wrong?"](https://www.youtube.com/watch?v=EZ05e7EMOLM)
