# Optimizing critical rendering path

## Introduction

The **Critical Rendering Path** is the sequence of steps the browser goes through to convert the HTML, CSS, and JavaScript into pixels on the screen.

Network > JavaScript > Render Tree > Layout > Paint > Composite

We optimizing the **Critical Rendering Path** we can improve:

- First Render

- Runtime performance (need to achieve 60fps)
  - optimize server
  - use caching
  - optimizing JS
  - lazy loading
  - pre-loading
  - and more

but we will talk only about **CSS**.

---

## Frame Rate

To run a smooth animation there are (1000ms / 60) = **16.6**ms per frame.

But actually 12ms for browser processing.

When animation takes longer than that, the frame rate drops.

Our focus is to reach 60fps when it matters to users - cannot achieve all the time.

---

## The Performance of style calculation

**The number of elements**

Changing style of 10 elements, is 10 times more expensive than changing only one.

First thing: **have less styles**

We can achieve this by writing a better and reusable CSS:

- limit the overriding of CSS styles
  - to write a new CSS to fix the old one is really bad

**Specificity war**

`nav a span.list-item: black !important`

~~a .list-item: red~~

~~a span: blue~~

~~a: red~~

We need **modular CSS**

Another one is:

**Selector Matching**

more specific selectors require more work

Browsers do selectors matching from the right side to the left

`nav#header ul.sidebar li {}`

- The browser finds all `li` elements
- next `ul.sidebar`
- `nav#header`
- more specific rules are more expensive, because it has travel more nodesq

`.list-item{}`

- more simple
- the browser only needs to find the elements with that class
- no more DOM travel
- definitely faster

---

```

```
