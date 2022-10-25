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

## Imports

It's an issue if you import a CSS from another CSS file:

e.g.:

`demo-css.css`

```
@import url('../dist/style.css);
```

because the second file is waiting to be downloaded.

You can confirm this by opening the network section.

Instead of using import, a better solution is to link CSS to HTML:

```
<link rel="stylesheet" href="demo-css/demo-css.css">
<link rel="stylesheet" href="dist/style.css">
```

and the browser will download them in parallel.

---

## Non Render Blocking CSS

What's the **minimum** amount of resources that we can ship to render the page?

It makes sense to split styles into multiple files:

```
<link href="style.css" rel="stylesheet">

<link href="print.css" rel="stylesheet"
  media="print">

<link href="mobile.css" rel="stylesheet"
  media="screen and (max-width: 768px)">

<link href="portrait.css" rel="stylesheet"
  media="all and (orientation: portrait)">
```

The browser assumes that it's style is render blocking. However, we can also tell the browser when the stylesheet will be applied.

Now, it's not render blocking anymore.

When we compare the different styles, we reduce the number of critical bytes. It still downloads all stylesheet, but it will not block rendering on print CSS, and the network looks much different.

The browser downloads multiple files in parallel.

---

## Unused Styles

Larger files take longer to download and longer to parse.

There are many online tools that may help us detect unused styles, but Chrome browser has it's own

Click on **More Tools > Coverage**

The **start recording**, as using the site.

In `styles.css`, the red section of the bar represents unused styles

Check percentage of **Unused Bytes**!

### Cutting down unused files

- More reliable
- Easier to maintain
- Simpler to debug
- Reduces the number of critical bytes
- Optimizes critical rendering path

Be careful with libraries like bootstrap, they come with many styles that probably you don't need.

You should never import a whole library if you don't use it.

All styles are downloaded and block the render of the page

Thing about if you really need a CSS library, and if you do make sure you import only those styles, that you really need because the file size can grow very fast.

---

## Base64

Instead of sending multiple HTTP requests for each icon you can get everything in one with CSS:

```
.icon.bicycle {
  background-image: url(data:image/png;base64,vsdgfsgahdsv)
}
```

We reduce the number of requests and both icons and styles arrive at the same time.

But with Base64 encoding we increase the style sheet file size.

Browser is loading CSS with heavy information, and CSS is render blocking.

> We turned hundreds of kilobytes of **non-blocking** resources into **blocking** ones!!!

Also it forces all assets bytes to be downloaded, even if they will never be used.

You should **avoid base64 encoding assets** in your CSS, instead of this, a better solution is:

- to use font icons
- or embedded SVG directly into HTML

---

## Minification and Gzip Compression

Network speed is the bottleneck of any web application. It's important to keep file size small, because nothing will render until all render blocking assets are downloaded.

### Minification removes

- Whitespaces
- Comments
- Non-required semicolons

CSS files are often larger from what they need to be:

`Input CSS`

```
/* headings */
h1 {
  color: red;
}
```

`Minified Output`

```
h1{color:red}
```

There are many ways to minify your styles

- gulp
- grunt
- webpack
- some frameworks
- online tools

We can get even better results with file compression:

### Gzip compression

**Gzip** is incredibly efficient at compressing repetitive strings

| Input CSS | Minified Output | Minified & Gzip |
| --------- | --------------- | --------------- |
| 187kb     | 156kb           | 20kb            |

> Netflix saw a 43% decrease in their bandwidth bill after turning on GZip"

---

## Optimizing Critical Resources and Critical Bytes Summary

**CSS is render blocking!** (by default)

The browser is as quick as your slowest render blocking file

### Reduce

- The number of critical resources
- The number of critical bytes

---
