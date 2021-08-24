# JSX 简单入门教学指南

![JSX 简单入门教学指南](./images/reactjs.png)

## 前言

根据 [React](https://facebook.github.io/react/) 官方定义，React 是一个构建使用者界面的 JavaScritp Library。以 MVC 模式来说，ReactJS 主要是负责 View 的部分。过去一段时间，我们被灌输了许多前端分离的观念，在前端三兄弟中（或三姐妹、三剑客）：HTML 掌管内容结构、CSS 负责外观样式，JavaScript 主管逻辑互动，千万不要混在一块。然而，在 React 世界里，所有事物都是 以 Component 为基础，将同一个 Compoent 相关的程序和资源都放在一起，而在编写 React Component 时我们通常会使用 [JSX](https://facebook.github.io/jsx/) 的方式来提升程序编写效率。事实上，JSX 并非一种全新的语言，而是一种语法糖（[Syntatic Sugar](https://en.wikipedia.org/wiki/Syntactic_sugar)），一种语法类似 [XML](https://zh.wikipedia.org/wiki/XML) 的 ECMAScript 语法扩充。在 JSX 中 HTML 和组建这些元素标签的代码有紧密的关系。因此你可能要熟悉一下以 Component 为单位的思考方式（本文主要使用 ES6 语法）。

此外，React 和 JSX 的思维在于善用 JavaScript 的强大能力，放弃蹩脚的模版语言，这和 [Angular](https://angularjs.org/) 强化 HTML 的理念也有所不同。当然 JSX 并非强制使用，你也可以选择不用，因为最终 JSX 的内容会转化成 JavaScript（浏览器只看的懂 JavaScript）。不过等你阅读完接下来的内容，你或许会开始发现 JSX 的好，认真考虑使用 JSX 的语法。

## 一、使用 JSX 的好处

### 1. 提供更加语意化且易懂的标签

由于 JSX 类似 XML 的语法，让一些非开发人员也更容易看懂，且能精确定义包含属性的树状结构。一般来说我们想做一个反馈表单，使用 HTML 写法通常会长这样：

```html
<form class="messageBox">
  <textarea></textarea>
  <button type="submit"></button>
</form>
```

使用 JSX，就像 XML 语法结构一样可以自行定义标签且有开始和关闭，容易理解：

```js
<MessageBox />
```

React 思路认为使用 Component 比起模版（Template）和显示逻辑（Display Logic）更能实现关注点分离的概念，而搭配 JSX 可以实现声明式 `Declarative`（注重 what to），而非命令式  `Imperative`（注重 how to）的程序编写方式：

![Facebook 上面点赞功能](./images/fb_like.jpg)

以 Facebook 上面点赞功能来说，若是命令式 `Imperative` 写法大约会是长这样：

```js

if(userLikes()) {
  if(!hasBlueLike()) {
    removeGrayLike();
    addBlueLike();
  }
} else {
  if(hasBlueLike()) {
    removeBlueLike();
    addGrayLike();
  }
}

```

若是声明式 `Declarative` 则是会长这样：

```js
if(this.state.liked) {
  return (<BlueLike />);
} else {
  return (<GrayLike />);
}
```

看完上述说明是不是感觉 `React` 结合 `JSX` 的写法更易读易懂？事实上，当 Component 组成越来越复杂时，若使用 JSX 将可以让整个结构更加直观，可读性较高。

### 2. 更加简洁

虽然最终 JSX 会转换成 JavaScript，但使用 JSX 可以让程序看起来更加简洁，以下为使用 JSX 和不使用 JSX 的范例：

```html
<a href="https://facebook.github.io/react/">Hello!</a>
```

不使用 JSX（记得我们说过 JSX 是选用的）：

```js
// React.createElement(元件/HTML标签, 元件属性，以对象表示, 子元件)
React.createElement('a', {href: 'https://facebook.github.io/react/'}, 'Hello!')
```

### 3. 结合原生 JavaScript 语法

JSX 并非一种全新的语言，而是一种语法糖（Syntatic Sugar），一种语法类似 XML 的 ECMAScript 语法扩充，所以并没有改变 JavaScript 语意。通过结合 JavaScript ，可以释放 JavaScript 语言本身能力。下面例子就是运用 `map` 方法，轻易把 `result` 值迭代出来，产生无序清单（ul）的内容，不用再使用蹩脚的模版语言（这边有个小地方要留意的是每个 `<li>` 元素记得加上独特的 key 这边用 map function 迭代出的 index，不然会出现问题）：

```js
// const 为常数
const lists = ['JavaScript', 'Java', 'Node', 'Python'];

class HelloMessage extends React.Component {
  render() {
    return (
    <ul>
      {lists.map((result, index) => {
        return (<li key={index}>{result}</li>);
      })}
    </ul>);
  }
}
```

## 二、JSX 用法摘要

### 1. 环境设定与使用方式

初步了解为何要使用 JSX 后，我们来聊聊 JSX 的用法。一般而言 JSX 通常有两种使用方式：

1. 使用 [browserify](http://browserify.org/) 或 [webpack](https://webpack.github.io/) 等 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) bundler 并整合 [babel](https://babeljs.io/) 预处理
2. 于浏览器端做解析

在这边简单起见，我们先使用第二种方式，先让大家专注熟悉 JSX 语法使用，等到后面章节再教大家使用 bundler 的方式去做解析（可以试著把下面的原始码贴到 [JSbin](http://jsbin.com/) 的 HTML 看结果）：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <!-- 请先于 index.html 中引入 react.js, react-dom.js 和 babel-core 的 browser.min.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.0.1/react.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.0.1/react-dom.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      // 代码写在这边！
      ReactDOM.render(
        <h1>Hello, world!</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>
```

一般加载 JSX 方式有：

- 内嵌

```html
<script type="text/babel">
  ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('example')
  );
</script>
```

- 从外部引入

`<script type="text/jsx" src="main.jsx"></script>`

### 2. 标签用法

JSX 标签非常类似 XML ，可以直接书写。一般 Component 命名首字大写，HTML Tags 小写。以下是一个建立 Component 的 class：

```js
class HelloMessage extends React.Component {
  render() {
    return (
      <div>
        <p>Hello React!</p>
        <MessageList />
      </div>
    );
  }
}
```

### 3. 转换成 JavaScript

JSX 最终会转换成浏览器可以读取的 JavaScript，以下为其规则：

```js
React.createElement(
  string/ReactClass, // 表示 HTML 元素或是 React Component
  [object props], // 属性值，用对象表示
  [children] // 接下来参数皆为元素子元素
)
```

解析前（特别注意在 JSX 中使用 JavaScript 表达式时使用 `{}` 括起，如下方范例的 `text`，里面对应的是变数。若需希望放置一般文字，请加上 `''`）：

```js
var text = 'Hello React';
<h1>{text}</h1>
<h1>{'text'}</h1>
```

解析完后：

```js
var text = 'Hello React';
React.createElement("h1", null, "Hello React!");
```

另外要特别要注意的是由于 JSX 最终会转成 JavaScript 且每一个 JSX 节点都对应到一个 JavaScript 函数，所以在 Component 的 `render` 方法中只能回传一个根节点（Root Nodes）。例如：若有多个 `<div>` 要 `render` 请在外面包一个 Component 或 `<div>`、`<span>` 元素。

### 4. 注解

由于 JSX 最终会编译成 JavaScript，注解也一样使用 `//` 和 `/**/` 当做注解方式：

```js
// 单行注解

/*
  多行注解
*/

var content = (
  <List>
      {/* 若是在子元件注解要加 {}  */}
      <Item
        /* 多行
           注解
           喔 */
        name={window.isLoggedIn ? window.name : ''} // 单行注解
      />
  </List>
);
```

### 5. 属性

在 HTML 中，我们可以通过标签上的属性来改变标签外观样式，在 JSX 中也可以，但要注意 `class` 和 `for` 由于为 JavaScript 保留关键字用法，因此在 JSX 中使用 `className` 和 `htmlFor` 替代。

```js
class HelloMessage extends React.Component {
  render() {
    return (
      <div className="message">
        <p>Hello React!</p>
      </div>
    );
  }
}
```

#### Boolean 属性

在 JSX 中预设只有属性名称但没设值为 `true`，例如以下第一个 input 标签 `disabled` 虽然没设值，但结果和下面的 input 为相同：

```html
<input type="button" disabled />;
<input type="button" disabled={true} />;
```

反之，若是没有属性，则预设预设为 `false`：

```html
<input type="button" />;
<input type="button" disabled={false} />;
```

### 6. 扩展属性

在 ES6 中使用 `...` 是迭代对象的意思，可以把所有对象对应的值迭代出来设定属性，但要注意后面设定的属性会盖掉前面相同属性：

```js
var props = {
  style: "width:20px",
  className: "main",
  value: "yo",
}

<HelloMessage  {...props} value="yo" />

// 等于以下
React.createElement("h1", React._spread({}, props, {value: "yo"}), "Hello React!");

```

### 7. 自定义属性

若是希望使用自定义属性，可以使用 `data-`：

```js
<HelloMessage data-attr="xd" />
```

### 8. 显示 HTML

通常为了避免信息安全问题，我们会过滤掉 HTML，若需要显示的话可以使用：

```html
<div>{{_html: '<h1>Hello World!!</h1>'}}</div>
```

### 9. 样式使用

在 JSX 中使用外观样式方法如下，第一个 `{}` 是 JSX 语法，第二个为 JavaScript 对象。与一般属性值用 `-` 分隔不同，为驼峰式命名写法：

```js
<HelloMessage style={{ color: '#FFFFFF', fontSize: '30px'}} />
```

### 10. 事件处理

事件处理为前端开发的重头戏，在 JSX 中通过 inline 事件的绑定来监听并处理事件（注意也是驼峰式写法），更多事件处理方法请[参考官网](https://facebook.github.io/react/docs/events.html#supported-events)

```js
<HelloMessage onClick={this.onBtn} />
```

## 总结

以上就是 JSX 简明入门教学，希望通过以上介绍，让读者了解在 React 中为何要使用 JSX，以及 JSX 基本概念和用法。最后为大家复习一下：在 React 世界里，所有事物都是以 Component 为基础，通常会将同一个 Compoent 相关的程序和资源都放在一起，而在编写 React Component 时我们常会使用 [JSX](https://facebook.github.io/jsx/) 的方式来提升程序编写效率。JSX 是一种语法类似 XML 的 ECMAScript 语法扩充，可以善用 JavaScript 的强大能力，放弃蹩脚的模版语言。当然 JSX 并非强制使用，你也可以选择不用，因为最终 JSX 的内容会转化成 JavaScript。当相信阅读完上述的内容后，你会开始认真考虑使用 JSX 的语法。

## 延伸阅读

1. [Imperative programming or declarative programming](http://www.puritys.me/docs-blog/article-320-Imperative-programming-or-declarative-programming.html)
2. [JSX in Depth](https://facebook.github.io/react/docs/jsx-in-depth.html)
3. [从零开始学 React（ReactJS 101）](https://www.gitbook.com/book/kdchang/react101/details)

（image via [adweek](http://www.adweek.com/socialtimes/files/2014/05/LikeButtoniOSApps650.jpg), [codecondo](http://codecondo.com/wp-content/uploads/2015/12/Useful-Features-of-React_7851.png)）

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：ReactJS 与 Component 设计入门介绍](https://github.com/aclk/reactjs101/blob/master/Ch03/reactjs-introduction.md) | [下一章：Props、State、Refs 与表单处理](https://github.com/aclk/reactjs101/blob/master/Ch04/props-state-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
