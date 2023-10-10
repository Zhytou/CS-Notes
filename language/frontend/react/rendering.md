# 渲染

- [渲染](#渲染)
  - [React的原理](#react的原理)
  - [组件生命周期](#组件生命周期)
  - [响应事件](#响应事件)
    - [事件处理函数](#事件处理函数)
    - [事件传播](#事件传播)
  - [ref](#ref)
  - [参考](#参考)

## React的原理

**JSX语法糖**：

## 组件生命周期

每个 React 组件都经历相同的生命周期：

- 当组件被添加到屏幕上时，它会进行组件的挂载。
- 当组件接收到新的props或state时，通常是作为对交互的响应，它会进行组件的更新。
- 当组件从屏幕上移除时，它会进行组件的卸载。

## 响应事件

### 事件处理函数

使用React可以在JSX中添加事件处理函数。比如，内置组件（`<button>`和`<div>`）的onClick，或者自定义组件的props。其中事件处理函数为自定义函数，它将在响应交互（如点击、悬停、表单输入框获得焦点等）时触发。例如：

```javascript
export default function Button() {
  function handleClick() {
    alert('你点击了我！');
  }

  return (
    <button onClick={handleClick}>
      点我
    </button>
  );
}
```

此外，当事件处理函数声明于组件内部，它们还可以直接访问组件的props。例如：

```javascript
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="正在播放！">
        播放电影
      </AlertButton>
      <AlertButton message="正在上传！">
        上传图片
      </AlertButton>
    </div>
  );
}
```

事件处理函数还能在父组件中定义，并利用props传递。例如：

```javascript
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`正在播放 ${movieName}！`);
  }

  return (
    <Button onClick={handlePlayClick}>
      播放 "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('正在上传！')}>
      上传图片
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="魔女宅急便" />
      <UploadButton />
    </div>
  );
}
```

### 事件传播

事件处理函数还将捕获任何来自子组件的事件。通常，我们会说事件会沿着树向上“冒泡”或“传播”：它从事件发生的地方开始，然后沿着树向上传播。

事件处理函数接收一个事件对象作为唯一的参数。按照惯例，它通常被称为e，代表 “event”（事件）。我们可以在子组件中调用e.stopPropagation()来阻止一个事件到达父组件

## ref

## 参考

[How React works under the hood](https://www.freecodecamp.org/news/react-under-the-hood/)
