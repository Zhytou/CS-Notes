# 基础

- [基础](#基础)
  - [运行环境](#运行环境)
  - [变量\&常量](#变量常量)
    - [类型](#类型)
    - [基础类型](#基础类型)
    - [引用类型](#引用类型)

## 运行环境

**front-end**:

firefox: monkey

chrome: v8

**back-end**:

node.js: a wrapper of  chrome v8 engine  

## 变量&常量

**变量**：

在Javascript中，可以使用let和var定义变量。其中，let定义的变量不可以跨块访问，而var定义的则可以（但不能跨函数访问）。

``` javascript
var i = 1;

//let is preferred
let name = 'zhy';
console.log(name);

// name rules are similar to c++ 
```

**常量**：

在Javascript中，const用来定义常量，使用时必须初始化(即必须赋值)，只能在块作用域里访问，而且不能修改。

``` javascript
const num = 1;
//the variable is constant
```

### 类型

JavaScript 中的数据类型可以分为两种类型：

- 基本数据类型（值类型）：字符串（String）、数字（Number）、布尔（Boolean）、空（Null）、未定义（Undefined）；
- 引用数据类型：对象（Object）、数组（Array）、函数（Function）。

此外，Javascript还是一种动态语言，即：变量与任何特定值类型没有任何关联，任何变量都可以被赋予（和重新赋予）各种类型的值。

### 基础类型

``` javascript
let name = 'zhy'; // String Literal
let age = 22;     // Number Literal
let male = true;  // Boolen Literal
let firstName = unfefined; 
let lastName = null; 
```

### 引用类型

**object**：

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

**array**：

``` javascript
let colors = ['red', 'blue'];
// the length can be changed
colors[2] = 'green';
// the array can store variables of different type
colors[3] = 1;
```

**function**：

``` javascript
function greet() {
    console.log('Hello World');
}
// no need for the ;
```
