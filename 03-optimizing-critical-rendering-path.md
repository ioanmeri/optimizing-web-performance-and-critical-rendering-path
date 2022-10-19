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

No more green flashing with the Paint profiler!

Confirm this by tracking the paint count

**The layout and paint is no more triggered.**

---

## Multiple Layers

We can promote every element to a layer.

```
body * {
  will-change: transform;
}
```

but in this way we have thousands of layers and the browser has too much work to compose them.

The more layers you have, the more time will be spend on layer managament and composing it.

If you have too many layers the composite and update layer tree will grow very large.

There is a trade off between reducing **Paint** time and **Composite Layers**

So:

- Promote layers only when it makes sense
- Avoid Paint problem
- Let the browser manage layers
- Layer management and compositing are not free
- Never promote elements without profiling

---

## Older browsers

In older browser the header while scrolling is totally green. So we have a painting problem again.

The layout and paint are triggered during the scroll in each frame. This is regular scrolling, nothing special, no JS.

If we look at layers, it seems that different browser engines or even different versions may manage layers in different way.

To fix this problem, let's promote the element to a layer

```
will-change: transform;
```

No more paint and there is a new layer. But how can we know if we should promote an element to a new layer?

---

## Properties impact

Different CSS properties have a different effect on the rendering process.


Example

### left

Changing `left` alters the geometry of the element. That means that it may affect the position or size of other elements on the page, both of which require the browser to perform **layout operations**.

Once those layout operations have completed any damaged pixels will need to be **painted** and the page must then be **composited** together.


### width

It also triggers the pipeline

### transform

Changing `transform` does not trigger any geometry changes or painting. which is very good. This means that the operation can likely be carried out by the **compositor thread** with the help of the **GPU**.

In the rendering process, the main thread handles most of the code we send to the user. 

The compositor thread rasterize each layer. A layer could be very large so the compositor thread divides it into tiles and sends each tile off to raster threads, it rasterizes each tile and stores them in GPU memory. It's very easy for the GPU to do transformations with these tiles.

GPU is higly optimized for tasks like moving pixels around and changing opacity. Once a layer it's created, it's trivial for the GPU to move those pixel around and composite them together. But the GPU is very limited for other things, for example changing color.

In order to change color, we need to repaint it on CPU and re-upload to the GPU, and this might be expensive.

On the other hand, transform doesn't trigger the layout, neither paint **IF** the element has its **OWN LAYER**

Transform is the best property to animate our sidebar.

### Animating the transform property if the element has each own layer, looks like this:

**Style** > **Composite**

The GPU moves those pixels and composes them together.

### Animating left property

**Style** > **Layout** > **Paint** > **Composite**

we still have a layout and paint and this is what we want to avoid.

If we animate any properties that affect the element position like:
- left
- right
- top
- bottom...

or any properties that affects the elements geometry like
- width
- height
- padding
- margin...

they will trigger the layout.

If you trigger the layout, any affected pixels will need to be repainted.

On the other hand

### Animating the "paint only"

This will always trigger the paint

**Style** > **Paint** > **Composite**

no matter if the element has it's own layer, or not.

If you change any visual property like:

- Background color
- color
- visibility
- shadows etc.

it will trigger the paint. Affected area needs to be repainted. Creating new layer in this situation **will be even worse** because:

- The CPU still has to paint affected area
- Extra layer

So the browser has more work to do in layer management.

In this situation the pipeline looks like this:

**Style** > **Paint** > **Composite**

So if you need to animate `background-color`, you can't eliminate the paint, it will be triggered in any case.

---




