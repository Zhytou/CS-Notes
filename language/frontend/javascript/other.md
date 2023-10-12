# 杂项

- [杂项](#杂项)
  - [遍历](#遍历)
    - [基本功能](#基本功能)
    - [可链接其他方法](#可链接其他方法)
    - [可变性](#可变性)
  - [查找和过滤](#查找和过滤)
  - [累加器](#累加器)
  - [解构赋值](#解构赋值)
    - [数组解构](#数组解构)
    - [对象解构](#对象解构)
  - [展开运算符](#展开运算符)

## 遍历

除了常规for或者while这样的流程控制语句之外，JavaScript还提供了一些方便的方法帮助我们遍历数组。最常用于迭代的两个是 Array.prototype.map()和Array.prototype.forEach()。

### 基本功能

map()方法接收一个函数作为参数。然后它将参数应用于每个元素并返回一个全新的数组，其中填充了调用提供的函数的结果。这意味着它返回一个新数组，其中包含数组每个元素的执行结果。它将始终返回相同数量的项目。例如，下面这段代码就返回了所有元素的平方。

```javascript
const nums = [5, 4, 3, 2, 1]
nums.map(num => num * num)
```

与map()一样，forEach()方法接收一个函数作为参数，并对每个数组元素执行一次。但是，它不会返回像map()这样的新数组，而是返回 undefined。例如，下面这段代码就将数组所有元素的全部打印。

```javascript
const nums = [5, 4, 3, 2, 1]
nums.forEach(num => console.log(num))
```

### 可链接其他方法

由于对数组执行map()方法后会返回一个全新的数组，这就意味着可以在map()方法后附加reduce()、sort()、filter()等。而forEach()则不能。例如：

```javascript
const nums = [1, 2, 3, 4, 5]
nums.forEach(x => x * x).reduce((total, value) => total + value)
```

### 可变性

map()不会修改原数组，而forEach()则可能会。

## 查找和过滤

Array.prototype.filter()和Array.prototype.find()是两个常常让人混淆的函数。其中，前者是返回由满足回调函数中指定条件的所有元素构成的一个新数组；而后者则是返回满足回调函数中指定条件的第一个元素。例如：

```javascript
const x = [1, 2, 3, 4, 5];

const y = x.filter(el => el*2 === 2);
console.log("y is: ", y); // y is: [1]

const y2 = x.find(el => el*2 === 2);
console.log("y2 is: ", y2); // y2 is: 1
```

## 累加器

Array.prototype.reduce()方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。

## 解构赋值

解构赋值语法是一种 Javascript 表达式。可以将数组中的值或对象的属性取出，赋值给其他变量。例如：

```javascript
let introduction = ["Hello", "I" , "am", "Sarah"];
let [greeting, pronoun] = introduction;

console.log(greeting);//"Hello"
console.log(pronoun);//"I"
```

### 数组解构

如果我们想获取数组中的第一项和最后一项，而不是第一项和第二项，并且只想分配两个变量，该怎么办？ 这也可以实现。看下面的例子：

```javascript
let [greeting,,,name] = ["Hello", "I" , "am", "Sarah"];

console.log(greeting);//"Hello"
console.log(name);//"Sarah"
```

如果我们想将数组中的某些项赋值给一些变量，而将数组中的其余项分配给一个特定的变量，该怎么做呢？在这种情况下，我们可以这样做：

```javascript
let [greeting,...intro] = ["Hello", "I" , "am", "Sarah"];

console.log(greeting);//"Hello"
console.log(intro);//["I", "am", "Sarah"]
```

此外，我们可以为变量分配默认值，以防止从数组中提取的值是 undefined：

```javascript
let [greeting = "hi",name = "Sarah"] = ["hello"];

console.log(greeting);//"Hello"
console.log(name);//"Sarah"
```

### 对象解构

除了数组能被解构赋值外，对象也支持这些操作，例如：

```javascript
let {name, country, job} = {name: "Sarah", country: "Nigeria", job: "Developer"};

console.log(name);//"Sarah"
console.log(country);//"Nigeria"
console.log(job);//Developer"
```

如果要将对象的值分配给新变量，而不是使用属性名称，则可以执行以下操作：

```javascript
let {name: foo, job: bar} = {name: "Sarah", country: "Nigeria", job: "Developer"};;

console.log(foo);//"Sarah"
console.log(bar);//"Developer"
```

## 展开运算符

展开运算符允许⼀个表达式在某处展开。展开运算符在多个参数（⽤于函数调⽤）或多个元素（⽤于数组字⾯量）或者多个变量（⽤于解构赋值）的地⽅可以使⽤。
