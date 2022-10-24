# Optimizing Critical Resources and Critical Bytes

## Introduction

To optimize the Critical Rendering Path even more, we need to improve download time - the Network. We just need critical resources.

A **Critical Resource** is a resource that could block initial rendering of the page.

**Critical Bytes** are the total number of bytes required to complete the initial render of the page.

### Optimize Initial Rendering

- Critical Resources
- Critical Bytes

What's the **minimum** amount of resources that we can ship to render the page?

We need to show to user the most important content in the first second or two.

We don't want to wait for all files to be downloaded before we can show something on the screen.

---

## Parser Blocking JavaScript

The main blockers for accelerating rendering are **style sheets** and **scripts**

JavaScript is **parse blocking** by default

When the HTML parser finds a script tag, it pauses the parsing of the HTML document and has to load, parse and execute the JavaScript code.

The browser waits for the script to be fetched from the server, which can add thousands of milliseconds of delay to the critical rendering path.

Therefore, synchronous scripts in the head of the HTML, **block the entire page** from rendering until they finish loading.

To eliminate unnecessary JavaScript from the critical Rendering Path, one option is to make the JavaScript async

```
<script async src="scripts/script.js"></script>
```

This allows the browser to continue to construct the DOM, and let the script execute when it's ready. But you can use it only if the JS doesn't change the DOM structure.

The script will be downloaded, in both cases. But the script is **no longer parser blocking**

There is a common rule to put script tags just before closing the body.

What if you have a large image?
The good things is that images do not the render of the page. Browser will render a page even if an image has not arrived at all.

---

## Render Blocking CSS

CSS is a **render blocking** resource (by default)

The browser will not start creating the CSS Object Model without fully downloaded CSS.

CSS is critical for rendering a page. It will not do Layout or Paint because it needs a Render Tree and Render Tree needs CSS Object Model.

The browser cannot generate Render Tree until it gets a full **Style Sheet**, we can't use partial style sheet, imagine a rule:

```
* {
  display: none !important;
}
```

at the end of the style.

> Browser is as quick as your slowest CSS file (or any critical resource)

So, we need to ensure that we deliver CSS to the browser, as quickly as possible.

Secondly, we don't need all megabytes to render the page. Initially we need to render CSS that really needs to render the pixels.

---
