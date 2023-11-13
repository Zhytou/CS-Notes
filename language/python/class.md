# 类&实例

- [类\&实例](#类实例)
  - [属性](#属性)
  - [构造\&析构](#构造析构)
  - [方法](#方法)
  - [访问权限](#访问权限)
  - [继承](#继承)
  - [装饰器](#装饰器)

和其它面向对象语言类似，Python中也允许用户定义自己的类型。比如：

```python
class Student(object):
    '''定义学生类'''
    # 类属性
    schoolName = 'ZJU'
    studenCount = 0

    # 实例方法
    def __init__(self, name, age):
        self.name = name
        self.age = age
        studentCount += 1

    def displayStudent(self):
        print(f"Name: {self.name}, Age: {self.age}")

    # 类方法
    @classmethod
    def displaySchool(cls):
        print(f"School Name: {cls.schoolName}, Student Count: {cls.studentCount}")

    # 静态方法
    @staticmethod
    def displayWelcome():
        print(f"Hello!")
```

## 属性

**类的属性**：

类的属性就是在类中定义的变量，这个属性虽然归类所有，但类的所有实例都可以访问到。在类外，可以通过类和实例访问公有的属性；而对于私有属性，即任何以双下画线开头的名字，只能在该类中被调用，不能在外部调用或者继承，不能被子类继承，也不会被子类覆盖，

## 构造&析构

**\__init__**：

init 是初始化方法，用于设置实例的相关属性。在创建类的实例的时候一般会自动调用这个方法，对实例的属性进行初始化。init()方法的第一个参数永远都是self，表示创建实例本身，在 init 方法内部，可以把各种属性绑定到 self，因为 self指向创建实例本身。

**\__del__**：

## 方法

**类方法**：

类的方法由类调用，采用@classmethod 装饰，至少传入一个 cls（代指类本身，类似 self）参数。执行类方法时，自动将调用该方法的类赋值给 cls。

**静态方法**：

静态方法由类调用，无默认参数。将实例方法参数中的 self 去掉，在方

## 访问权限

在Python中，属性的访问权限是通过命名约定来实现的，而不是通过严格的访问控制机制。Python使用一种命名约定来表示属性的可访问性，其中以下划线开头的属性被视为私有属性，没有下划线开头的属性被视为公开属性。

需要注意的是，Python的访问控制约定是基于约定而不是强制的。私有属性仍然可以被访问和修改，但强烈建议不要在类的外部直接访问私有属性。这种约定是为了建议开发者遵循良好的编程实践，以便更好地组织和维护代码。

## 继承

事实上，Python的继承机制十分类似Javascript，它们都是基于原型链的继承(Prototype-Based Inheritance)。但不同的是，Python使用类定义对象模板，而Javascript则使用函数。或者换一句话说，Javascript的类模板就是函数，比如：下面这段Javascript代码定义了一个名为Shape的类，其中还有一个move成员函数。

``` javascript
function Shape() {}

Shape.prototype.move = function() {
  console.log('move');
}

let a = new Shape();
a.move(); 
```

而当我有一个名为Rectangle类希望继承于Shape类时，

``` javascript
function Rectangle() {
  Shape.call(this); // call super constructor.
}
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

let b = new Rectangle();
b.move();
```

同样的两个类使用Python实现如下：

```python
class Shape:
  def __init__(self):
    # ....

  def move(self):
    print('move')
  
class Rectangle(Shape):
  def __init__(self):
    super().__init__()
```

## 装饰器

Python装饰器是一种语法糖，它的本质是将被修饰的函数作为参数，传入另一个函数
