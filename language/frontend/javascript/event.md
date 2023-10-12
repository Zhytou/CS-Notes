# 事件Event

## 概述

事件是您在编程时系统内发生的动作或者发生的事情，系统响应事件后，如果需要，您可以某种方式对事件做出回应。

每个可用的事件都会有一个**事件处理器**，也就是事件触发时会运行的代码块。当我们定义了一个用来回应事件被激发的代码块的时候，我们说我们**注册了一个事件处理器**。

注意事件处理器有时候被叫做**事件监听器**——从我们的用意来看这两个名字是相同的，尽管严格地来说这块代码既监听也处理事件。监听器留意事件是否发生，然后处理器就是对事件发生做出的回应。

## 使用

### 事件处理器属性

``` javascript
const btn = document.querySelector('button');

btn.onclick = function() {
  const rndCol = 'rgb(' + random(255) + ',' + random(255) + ',' + random(255) + ')';
  document.body.style.backgroundColor = rndCol;
}
```

这个 `onclick` 是被用在这个情景下的事件处理器的属性，它就像 button 其他的属性（如 `btn.textContent`, or `btn.style`), 但是有一个特别的地方——当您将一些代码赋值给它的时候，只要事件触发代码就会运行。

一些事件非常通用，几乎在任何地方都可以用（比如 onclick 几乎可以用在几乎每一个元素上），然而另一些元素就只能在特定场景下使用，比如我们只能在 video 元素上使用 [onplay](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onplay) 。

### 行内事件处理器 —— 不要使用

``` javascript
<button onclick="bgChange()">Press me</button>

function bgChange() {
  const rndCol = 'rgb(' + random(255) + ',' + random(255) + ',' + random(255) + ')';
  document.body.style.backgroundColor = rndCol;
}
```

在Web上注册事件处理程序的最早方法是类似于上面所示的**事件处理程序HTML属性**(也称为内联事件处理程序)—属性值实际上是当事件发生时要运行的JavaScript代码。

### addEventListener()和removeEventListener()

新的事件触发机制被定义在 [Document Object Model (DOM) Level 2 Events](https://www.w3.org/TR/DOM-Level-2-Events/) Specification, 这个细则给浏览器提供了一个函数 — `addEventListener()`。这个函数和事件处理属性是类似的，但是语法略有不同。我们可以重写上面的随机颜色背景代码：

``` javascript
const btn = document.querySelector('button');

function bgChange() {
  const rndCol = 'rgb(' + random(255) + ',' + random(255) + ',' + random(255) + ')';
  document.body.style.backgroundColor = rndCol;
}

btn.addEventListener('click', bgChange);
```

这个机制带来了一些相较于旧方式的优点。有一个相对应的方法，`removeEventListener()，`这个方法移除事件监听器。例如，下面的代码将会移除上个代码块中的事件监听器：

``` javascript
btn.removeEventListener('click', bgChange);
```

## 参考References

+ [MDN - 事件介绍](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Building_blocks/Events)
