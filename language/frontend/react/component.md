# React组件

- [React组件](#react组件)
  - [分类](#分类)
  - [JSX](#jsx)
    - [JSX规则](#jsx规则)
    - [在JSX中使用大括号传递变量](#在jsx中使用大括号传递变量)
  - [组件组合](#组件组合)
  - [Props](#props)
  - [State](#state)
  - [条件渲染](#条件渲染)

React组件是React应用中可重用的UI元素。它可以分为类组件或函数组件。

## 分类

**Functional Component**：

函数组件是一个返回JSX语法块的JavaScript函数。例如：

```javascript
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

其中，返回语句可以写成一行，例如：

```javascript
function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}
```

但是，如果你的标签和 return 关键字不在同一行，则必须把它包裹在一对括号中。

**Class Component**：

类组件与函数组件类似，也需要返回一个UI元素。但它必须继承自React.Component，例如：

```javascript
class Car extends React.Component {
  render() {
    return <h2>Hi, I am a Car!</h2>;
  }
}
```

不过目前类组件已经基本不再使用。

## JSX

JSX是JavaScript语法扩展，它能够让我们在JavaScript文件中书写类似HTML的标签，进而使得编写React组件更加直观和易于理解。

一般来说，网页是构建在HTML、CSS和JavaScript之上的。其中，内容存放在HTML中，样式放在CSS中，而逻辑则放在JavaScript中。

但随着Web的交互性越来越强，逻辑越来越决定页面中的内容。JavaScript逐渐开始大包大揽！这也是为什么在React中，渲染逻辑和标签共同存在于同一个地方——组件。

> 值得注意的是，尽管JSX和React经常一起使用，但它们是相互独立的。我们可以单独使用它们中的任意一个，JSX是一种语法扩展，而React则是一个JavaScript的库。

### JSX规则

- 只能返回一个根元素；
- 标签必须闭合；
- 使用驼峰式命名法给 所有大部分属性命名，比如：className等。

### 在JSX中使用大括号传递变量

在 JSX 中，只能在以下两种场景中使用大括号：

- 用作JSX标签内的文本： `<h1>{name}'s To Do List</h1>`是有效的，但是`<{tag}>Gregorio Y. Zara's To Do List</{tag}>`无效。
- 用作紧跟在 = 符号后的 属性：src={avatar} 会读取 avatar 变量，但是 src="{avatar}" 只会传一个字符串 {avatar}。

## 组件组合

React允许组件的嵌套和组合使用，从而构建复杂的UI。例如：

```javascript


```

组合使得我们可以将应用程序拆分成更小的部分，从而使代码更加可维护和可重用。

## Props

React组件使用props来互相通信。每个父组件都可以提供props给它的子组件，从而将一些信息传递给它。例如：

```javascript
function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

值得注意的是，在声明props时， 不要忘记 ( 和 ) 之间的一对花括号 { 和 }。这种语法被称为 “解构”，等价于于从函数参数中读取属性：

```javascript
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

## State

## 条件渲染

条件渲染是一种根据条件来渲染不同内容的技术。在React中，我们可以使用条件语句（如 if 和 switch）或三元运算符来实现条件渲染。条件渲染使得我们可以根据不同的情况来显示不同的内容，从而使我们的应用程序更加灵活和可定制。
