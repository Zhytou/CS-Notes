# React环境

## 在HTML中使用React

使用React最简单的方法就是在HTML文件中直接编写React，不过要引入几个脚本。例如：

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  </head>
  <body>

    <div id="mydiv"></div>

    <script type="text/babel">
      function Hello() {
        return <h1>Hello World!</h1>;
      }

      ReactDOM.render(<Hello />, document.getElementById('mydiv'))
    </script>

  </body>
</html>
```

上面引入的三个脚本，前两个让我们能在JavaScript中编写React代码，第三个Babel允许我们在旧浏览器中编写JSX语法和ES6。

## React脚手架

```bash
# 运行此命令创建一个名为demo的React应用程序
npx create-react-app demo
```
