# ReactJS 与 Component 设计入门介绍

## 前言

在上一个章节中我们快速学习了 React 开发环境建置和 Webpack 入门。接下来我们将更进一步了解 React 和 Component 设计时需注意的几个重要特性。

## ReactJS 特性简介

React 原本是 Facebook 自己内部使用的开发工具，但却是一个目标远大的一个项目：`Learn once, write anywhere`。自从 2013 年开源后周边的生态系更是蓬勃发展。ReactJS 的出现让前端开发有许多革新性的思维出现，其中有几个重要特性值得我们去探讨：

1. 基于组件（Component）化思考
2. 用 JSX 进行声明式（Declarative）UI 设计
3. 使用 Virtual DOM
4. Component PropType 错误校对机制
5. Component 就像个状态机（State Machine），而且也有生命周期（Life Cycle）
6. 一律重绘（Always Redraw）和单向数据流（Unidirectional Data Flow）
7. 在 JavaScript 里写 CSS：Inline Style

## 基于组件（Component）化思考

![ReactJS 与 Component 设计入门介绍](./images/component.png "ReactJS 与 Component 设计入门介绍")

在 React 的世界中最基本的单元为组件（Component），每个组件也可以包含一个以上的子组件，并依照需求组装成一个组合式的（Composable）组件，因此具有封装（encapsulation）、关注点分离 (Separation of Concerns)、复用 (Reuse) 、组合 (Compose) 等特性。

`<TodoApp>` 组件可以包含 `<TodoHeader />`、`<TodoList />` 子组件

```javascript
    <div>
        <TodoHeader />
        <TodoList />
    </div>
```

`<TodoList />` 组件内部长相：

```javascript
    <div>
        <ul>
            <li>写程式码</li>
            <li>哄妹子</li>
            <li>买书</li>
        </ul>
    </div>
```

组件化一直是网页前端开发的万金油，许多开发者最希望的就是可以最大化重复使用（reuse）过去所写的程式码，不要重复造轮子（DRY）。在 React 中组件是一切的基础，让开发应用程式就好像在堆积木一样。然而对于过去习惯模版式（template）开发的前端工程师来说，短时间要转换成组件化思考模式并不容易，尤其过去我们往往习惯于将 HTML、CSS 和 JavaScript 分离，现在却要把它们都封装在一起。

一个比较好的方式就是训练自己看到不同的网页或应用程式时，强迫自己将看到的页面切成一个个组件。相信过了一段时间后，天眼开了，就比较容易习惯组件化的思考方式。

以下是一般 React Component 撰写的主要两种方式：

1. 使用 ES6 的 Class（可以进行比较复杂的操作和组件生命周期的控制，相对于 stateless components 耗费资源）

    ```javascript
    //  注意组件开头第一个字母都要大写
    class MyComponent extends React.Component {
        // render 是 Class based 组件唯一必须的方法（method）
        render() {
            return (
                <div>Hello, World!</div>
            );
        }
    }

    // 将 <MyComponent /> 组件插入 id 为 app 的 DOM 元素中
    ReactDOM.render(<MyComponent/>, document.getElementById('app'));
    ```

2. 使用 Funtional Component 写法（单纯地 render UI 的 stateless components，没有内部状态、没有实作物件和 ref，没有生命周期函数。若非需要控制生命周期的话建议多使用 stateless components 获得比较好的性能）

    ```javascript
    // 使用 arror function 来设计 Funtional Component 让 UI 设计更单纯（f(D) => UI），减少副作用（side effect）
    const MyComponent = () => (
        <div>Hello, World!</div>
    );

    // 将 <MyComponent /> 组件插入 id 为 app 的 DOM 元素中
    ReactDOM.render(<MyComponent/>, document.getElementById('app'));
    ```

## 用 JSX 进行声明式（Declarative）UI 设计

React 在设计上的思路认为使用 Component 比起模版（Template）和显示逻辑（Display Logic）更能实现关注点分离的概念，而搭配 JSX 可以实现声明式 Declarative（注重 what to），而非命令式 Imperative（注重 how to）的程序编写方式。

像下述的声明式（Declarative）UI 设计就比单纯用（Template）式的方式更易懂：

```javascript
// 使用声明式（Declarative）UI 设计很容易可以看出这个组件的功能
<MailForm />
```

```javascript
// <MailForm /> 内部长相
<form>
    <input type="text" name="email" />
    <button type="submit"></button>
</form>
```

由于 JSX 在 React 组件编写上扮演很重要的角色，因此在下一个章节我们也将更深入讲解 JSX 使用细节。

## 使用 Virtual DOM

在传统 Web 中一般是使用 jQuery 进行 DOM 的直接操作。然而更改 DOM 往往是 Web 性能的瓶颈，因此在 React 世界设计有 Virtual DOM 的机制，让 App 和 DOM 之间用 Virtual DOM 进行沟通。当更改 DOM 时，会通过 React 自身的 diff 算法去计算出最小更新，进而去最小化更新真实的 DOM。

## Component PropType 错误校对机制

在 React 设计时除了提供 props 预设值设定（Default Prop Values）外，也提供了 Prop 的验证（Validation）机制，让整个 Component 设计更加稳健：

```javascript
//  注意组件开头第一个字母都要大写
class MyComponent extends React.Component {
    // render 是 Class based 组件唯一必须的方法（method）
    render() {
        return (
            <div>Hello, World!</div>
        );
    }
}

// PropTypes 验证，若传入的 props type 不符合将会显示错误
MyComponent.propTypes = {
  todo: React.PropTypes.object,
  name: React.PropTypes.string,
}

// Prop 预设值，若对应 props 没传入值将会使用 default 值
MyComponent.defaultProps = {
 todo: {},
 name: '',
}
```

关于更多的 Validation 用法可以参考[官方网站](https://facebook.github.io/react/docs/reusable-components.html) 的说明。

## Component 就像个状态机（State Machine），而且也有生命周期（Life Cycle）

Component 就像个状态机（State Machine），根据不同的 state（通过 `setState()` 修改）和 props（由父元素传入），Component 会出现对应的显示结果。而人有生老病死，组件也有生命周期。通过操作生命周期处理函数，可以在对应的时间点进行 Component 需要的处理，关于更详细的组件生命周期介绍我们会再下一个章节进行更一步说明。

## 一律重绘（Always Redraw）和单向数据流（Unidirectional Data Flow）

在 React 世界中，props 和 state 是影响 React Component 长相的重要要素。其中 props 都是由父元素所传进来，不能更改，若要更改 props 则必须由父元素进行更改。而 state 则是根据使用者互动而产生的不同状态，主要是通过 setState() 方法进行修改。当 React 发现 props 或是 state 更新时，就会重绘整个 UI。当然你也可以使用 forceUpdate() 去强迫重绘 Component。而 React 通过整合 Flux 或 Flux-like（例如：Redux）可以更具体实现单向数据流（Unidirectional Data Flow），让数据流的管理更为清晰。

## 在 JavaScript 里写 CSS：Inline Style

在 React Component 中 CSS 使用 Inline Style 写法，全都封装在 JavaScript 当中：

```javascript
const divStyle = {
  color: 'red',
  backgroundImage: 'url(' + imgUrl + ')',
};

ReactDOM.render(<div style={divStyle}>Hello World!</div>, document.getElementById('app'));
```

## 总结

以上介绍了 ReactJS 的几个重要特性：

1. 基于组件（Component）化思考
2. 用 JSX 进行声明式（Declarative）UI 设计
3. 使用 Virtual DOM
4. Component PropType 错误校对机制
5. Component 就像个状态机（State Machine），而且也有生命周期（Life Cycle）
6. 一律重绘（Always Redraw）和单向数据流（Unidirectional Data Flow）
7. 在 JavaScript 里写 CSS：Inline Style

接下来我们将进一步探讨 React 里 JSX 的使用方式。

## 延伸阅读

1. [React 入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html)
2. [React Demystified](http://blog.reverberate.org/2014/02/react-demystified.html)
3. [Top-Level API](https://facebook.github.io/react/docs/top-level-api.html)
4. [ES6 Classes Component](https://facebook.github.io/react/docs/reusable-components.html#es6-classes)

（image via [maketea](http://maketea.co.uk/images/2014-03-05-robust-web-apps-with-react-part-1/wireframe_deconstructed.png)）

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：React 开发环境设置与 Webpack 入门教学](https://github.com/aclk/reactjs101/blob/master/Ch02/webpack-dev-enviroment.md) | [下一章：JSX 简明入门教学指南](https://github.com/aclk/reactjs101/blob/master/Ch03/react-jsx-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
