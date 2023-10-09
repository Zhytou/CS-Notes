# React Component

- [React Component](#react-component)
  - [Functional Component](#functional-component)
  - [JSX](#jsx)
  - [Props vs State](#props-vs-state)
  - [Conditional Rendering](#conditional-rendering)
  - [Composition](#composition)

React Component 是 React 应用程序的基本构建块。它们是可重用的代码块，可以接受输入（称为“props”）并返回要在页面上显示的元素。React Component 可以是类组件或函数组件。

## Functional Component

函数组件是一种更简单的组件形式，它们只是一个接受 props 并返回 React 元素的函数。函数组件没有状态（state），也没有生命周期方法。它们通常用于只需要渲染静态内容的组件。

## JSX

JSX 是一种 JavaScript 语法扩展，它允许我们在 JavaScript 中编写类似 HTML 的代码。JSX 使得编写 React 组件更加直观和易于理解。

## Props vs State

Props 和 State 是 React 组件中两个重要的概念。Props 是组件的输入，它们是从父组件传递给子组件的数据。State 是组件的内部状态，它们是组件自己管理的数据。当组件的 props 或 state 发生变化时，React 会自动重新渲染组件。

## Conditional Rendering

条件渲染是一种根据条件来渲染不同内容的技术。在 React 中，我们可以使用条件语句（如 if 和 switch）或三元运算符来实现条件渲染。条件渲染使得我们可以根据不同的情况来显示不同的内容，从而使我们的应用程序更加灵活和可定制。

## Composition

Composition 是一种将多个小组件组合成一个大组件的技术。在 React 中，我们可以使用组合来构建复杂的 UI。组合使得我们可以将应用程序拆分成更小的部分，从而使代码更加可维护和可重用。

总之，React Component 是 React 应用程序的基本构建块。函数组件是一种更简单的组件形式，JSX 使得编写 React 组件更加直观和易于理解。Props 和 State 是 React 组件中两个重要的概念，条件渲染和 Composition 是构建复杂 UI 的重要技术。
