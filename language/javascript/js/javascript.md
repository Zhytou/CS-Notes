# Js 入门

> [JavaScript Tutorial for Beginners: Learn JavaScript in 1 Hour](https://www.youtube.com/watch?v=W6NZfCO5SIk)

## Javascript Engine

**front-end**:

firefox: monkey

chrome: v8

**back-end**:

node.js: a wrapper of  chrome v8 engine  

## Env

**editor**:

VScode + Live browser

**package control** :

npm

**runtime env**:

node

## Semantics

### .html & .js

``` javascript
// in index.html
<script src = "index.js"> </script>

// in index.js
console.log("hello world");
```

### variable

``` javascript
var i = 1;
//let is preferred

let name = 'zhy';
console.log(name);

// name rules are similar to c++ 
```

### const

``` javascript
const num = 1;
//the variable is constant
```

### primitive type

``` javascript
let name = 'zhy'; // String Literal
let age = 22;     // Number Literal
let male = true;  // Boolen Literal
let firstName = unfefined; 
let lastName = null; 
```

### dynamic typing

javascript is a dynamic language, which means the type of variables can be changed at runtime.

### reference type

Variables in javascript are divided into two groups. One of them  is primitive type also known as value type, which is passed by value, while the other one is known as reference type, which is passed by reference.

### object

``` javascript
// reference type includes object, function and array.
let person = {
    name: 'zhy',
    age: 22
};

//Dot Notation
person.name = 'zhy';

//Bracket Notation
person['name'] = 'zhy';
```

### array

``` javascript
let colors = ['red', 'blue'];
// the length can be changed
colors[2] = 'green';
// the array can store variables of different type
colors[3] = 1;
```

### function

``` javascript
function greet() {
    console.log('Hello World');
}
// no need for the ;
```

## other

### where is the best place to insert script tag

Here's what happens when a browser loads a website with a `<script>` tag on it:

1. Fetch the HTML page (e.g. *index.html*)
2. Begin parsing the HTML
3. The parser encounters a `<script>` tag referencing an external script file.
4. The browser requests the script file. Meanwhile, the parser blocks and stops parsing the other HTML on your page.
5. After some time the script is downloaded and subsequently executed.
6. The parser continues parsing the rest of the HTML document.

step#4 causes a bad user experience. Your website basically stops loading until you've downloaded all scripts. If there's one thing that users hate it's waiting for a website to load.

#### unrecommended old approach

The old approach to solving this problem was to put `<script>` tags at the bottom of your `<body>`, because this ensures the parser isn't blocked until the very end.

#### modern approach

Today, browsers support the `async` and `defer` attributes on scripts. These attributes tell the browser it's safe to continue parsing while the scripts are being downloaded.

**async vs defer**:

If the `async` attribute is set, the script is downloaded in parallel to parsing the page, and executed as soon as it is available. The parsing of the page is interrupted once the script is downloaded completely, and then the script is executed, before the parsing of the rest of the page continues.

If the `defer` attribute is set, it specifies that the script is downloaded in parallel to parsing the page, and executed after the page has finished parsing.

**code example**:

``` javascript
<script src="path/to/script1.js" async></script>
<script src="path/to/script1.js" defer></script>
```

### how to get event param in js addeventlistener

``` javascript
newImage.addEventListener('click', function(e) {
    displayedImage.src = e.target.src;
});
```

### how to use rgb color in js

``` javascript
btn.onclick = function() {
    if (btn.className == 'dark') {
        btn.className = 'light';
        btn.innerText = 'Lighten';
        overlay.style['background-color'] = "rgba(0,0,0,0.5)";
    } else {
        btn.className = 'dark';
        btn.innerText = 'Darken';
        overlay.style['background-color'] = "rgba(0,0,0,0)";
    }
}
```

## References

+ [stackoverflow - where is the best place to insert script tag](https://stackoverflow.com/questions/436411/where-should-i-put-script-tags-in-html-markup)
+ [stackoverflow - how to get event param in js addeventlistener](https://stackoverflow.com/questions/7394750/adding-event-as-parameter-within-function-using-addeventlistener-doesnt-work)
+ [javascript overview](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Language_Overview)