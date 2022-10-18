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

## Paint Profiler

In Chrome: More Tools > Rendering > Check **Paint flashing**

It will show you when and where painting is happening on the page.

Confirm slow animation by opening a layer tab

More tools > **Layers**

The is one useful piece of data: the **Paint count**

e.g.: The sidebar layer is being repainted all the time, this is what we want to avoid.

---

## Performance Analysis

If the problem was caused by JS, the issue would exist only in the **Main** area, also adding one class is (sidenav is open) is very simple.

The problem is in animating the **left** CSS property

Can try animate the width but the frame rate is still 17.

**Layout and paint are triggered in every paint**

Instead, you should use **transform**. We achieve a good animation, the frame rate is around 60.

There are still

- **Recalculate Style**
- **Update Layer Tree**
- **Compose**

but we lose the:

- **Layout**
- **Paint**

These are great news.

Enable paint flashing to confirm this. There is flashing only at the end, but is seems that we still have unnecessary paint (paint count increases by 1 every time we open the sidenav).

---

## How Layout works

We should understand the implication of every property we try to animate.

If we move the sidebar to a new layer, we don't paint it all the time.

With a separate layer, we can slide, rotate, change opacity etc.

Creating layers is an automated process and we should let the browser manage it.

But if you have a painting issue you might want to promote an element to it's own layer.

---

## Creating a new layer

The best way to create a new layer is to use the:

```
will-change: transform;
```

property. That tells the browser that we intent to change the element's transform property at some point.

Most browsers react to this by **putting an element on it's own element**.

For older browsers can use the property:

```
transform: translateZ(0);
```

```
.sidebar {
  left: -450px;
  width: 450px;
  position: absolute;
  transition: 1s;
  will-change: transform;
}

.sidebar.is-open {
  transform: translateX(450px);
}
```

No green flashing with the Paint profiler!

Confirm this by tracking the paint count

The paint is no more triggered.

---
