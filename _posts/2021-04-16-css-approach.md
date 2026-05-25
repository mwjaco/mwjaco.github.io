---
title: "My Approach to Scaling CSS"
excerpt: How I fell in love with CUBE CSS
categories:
  - CSS
tags:
  - react
  - css
---

Topics discussed:

- A description of my preferred approach
- Weighing the pros and cons of the methodologies it incorporates
- Best practices

## CUBE CSS

Of all the methodologies developed in recent years for managing CSS at scale, [CUBE CSS](https://piccalil.li/blog/cube-css), an approach first described by Andy Bell, resonates most with my experience of what works.

CUBE stands for **Components,** **Utility,** **Block,** and **Exception**. No disrespect to Mr. Bell, whose recent book [Every Layout](https://every-layout.dev/) I consider essential, but I have swapped his term "Composition" for "Components," which I finder a bit clearer.

Taking these one at a time, **Components** refers to your team's design system. These are the first tools you should reach for when bringing a design to life.

But components never exist in isolation. Often we need to define the margin or padding between them, or match their colors. My team uses **utility classes**, also known as atomic classes, to make these modest changes. My favorite system is [Tailwind](tailwindcss.com/).

Of course, utility classes alone cannot do much. That's by design. Each represents exactly one pair of a CSS property and value, for instance Tailwind's `list-none`, which removes the bullet points from a `ul` element. When more design firepower is required, we introduce a "block."

**Blocks** refer to a family of BEM selectors (**Block**, **Element**, and **Modifier**). My team uses such blocks everywhere--in both design system components and the larger UIs in which they appear. A single component's stylesheet will use BEM for organizing and naming its selectors, while the stylesheet for an entire view will use BEM to organize the flexbox or CSS Grid implementation necessary to accomplish fine-grained control over a particular design.

Key to how these three parts fit together is this [reminder from Bell](https://piccalil.li/blog/cube-css#heading-block):

> A block at this point is really a minor thing, though, because so much has been handled by the other parts ... If you choose to write your CSS with this sort of methodology, you’ll probably find that your block CSS is tiny, because of this.

Finally, because sometimes rules need to be broken, we have **Exceptions**. Due to the inherent challenges of working with CSS at scale, e.g. specificity and the global namespace, we sometimes need an escape hatch. What if, for instance, we can't access the `className` of a deeply nested React component? Exceptions allow us to break encapsulation once in a while and write a selector that targets a design system component class.

## Pros and cons

### Pro: composable CSS means versatile components

Our style of writing CSS draws on a key insight [Nicole Sullivan](https://github.com/stubbornella) first theorized with [Object-Oriented CSS (OOCSS)](https://github.com/stubbornella/oocss/wiki) nearly a decade ago.

OOCSS refers to the idea, borrowed from object-oriented programming, of packaging up reusable behavior as an **interface layer** and extending that layer where needed.

This helps teams who manage a component library and who confront a familiar CSS architectural challenge:

1. We can't know in advance all of a component's possible usages
2. Nevertheless, we have to make sure it looks the way it's supposed to

For example, say I needed to make a navbar component. With some very naïve CSS and plain HTML I could represent that like this...

```html
<style>
  ul {
    background-color: #333;
    list-style: none;
    display: flex;
  }
  ul li a {
    color: white;
    text-decoration: none;
  }
  ul li a:hover {
    color: tomato;
  }
</style>
<ul>
  <li>
    <a href="#">About</a>
  </li>
  <li>
    <a href="#">Search</a>
  </li>
  <li>
    <a href="#">Categories</a>
  </li>
  <li>
    <a href="#">Tags</a>
  </li>
  <li>
    <a href="#">Posts</a>
  </li>
</ul>
```

...but the styles are too coupled to the markup, and my component isn't reusable. We need our components to be as ignorant of their implementation details as possible, fit for any possible situation.

Fortunately, OOCSS offers [two guiding principles](https://github.com/stubbornella/oocss/wiki#two-main-principles-of-oocss) that point the way forward:

1. **"Separate structure from skin."**

   In other words, uncouple your stylesheet from your implementation details. We should be free to use a `ul`, a `nav`, a `div`, or any element we want in our navbar, and the CSS should still work.

2. **"Separate container from content."**

   Just like elements can change, so can source order and other implementation details. Styles that rely on parent-child relationships, e.g. `ul li a`, are hard to reuse and easily break.

A better pattern--the pattern prescribed by OOCSS--would look more like this:

```html
<style>
  .navigation {
    background-color: #333;
    display: flex;
    list-style: none; /* Remains in the case we need to support ul */
  }

  .navigation-item-link {
    color: white;
    display: inline;
    text-decoration: none;
  }

  .navigation-item-link:hover {
    color: tomato;
  }
</style>

<ul class="navigation">
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">About</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Search</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Categories</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Tags</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Posts</a>
  </li>
</ul>
```

Now switching one element for another, or switching an element for an entire component, is trivial:

```html
<nav class="navigation">
  <div class="navigation-item">
    <a class="navigation-item-link" href="#">About</a>
  </div>
  <div class="navigation-item">
    <a class="navigation-item-link" href="#">Search</a>
  </div>
  <div class="navigation-item">
    <a class="navigation-item-link" href="#">Categories</a>
  </div>
  <div class="navigation-item">
    <a class="navigation-item-link" href="#">Tags</a>
  </div>
  <div class="navigation-item">
    <a class="navigation-item-link" href="#">Posts</a>
  </div>
</nav>
```

We can even change the source order if we like, or flip-flop which element takes which class--extending our object, if you will.

### Pro: BEM makes organizing and naming things easy

Of course, you won't find a selector like `navigation-item-link` in my ideal codebase, because I name and organize my selectors [according to BEM](https://www.integralist.co.uk/posts/bem/):

> BEM stands for “Block, Element, Modifier” and is a simple but effective way to group together different components/widget ... Within each defined ‘Block’ you can have multiple ‘elements’ that make up the object, and for each element (depending on where it appears within the block) you might need to ‘modify’ the state of the element.

Below we can see how OOCSS and BEM complement one another. We respect a separation of concerns: our stylesheet remains uncoupled from the implementation details of our markup.

This is especially useful for interactive UX elements. Say we wanted to add a subnav to our component. Again, the markup doesn't matter, as long as I express the desired behavior through the CSS selectors.

```html
<style>
  .navigation {
    background-color: #333;
    display: flex;
    list-style: none; /* Remains in the case we need to support ul */
  }

  .navigation__item {
    position: relative;
  }

  .navigation__item__link {
    color: white;
    display: inline;
    text-decoration: none;
  }

  .navigation__item__link:hover {
    color: tomato;
  }

  .navigation__item__sub {
    display: none;
    position: absolute;
  }

  .navigation__item:hover .navigation__item__sub {
    display: flex;
    flex-direction: column;
  }
</style>

<ul class="navigation">
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">About</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Search</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Categories</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Tags</a>
  </li>
  <li class="navigation-item">
    <a class="navigation-item-link" href="#">Posts</a>
    <ul class="navigation navigation__item__sub">
      <li className="navigation__item">
        <a className="navigation__item__link" href="#"> Drafts </a>
      </li>
      <li className="navigation__item">
        <a className="navigation__item__link" href="#"> Published </a>
      </li>
    </ul>
  </li>
</ul>
```

Like OOCSS promises, I can combine selectors and thereby hook into multiple features from my integration layer; by combining `.navigation` and `.navigation__item__sub` I can gradually introduce new behavior, combining existing selectors with new ones, while DRYing up my stylesheet.

But BEM has other practical advantages:

1. It encourages us to give our selectors unique names. This eliminates any problems related to CSS's global namespace.
2. It's readable: we can infer some basic facts about the structure of our markup from our stylesheet, even while respecting a separation of concerns.

### Con: the ever-growing append-only stylesheet

When most teams need to style a component, they simply add a Sass file or CSS module (or Styled Component library) to the project, write some new rules, and import it in our component file. At build time, the bundler crawls the codebase, finds the styles, and appends it to a compiled `app.css`, `main.css`, or `styles.css` stylesheet.

In such cases, that means `app.css` has grows longer and longer over time.

The industry term for this is ["append-only stylesheets"](https://css-tricks.com/oh-no-stylesheet-grows-grows-grows-append-stylesheet-problem/) and its problems are detailed in the CSS-Tricks article.

This is far from ideal, but it's unclear how much we should worry thanks to gzipping.

Plus, as I indicated at the start, there are strategies to work within these constraints and reduce the amount of CSS we ship.

## DRYing up what we can with CUBE

### Components

The first way you can limit how much new CSS you write is via React's component paradigm. You write the CSS that governs a component only once, but you can use that component as often as you like. As we just saw, the strategy for styling for any given component is deeply informed by OOCSS and BEM.

### Utility classes

A typical entireprise stylesheet can have nearly 3000 rules for handling margin alone (I know from experience). Theoretically, if you replaced each usage with a utility class, you could achieve the same result in five or six rules.

Utility classes can also accomplish modest changes to the components of our atomic design system.

By replacing their proprietary styles with utility classes, we can improve our components' APIs via the principle of [inversion of control](https://kentcdodds.com/blog/inversion-of-control). Our components become even less opinionated and even more extensible.

For example, I recently worked on a design where I needed finer-grained control over a tab component's overflow behavior. Unfortunately, due to the append-only nature of my compiled stylesheet, the component ignored it when I passed it an `overflow-hidden` selector, because of the top-to-bottom nature of CSS specificity: in my stylesheet, the tab's styles appeared later than the utility class, thus overriding it.

What I needed to do was make its overflow behavior _less_ specific. I abstracted the `overflow: auto;` rule from the tab component's local styles and replaced it with a utility class. Now the selector responsible for the component's overflow behavior was no more specific than the utility class I wanted to pass it. I can control its overflow behavior directly via utility class.

Given the high upside to using utility classes, it's important to design component APIs to be as open as possible. One such strategy for doing so is the primitive pattern, which I'll describe in a future post.

Between the components included in our design system and our library of utility classes, we are prepared to support nearly any request from our designers.

The exception is complex layouts, which brings us back to blocks.

### Blocks: One-off implementations to solve complex design challenges

Most teams practice responsive design and deliver tailored user experiences to every device size. Since our page layouts tend to be so highly complex, it's arguable they would be ill-served by utility classes. Design requirements may prove demanding enough that the freedom to write vanilla CSS--directly manipulating the flexbox and grid APIs, versus using an abstraction layer--is well worth the additional lines of code you ship.

This reality, along with our ability to negotiate selector specificity demonstrated earlier, means that _any_ CSS management strategy rewards a mastery of flexbox and CSS grid.

### Exceptions: Knowing when to break the rules

Our CSS management strategy also rewards a strong understanding of selector specificity and the CSS cascade, the better with which to carve out exceptions to what OOCSS and BEM might dictate.

## Takeaways

### 1. Aim to be a CSS expert

Strong front-end engineer can read, write, and debug CSS using the native API. We understand how selector specificity works. We can progressively enhance or gracefully degrade an experience for different devices using only media queries and vendor prefixes--no JavaScript or browser-sniffing.

#### 2. ...but we want to write as little of it as possible

Practice good component architecture and reuse as much of your work as possible. This limits how much CSS you ultimately ship.

#### 3. Too much abstraction benefits no one

Every abstraction layer we add between our components and our CSS stylesheets makes #1 that much more difficult. First, it puts us at a remove from manipulating CSS directly. It also introduces more cognitive overhead.

E.g. most design systems have a `<Card />` component. If you're using React, the `<Card />` will often accept a `paddingLevel` prop. This is an inferior API to manipulating the CSS directly via the `className` prop, and pass it a utility class. Why? Because now, instead of learning one API (our utility classes, or CSS itself), we have to remember what `paddingLevel` does too. After a while, these "gotchas" start to add up.

## Further Reading

What follows is some of my favorite writing about CSS:

[Andy Bell, "CUBE CSS"](https://piccalil.li/blog/cube-css/)
[Nicole Sullivan, Object Oriented CSS (OOCSS)](https://github.com/stubbornella/oocss/wiki)
[Adam Wathan, "CSS Utility Classes and Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)
[Andy Bell and Heydon Pickering, "Every Layout"](every-layout.dev)
[Jonathan Snook, "Scalable and Modular Architecture for CSS (SMACSS)"](http://smacss.com/)
