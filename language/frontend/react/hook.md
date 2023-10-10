# Hook

## State Hook

state 让一个组件 “记住”用户输入的信息，比如，一个表单组件可以使用 state 来存储输入值，而一个图片库组件可以使用 state 来存储选定的图片索引。

使用下面的 Hook 以向组件添加状态：

使用 useState 声明可以直接更新的 state 变量。
使用 useReducer 在 reducer 函数 中声明带有更新逻辑的 state 变量。

## Context Hook

context 允许组件 从祖先组件中接收信息，而不需要将其作为 props 传递。比如，app 的顶层组件可以将当前的 UI 主题传递给下面的所有组件，无论它们层级多深。

## Ref Hook

ref 允许组件 持有一些不用于渲染的信息，如 DOM 节点或 timeout ID。与 state 不同的是，更新 ref 并不会重新渲染组件。ref 是 React 范式的一个“规避机制”。当需要与非 React 系统如浏览器内置 API 一起工作时，ref 将会很有用。

使用 useRef 声明 ref。你可以在其中保存任何值，但最常见的是使用 ref 保存 DOM 节点。
使用 useImperativeHandle 自定义从组件中暴露的 ref，但是一般很少使用。

## Effect Hook

Effect 允许组件 连接到外部系统并与之同步。这包括处理网络、浏览器、DOM、动画、使用不同 UI 库编写的 widgets 以及其他非 React 代码。

useEffect 能够将组件连接到外部系统。

## Reference

[react docs](https://zh-hans.react.dev/reference/react)
