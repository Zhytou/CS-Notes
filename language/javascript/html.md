# html 入门

> [Youtube - HTML Tutorial for Beginners](https://www.youtube.com/watch?v=qz0aGYrrlhU)

## header section

``` html
<!DOCTYPE html>
<html>
    <head>
        <title></title>
    </head>
    <body>
        
    </body>
</html>
```

First meta defines character set(e.g. ASCII). In this case, the character set is UTF-8. While the second meta element defines the viewport which is the range that user can see when they first come to the webpage.

These two meta element are the basic information you should at least define for the html.

## text

| tag       | meaning       | function                                                     |
| --------- | ------------- | ------------------------------------------------------------ |
| `<p>`       | paragraph     | simple text                                                  |
| `<em>`      | emphasis      | highlight                                                    |
| `<strong>`  | also emphasis | highlight                                                    |
| `<h1>~<h6>` | heading       | heading is never about font-size, it should be used to represent hierarchy. |

## entity

| entity  | symbol |
| ------- | ------ |
| \&lt;   | <      |
| \&gt;   | >      |
| \&copy; | &copy; |

## link

The `<a>` tag defines a hyperlink, which is used to link from one page to another.

The most important attribute of the `<a>` element is the `href` attribute, which indicates the link's destination.

By default, links will appear as follows in all browsers:

- An unvisited link is underlined and blue
- A visited link is underlined and purple
- An active link is underlined and red

## img

The `<img>` tag is used to embed an image in an HTML page.

Images are not technically inserted into a web page; images are linked to web pages. The `<img>` tag creates a holding space for the referenced image.

The `<img>` tag has two required attributes:

- src - Specifies the path to the image
- alt - Specifies an alternate text for the image, if the image for some reason cannot be displayed

## class

The class is an attribute which specifies one or more class names for an HTML element.

The class attribute can be used on any HTML element.

The class name can be used by CSS and JavaScript to perform certain tasks for elements with the specified class name.

## id

The `id` attribute specifies a unique id for an HTML element. The value of the `id` attribute must be unique within the HTML document.

The `id` attribute is used to point to a specific style declaration in a style sheet. It is also used by JavaScript to access and manipulate the element with the specific id.

The syntax for id is: write a hash character (#), followed by an id name. Then, define the CSS properties within curly braces {}.

In the following example we have an `<h1>` element that points to the id name "myHeader". This `<h1>` element will be styled according to the `#myHeader` style definition in the head section:

**id vs class**:

A class name can be used by multiple HTML elements, while an id name must only be used by one HTML element within the page
