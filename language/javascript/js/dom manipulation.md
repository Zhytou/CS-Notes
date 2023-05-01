# Document Object Model

## Introduction

### DOM

The DOM is a W3C (World Wide Web Consortium) standard.

The DOM defines a standard for accessing documents.

The W3C DOM standard is separated into 3 different parts:

- Core DOM - standard model for all document types
- XML DOM - standard model for XML documents
- HTML DOM - standard model for HTML documents

### HTML DOM

The HTML DOM is a standard **object** model and **programming interface** for HTML. It defines:

- The HTML elements as **objects**
- The **properties** of all HTML elements
- The **methods** to access all HTML elements
- The **events** for all HTML elements

In other words: **The HTML DOM is a standard for how to get, change, add, or delete HTML elements.**

### Method & Property

HTML DOM methods are **actions** you can perform (on HTML Elements).

HTML DOM properties are **values** (of HTML Elements) that you can set or change.

### Object or Element

In the DOM, all HTML elements are defined as **objects**.

## Method

### Finding HTML Element

| Method                                 | Description                   |
| -------------------------------------- | ----------------------------- |
| `document.getElementById(id)`          | Find an element by element id |
| `document.getElementByTagName(name)`   | Find elements by tag name     |
| `document.getELementByClassName(name)` | Find elements by class name   |

### Adding and Deleting HTML Elements

| Method                            | Description                       |
| --------------------------------- | --------------------------------- |
| `document.createElement(element)` | Create an HTML element            |
| `document.removeChild(element)`   | Remove an HTML element            |
| `document.appendChild(element)`   | Add an HTML element               |
| `document.replaceChild(new,old)`  | Replace an HTML element           |
| `document.write(text)`            | Write into the HTML output stream |

## Property

### Changing HTML Elements

| Property                               | Description                                   |
| -------------------------------------- | --------------------------------------------- |
| `element.innerHTML = new html content` | Change the inner HTML of an element           |
| `element.attribute = new value`        | Change the attribute value of an HTML element |
| `element.style.property = new style`   | Change the style of an HTML element           |

## Document

The HTML DOM document object is the owner of all other objects in your web page.

## References

[w3school - DOM](https://www.w3schools.com/js/js_htmldom.asp)
