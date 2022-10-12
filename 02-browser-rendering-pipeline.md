# Browser rendering pipeline

## Steps browser takes to render a page

- Web browser sends an HTTP request to Web server
- The browser gets the response which is a pure HTML
- The browser sees that the is another file, the CSS
- Fetches that file

The browser now must process the HTML and build the **Document Object Model** **DOM**

Then, it parses the CSS and builds the **CSSOM**

It combines those to create the **Render tree**

The browser knows which elements to render on the page, and the styles to apply to them.

The next step is to calculate the position of each element > **Layout**

All of the relative measurement like:

- percentages
- ems
- rems

are converted to absolute units, the pixels.

Each element will be drawn on the picture one by one > **Paint**

However, your application can have multiple layers. The browser can paint each of the layers individually. The process of handling theses layers it's called > **Composite**

**JavaScript** > **Style** > **Layout** > **Paint** > **Composite**

---

## Style calculations

### HTML Parsing algorithm

- Conversion
  - raw bytes to individual characters
- Tokenizing
  - converts the string of characters into distinct tokens
  - StartTag:html StartTag:head ... EndTag:head StartTag:body ...
- Lexing
  - Converts tokens to nodes
    - html, head, body, p, span, Hello, user
- DOMTree
  - finally it creates the DOM
  - parent-child relationship

### Parse CSS

- Bytes
- Characters
- Tokens
- Nodes
- CSSOM

It contains all the styles of the page

### Render Tree

- Selector matching from the two trees
- Figure out final styles

as a result we get a new tree, the render tree, pretty similar to DOM tree but

- no html, head elements
- no `display: none` elements

Only elements that will be actually displayed on the screen will exist on the render tree.

---

## Layout

- Calculates the **Geometry** of the elements
- Position: x, y, width, height

The output of the layout is **the box model** of each element

Relative units are converted into pixels

Can inspect the Performance for the Layout calculation

---

## Paint

painting or rasterizing

```
drawTextBlob(X,Y,config)
drawTextBlob(X,Y,config)

drawRect(...)

drawImageRect()
```

Check paint profile > "Setting" > "Enable advanced paint instrumentation (slow)" > Profile page again > Open "Paint Profile" tab

There is a list of functions that paint the area

---

## Update Layer Tree

Can have multiple paints, because the browser can make multiple surfaces called layers.

The update layer tree is the process where the browser engine figures out what layers are needed for the page.

It analyzes the styles of the elements to figure out how many layers needs

---

## Composite Layers

Is the process where the painting parts of the page are put together for displaying on screen.

- Update Layer Tree

- Composite Layers

Settings (3 dots) > More tools > Layers

Can promote an element to each layer to improve runtime performance

---
