# Optimizing web performances and critical rendering path

The **Critical Rendering Path** is the sequence of steps the browser goes through to convert the HTML, CSS, and JavaScript into pixels on the screen.

JavaScript > Style > Layout > Paint > Composite

---

## Improve Runtime Performance

If there is any kind of visual change on the screen like:

- scrolling
- hover effect
- animation

the browser is going to render a new frame.

If the browser makes too long to make the frame, the frame rate will drop and users will see that.

In many cases, that's has nothing to do with JavaScript, the problem is CSS.

---

## Fast animations

Need to understand how the browser works, how it renders pixels from HTML and CSS.

- Render blocking CSS

---

## Optimize initial rendering

- Critical resources
- Critical bytes

CSS is **render blocking** by default

> "Yahoo increased traffic by 9% for each 400ms of improvement in load time"

> "Amazon saw that every 100ms increase in load time resulted in a 1% decrease in revenue."

Users like **fast** web applications.

---
