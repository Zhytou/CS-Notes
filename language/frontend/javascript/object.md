# 对象

- [对象](#对象)
  - [方法](#方法)
  - [使用](#使用)
  - [构造函数](#构造函数)
  - [对象原型](#对象原型)
    - [修改原型](#修改原型)

**概念**：

对象是一个包含相关数据和方法的集合（通常由一些变量和函数组成，我们称之为对象里面的属性和方法）。

准确的说，JavaScript的对象是键值对的容器。这种键值对容器写法，类似Python 中的字典或C++ 语言中的哈希表。

**例子**：

```javascript
var person = {
  name : ['Bob', 'Smith'],
  age : 32,
  gender : 'male',
  interests : ['music', 'skiing'],
  bio : function() {
    alert(this.name[0] + ' ' + this.name[1] + ' is ' + this.age + ' years old. He likes ' + this.interests[0] + ' and ' + this.interests[1] + '.');
  },
  greeting: function() {
    alert('Hi! I\'m ' + this.name[0] + '.');
  }
};
```

## 方法

在JavaScript中可以用`this`直接定义方法，也可以将方法定义在`prototype`属性中。

JavaScript存在原型链的特点使得第二种定义对象方法的方式称为可能，因为对象会不断沿原型链向上搜索直到找到所需要的方法或者`prototype`属性为`undefined`。

这两种定义对象方法的不同之处，具体可以参考 [stackoverflow的这个回答](https://stackoverflow.com/questions/310870/use-of-prototype-vs-this-in-javascript)。简单来说，就是按照第二种定义的方法没有被复制到其他对象——它们被访问需要通过前面所说的“原型链”的方式。

用`this`直接定义方法：

```javascript
function Ball(x, y) {
    this.moveTo = function (dx, dy) {
        //do something
    };
};
```

将方法定义在`prototype`属性中：

```javascript
function Ball(x, y) {
};
Ball.prototype.moveTo = function(dx, dy) {
    //do something
};
```

## 使用

**点表示法**：

JavaScript运行使用点表示法(dot notation)来访问对象的属性和方法。

```javascript
person.age
person.interests[1]
person.bio()
```

**括号表示法**：

另外一种访问属性的方式是使用括号表示法(bracket notation)来访问对象的属性和方法。

```javascript
person['age']
person['name']['first']
```

## 构造函数

**definition**：

有些人认为 JavaScript 不是真正的面向对象的语言，比如它没有像许多面向对象的语言一样有用于创建class类的声明。JavaScript 用一种称为**构建函数**的特殊函数来定义对象和它们的特征。

构建函数非常有用，因为很多情况下您不知道实际需要多少个对象（实例）。**构建函数**提供了创建您所需对象（实例）的有效方法，将对象的数据和特征函数按需联结至相应对象。

**code**：

```javascript
function Person(first, last, age, gender, interests) {
  this.name = {
    'first': first,
    'last': last
  };
  this.age = age;
  this.gender = gender;
  this.interests = interests;
  this.bio = function() {
    alert(this.name.first + ' ' + this.name.last + ' is ' + this.age + ' years old. He likes ' + this.interests[0] + ' and ' + this.interests[1] + '.');
  };
  this.greeting = function() {
    alert('Hi! I\'m ' + this.name.first + '.');
  };
};
```

## 对象原型

JavaScript 常被描述为一种**基于原型的语言 (prototype-based language)**——每个对象拥有一个**原型对象**，对象以其原型为模板、从原型继承方法和属性。

原型对象也可能拥有原型，并从中继承方法和属性，一层一层、以此类推。这种关系常被称为**原型链 (prototype chain)**，它解释了为何一个对象会拥有定义在其他对象中的属性和方法。

![对象原型](https://image-static.segmentfault.com/414/998/4149982343-58e5a767abec1_fix732)

`prototype` 属性的值是一个对象，我们希望被原型链下游的对象继承的属性和方法，都被储存在其中。

**使用Object.create()实现继承**：

```javascript
// Shape - 父类(superclass)
function Shape() {
  this.x = 0;
  this.y = 0;
}

// 父类的方法
Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};

// Rectangle - 子类(subclass)
function Rectangle() {
  Shape.call(this); // call super constructor.
}

// 子类续承父类
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();

console.log('Is rect an instance of Rectangle?',
  rect instanceof Rectangle); // true
console.log('Is rect an instance of Shape?',
  rect instanceof Shape); // true
rect.move(1, 1); // Outputs, 'Shape moved.'
```

### 修改原型

**原型链自动更新**：

在修改原型前创建的对象实例，也可以调用新加入原型的函数。即：有对象实例的可用功能被自动更新了。

这证明了先前描述的原型链模型——上游对象的方法不会复制到下游的对象实例中；下游对象本身虽然没有定义这些方法，但浏览器会通过上溯原型链、从上游对象中找到它们。

**在原型上定义方法**：

在构造器（函数体）中定义属性、在 `prototype` 属性上定义方法。
