# 异步

- [异步](#异步)
  - [回调](#回调)
  - [承诺](#承诺)
    - [状态](#状态)
    - [流程](#流程)
    - [优点](#优点)

## 回调

`callback`其实就是函数，只不过是作为参数传递给那些在后台执行的其他函数。当那些后台函数运行结束，就调用`callback`函数，通知你工作已经完成或者做出某些反映。因此`callback`函数也被称为回调函数。
函数`addEventListener()`第二个参数就是一个回调函数。

```javascript
btn.addEventListener('click', () => {
  alert('You clicked me!');

  let pElem = document.createElement('p');
  pElem.textContent = 'This is a newly-added paragraph.';
  document.body.appendChild(pElem);
});
```

## 承诺

promise是表示异步操作完成或失败的对象。可以说，它代表了一种中间状态。 本质上，这是浏览器说“我保证尽快给您答复”的方式，因此得名“promise”。

### 状态

- 创建promise时，它既不是成功也不是失败状态。这个状态叫作**pending**（待定）。
- 当promise返回时，称为 **resolved**（已解决）.
  - 一个成功**resolved**的promise称为**fullfilled**（**实现**）。它返回一个值，可以通过将`.then()`块链接到promise链的末尾来访问该值。 `.then()`块中的执行程序函数将包含promise的返回值。
  - 一个不成功**resolved**的promise被称为**rejected**（**拒绝**）了。它返回一个原因（**reason**），一条错误消息，说明为什么拒绝promise。可以通过将`.catch()`块链接到promise链的末尾来访问此原因。

### 流程

像promise这样的异步操作被放入事件队列中，事件队列在主线程完成处理后运行，这样它们就不会阻止后续JavaScript代码的运行。排队操作将尽快完成，然后将结果返回到JavaScript环境。例如：

```javascript
fetch('products.json').then(function(response) {
  return response.json();
}).then(function(json) {
  products = json;
  initialize();
}).catch(function(err) {
  console.log('Fetch problem: ' + err.message);
});
```

- 上述代码两个`.then()`块，都分别包含一个回调函数。当`fecth`或上一个`.then()`块结束后，返回一个 `promise`，接着到下一个块，从而依次执行多个异步操作。
- 当其中任何一个`.then()`块执行失败，就会跳转执行`.catch()`块。
- `.then()`块的工作方式类似于使`AddEventListener()`向对象添加事件侦听器时的方式。它不会在事件发生之前运行（当promise履行时）。最显着的区别是`.then()`每次使用时只运行一次，而事件监听器可以多次调用。

### 优点

- 您可以使用多个then()操作将多个异步操作链接在一起，并将其中一个操作的结果作为输入传递给下一个操作。这种链接方式对回调来说要难得多，会使回调以混乱的“末日金字塔”告终。
- Promise总是严格按照它们放置在事件队列中的顺序调用。
- 错误处理要好得多——所有的错误都由块末尾的一个.catch()块处理，而不是在“金字塔”的每一层单独处理。

```javascript
//老式callback
chooseToppings(function(toppings) {
  placeOrder(toppings, function(order) {
    collectOrder(order, function(pizza) {
      eatPizza(pizza);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
//新式promise
chooseToppings()
.then(toppings => placeOrder(toppings))
.then(order => collectOrder(order))
.then(pizza => eatPizza(pizza))
.catch(failureCallback);
```
