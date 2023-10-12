# CSS 入门

- [CSS 入门](#css-入门)
  - [What is CSS?](#what-is-css)
  - [CSS Syntax](#css-syntax)
  - [CSS Selectors](#css-selectors)
  - [Selector Combination](#selector-combination)
  - [How to load CSS](#how-to-load-css)

## What is CSS?

- note a programming language
- styling language
- used for presentation

## CSS Syntax

```css
selector {
    property1: value;
    property2: value;
}
```

## CSS Selectors

- element selector

```css
h1 {
    property: value;
}
```

- class selector

```css
.classname {
    property: value;
}
```

- id selector

```css
#id {
    property: value;
}
```

Among the three selectors, id selector has the highest priority. while the class select has the second priority.

## Selector Combination

## How to load CSS

- inline: write css style in every element in html
- style element: write a style element in html header
- external css: use a link element to implement

```html
<link rel="stylesheet" href="style.css"></link>
```
