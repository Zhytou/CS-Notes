# 函数

- [函数](#函数)
  - [箭头函数](#箭头函数)
  - [自定义对象](#自定义对象)

学习 JavaScript 最重要的就是要理解对象和函数两个部分。最简单的函数就像下面这个这么简单：

```javascript
function add(x, y) {
  var total = x + y;
  return total;
}
```

## 箭头函数

```javascript
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
//相当于：(param1, param2, …, paramN) =>{ return expression; }

// 当只有一个参数时，圆括号是可选的：
(singleParam) => { statements }
singleParam => { statements }

// 没有参数的函数应该写成一对圆括号。
() => { statements }

//加括号的函数体返回对象字面量表达式：
params => ({foo: bar})

//支持剩余参数和默认参数
(param1, param2, ...rest) => { statements }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => {
statements }
```

## 自定义对象

与 C++ 和 Java 不同，JavaScript 是一种基于原型的编程语言，并没有 class 语句，而是把函数用作类。例如，定义一个人名对象，这个对象包括人的姓和名两个域。

```javascript
function makePerson(first, last) {
  return {
    first: first,
    last: last,
    fullName: function () {
      return this.first + " " + this.last;
    },
    fullNameReversed: function () {
      return this.last + ", " + this.first;
    },
  };
}
```

这种写法会让我们每次创建一个Person对象的时候，都在其中创建了两个新的函数对象——如果这个代码可以共享不是更好吗？

```javascript
function personFullName() {
  return this.first + " " + this.last;
}
function personFullNameReversed() {
  return this.last + ", " + this.first;
}
function Person(first, last) {
  this.first = first;
  this.last = last;
  this.fullName = personFullName;
  this.fullNameReversed = personFullNameReversed;
}
```

这种写法的好处是，我们只需要创建一次方法函数，在构造函数中引用它们。那是否还有更好的方法呢？

```javascript
function Person(first, last) {
  this.first = first;
  this.last = last;
}
Person.prototype.fullName = function () {
  return this.first + " " + this.last;
};
Person.prototype.fullNameReversed = function () {
  return this.last + ", " + this.first;
};
```

Person.prototype 是一个可以被 Person 的所有实例共享的对象。它是一个名叫原型链（prototype chain）的查询链的一部分：当你试图访问 Person 某个实例（例如上个例子中的 s）一个没有定义的属性时，解释器会首先检查这个 Person.prototype 来判断是否存在这样一个属性。所以，任何分配给 Person.prototype 的东西对通过 this 对象构造的实例都是可用的。
